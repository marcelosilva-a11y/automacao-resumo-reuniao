# Especificação Técnica: Automação de Resumos de Reunião (Pipo Saúde)

**Versão:** 1.0
**Data:** 20/07/2026
**Objetivo do documento:** descrever, de ponta a ponta, a automação que lê reuniões comerciais no Google Calendar, resume a transcrição (gerada pelo Gemini) e registra uma observação no HubSpot, notificando o time no Slack. Serve como guia de construção para o Claude Code.

> **Nota (21/07/2026):** o projeto foi construído seguindo este documento, com uma mudança de arquitetura na execução (scheduled tasks do Claude Code em vez de GitHub Actions) e alguns ajustes descobertos durante a implementação com dados reais. Ver `README.md` e `.claude/skills/processar-reunioes-automatico/SKILL.md` para o desenho final.

---

## 1. Visão geral

A automação roda na nuvem (GitHub Actions), duas vezes por dia, para cada pessoa do time de vendas. Em cada rodada ela:

1. Varre o Google Calendar da pessoa procurando reuniões dentro de uma janela de tempo.
2. Filtra apenas as reuniões que interessam (com cliente, ou Pré-Kickoff interno).
3. Localiza a transcrição da reunião (Google Doc gerado pelo Gemini, no Drive).
4. Classifica o tipo da reunião entre os 12 tipos comerciais.
5. Gera o resumo no formato definido, usando o template do tipo identificado.
6. Identifica a empresa e os negócios no HubSpot.
7. Cria a observação no HubSpot, associada à empresa e aos negócios qualificados.
8. Notifica no canal `#reunioes-vendas` do Slack, mencionando o dono da reunião.

O "cérebro" (classificação + templates de resumo) deve ser reutilizável, servindo tanto o modo automático quanto um modo manual (pessoa cola a transcrição no chat do Claude).

---

## 2. Arquitetura

- **Execução:** GitHub Actions, com agendamento via `schedule` (cron). Roda sem depender de máquina ligada.
- **Modelo multiusuário:** cada uma das 8 pessoas do time roda com as próprias credenciais (Google, HubSpot, Slack). Isso preserva privacidade (cada fluxo só acessa a agenda da própria pessoa) e mantém a autoria correta (observação e mensagem saem em nome de quem é dono da reunião).
- **Sem conta central:** decisão consciente para evitar que uma conta única tenha acesso à agenda/Drive de todos, e para não distorcer a autoria dos registros.

### Fluxo de dados por pessoa

```
Google Calendar (eventos da janela)
    → filtro de qualificação
    → Google Drive (Doc da transcrição do Gemini)
    → classificação + resumo (cérebro)
    → HubSpot (busca empresa + negócios; cria observação)
    → Slack (notifica #reunioes-vendas)
    → registro de "já processado" (controle de duplicatas)
```

---

## 3. Time de vendas (filtro mestre)

A automação só processa reuniões cujo **dono (organizador)** é uma destas pessoas:

- marcelo.silva@piposaude.com.br
- aline.furlaneto@piposaude.com.br
- breno.hauss@piposaude.com.br
- erika.silva@piposaude.com.br
- luciano.moreira@piposaude.com.br
- rafael.falcao@piposaude.com.br
- larissa.martins@piposaude.com.br
- barbara.borges@piposaude.com.br

Esta lista deve ficar em um arquivo de configuração, fácil de atualizar quando alguém entra ou sai do time, sem mexer na lógica.

---

## 4. Agendamento e janelas de tempo

Duas rodadas por dia (horário de Brasília):

### Rodada 12h30
- **Processa:** reuniões da **manhã de hoje**.
- **Double-check:** reuniões da **tarde de ontem** (pega qualquer uma que não tenha sido registrada).

### Rodada 18h15
- **Processa:** reuniões da **tarde de hoje**.
- **Double-check:** reuniões da **manhã de hoje** (pega qualquer uma que escapou na rodada do meio-dia).

Observação sobre fuso: o GitHub Actions usa UTC no cron. Brasília (UTC-3): 12h30 = `30 15 * * *` e 18h15 = `15 21 * * *`. Atenção a eventual horário de verão (hoje o Brasil não adota, mas registrar como ponto de manutenção).

---

## 5. Regra de qualificação (quais reuniões entram)

Uma reunião é processada **somente se** o dono está na lista do time de vendas **E** atende a um destes critérios:

1. **Tem participante externo** (alguém fora do domínio `@piposaude.com.br`): é reunião com cliente.
2. **É um Pré-Kickoff**: identificada pelo nome do evento (padrão a ser adotado pelo time no título, incluindo o nome da empresa), mesmo sendo reunião interna sem externos.

Se a reunião **não tem externo e não é Pré-Kickoff**, é ignorada (reunião interna, 1:1, etc.). Não resume, não registra.

---

## 6. Identificação da empresa

Ordem de prioridade:

1. **Pelo domínio do e-mail** dos participantes externos: buscar a empresa (Company) no HubSpot pelo domínio. Este é o método principal e mais confiável.
2. **Pelo nome no título do evento**: fallback, usado quando não há externo identificável pelo domínio. É o método usado no Pré-Kickoff (interno).

Nota de implementação: podem existir empresas homônimas no HubSpot (ex.: várias "Arizona"). O match por domínio resolve a ambiguidade; priorizar sempre o domínio sobre o nome.

---

## 7. Localização da transcrição

- A transcrição é gerada pelo **Gemini** e salva como **Google Doc no Drive** do organizador (tipicamente pasta "Meet Recordings"); o link costuma vir anexado ao evento do Calendar.
- O fluxo deve: pegar o link/anexo da transcrição no evento do Calendar, ler o conteúdo do Doc via Google Drive.
- Se a reunião qualifica mas **ainda não tem transcrição pronta** (o Gemini pode demorar), a reunião não é processada nessa rodada; fica para a próxima (o double-check garante nova tentativa).

---

## 8. Classificação do tipo de reunião

Classificar entre os 12 tipos, cruzando três sinais: título do evento, conteúdo da transcrição e estágio do negócio no HubSpot.

Os 12 tipos:

1. Reunião de Diagnóstico
2. Reunião de Briefing de Cotação
3. Reunião de Apresentação de Cotação
4. Reunião de Negociação
5. Reunião de Follow-up Comercial
6. Reunião com Parceiro
7. Apresentação do Time de Saúde
8. Apresentação da Plataforma
9. Briefing de Consultoria de Benefícios
10. Apresentação de Consultoria de Benefícios
11. Kickoff
12. Pré-Kickoff

Se não for possível cravar o tipo com segurança, **não forçar** uma classificação: usar um título temático e fazer um resumo geral organizado (assuntos, informações, dores, decisões, próximos passos, participantes). Nenhuma reunião qualificada fica sem registro.

Boa prática: declarar no topo da observação o tipo identificado, para o vendedor poder corrigir se necessário.

---

## 9. Formato do resumo (regras gerais)

Baseado no documento de instruções da Pipo:

- Escrever de forma objetiva, organizada, adequada ao HubSpot.
- **Não inventar, presumir ou completar** informações que não estejam na transcrição.
- Usar títulos e macrotópicos; evitar repetição.
- Priorizar o que é relevante para o processo comercial e a continuidade da oportunidade.
- **Toda** anotação tem seção de **próximos passos** (ações, responsáveis, prazos quando mencionados).
- Tamanho-alvo: ~1 página; até 2 páginas só quando houver volume real de informação relevante.
- **Título:** `Tipo de Reunião | DD/MM/AAAA`, em destaque visual (negrito/cabeçalho).
- **Última seção sempre:** `Participantes da reunião` (nome, empresa, cargo/time, e-mail quando identificável). Não inventar participantes.

As seções específicas por tipo seguem o documento de instruções da Pipo (as dores por temática/subtópicos, características do plano, contexto geral, pontos de atenção e oportunidades de negociação etc.). O template de Briefing de Cotação usa o snippet de campos específico (Plano de Saúde, Odontológico, Seguro de Vida, Outros Benefícios), incluindo seções de benefício apenas quando fazem parte do escopo.

Sobre "oportunidades sugeridas" (quando aplicável): manter breve e ancorado no que apareceu na reunião; não inventar diferenciais não citados.

---

## 10. Regra dos negócios (associação no HubSpot)

A observação é sempre associada à **empresa (Company)**. Quanto aos negócios (Deals):

- Associar a **todo negócio** da empresa que esteja em um dos três funis comerciais **E** aberto (qualquer estágio que não seja perdido/fechado).
- Os funis comerciais (pipelines) considerados: **Vendas, Oportunidades e Desenvolvimento de Negócios**.
  - No HubSpot da Pipo, "Oportunidades" corresponde ao pipeline de nome técnico `default`; "Desenvolvimento de Negócios" ao pipeline `8582978`. Confirmar o pipeline "Vendas" na conta (pode ser o mesmo "Oportunidades" ou um pipeline distinto) na implementação.
- Se houver **mais de um** negócio qualificado, associar a **todos**.
- Se **não houver nenhum** negócio qualificado, associar **somente à empresa** (fallback).

A criação usa o objeto `notes` do HubSpot, com associações à Company e aos Deals na mesma nota. Campos: `hs_note_body` (corpo, aceita HTML simples para formatação) e `hs_timestamp` (data da reunião).

---

## 11. Notificação no Slack

- Canal: **#reunioes-vendas** (ID `C0A2FRE5F6E`).
- **Envio automático, sem confirmação**: este é o comportamento desejado no fluxo agendado.
- A mensagem menciona o dono da reunião (@pessoa) e traz: nome da reunião, tipo identificado, data, onde foi registrado (empresa + negócios) e links diretos para os registros no HubSpot.
- Cada pessoa posta com o próprio Slack (autoria correta).

Modelo de mensagem:

```
📝 Nova observação registrada no HubSpot
@dono, o resumo da sua reunião foi registrado.
Reunião: [título]
Tipo: [tipo identificado]
Data: [DD/MM/AAAA]
Registrado em: empresa [nome] + negócio(s) [nome(s)]
🔗 [links HubSpot]
```

---

## 12. Controle de duplicatas

Essencial para o double-check funcionar sem registrar a mesma reunião duas vezes.

- O fluxo mantém um registro persistente de reuniões **já processadas** (chave sugerida: ID do evento do Calendar).
- Antes de processar, checa se o evento já está no registro. Se estiver, pula.
- Ao registrar com sucesso, grava o ID no registro.
- Onde guardar esse registro: opções simples são um arquivo no próprio repositório (commitado pela Action) ou um armazenamento externo leve. Definir na implementação; o importante é que persista entre rodadas.

---

## 13. Modo manual (rede de segurança)

O mesmo "cérebro" (classificação + templates + regra de registro) deve ser empacotado como uma **skill do projeto Claude**, para que:

- No **automático** (Claude Code), o fluxo busca a transcrição no Drive e chama o cérebro.
- No **manual**, uma pessoa do time cola a transcrição no chat e o cérebro processa igual: útil para os casos em que o **cliente é o organizador** (a transcrição fica no Drive do cliente, fora do alcance da automação).

No modo manual, como pode faltar o contexto do evento (convidados), o fluxo pode precisar perguntar/confirmar a empresa antes de registrar.

---

## 14. Credenciais necessárias (setup)

Para cada pessoa do time (8 no total):

- **Google (Calendar + Drive):** acesso de leitura à própria agenda e aos Docs de transcrição. Via OAuth da conta Google corporativa.
- **HubSpot:** token com escopo de **escrita de CRM** (criação de notes/engagements e associações). Gerar um "private app" no HubSpot (Configurações → Integrações → Private Apps), com os escopos de CRM necessários, e guardar o token com segurança.
- **Slack:** acesso para postar no canal #reunioes-vendas em nome da pessoa.

**Armazenamento seguro:** todas as credenciais ficam como **secrets** do GitHub (nunca no código). No modelo de 8 pessoas, organizar os secrets por pessoa.

> Nota de segurança: o token do HubSpot dá permissão de escrita na conta; tratar como credencial sensível, gerada por quem administra o HubSpot da Pipo e guardada apenas como secret.

---

## 15. Passos de construção sugeridos (ordem)

1. Implementar o "cérebro" (classificação + templates) como skill reutilizável e testá-lo no modo manual.
2. Implementar os conectores de leitura: Calendar (eventos + janelas) e Drive (transcrição).
3. Implementar a lógica de qualificação e identificação de empresa.
4. Implementar a escrita no HubSpot (busca de empresa/negócios + criação da observação com associações).
5. Implementar a notificação no Slack.
6. Implementar o controle de duplicatas.
7. Montar o workflow do GitHub Actions com os dois horários (cron) e os secrets.
8. Rodar um piloto real (validar com reuniões de verdade), depois ativar as 8 pessoas.

---

## 16. Itens de manutenção / atenção

- **Horário de verão:** se o Brasil voltar a adotar, revisar os horários do cron (UTC).
- **Lista do time:** manter atualizada quando houver entrada/saída.
- **Nome do pipeline "Vendas":** confirmar na conta HubSpot qual pipeline corresponde.
- **Padrão de nome do Pré-Kickoff:** garantir que o time siga o padrão no título do evento.
- **Latência da transcrição do Gemini:** reuniões sem transcrição pronta são retentadas na rodada seguinte (o double-check cobre isso).
