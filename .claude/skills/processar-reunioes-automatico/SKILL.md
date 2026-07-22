---
name: processar-reunioes-automatico
description: Fluxo automático de ponta a ponta que varre o Google Calendar de uma pessoa do time de vendas da Pipo, filtra reuniões qualificadas (cliente ou Pré-Kickoff), busca a transcrição no Drive, chama a skill resumo-reuniao para classificar e resumir, registra a observação no HubSpot e notifica o Slack. Rodada por uma scheduled task agendada (1x/dia, por pessoa).
---

# Processar Reuniões (fluxo automático)

Status: completo. Todas as fases (filtro mestre, qualificação, dedup, transcrição, cérebro, HubSpot, Slack) implementadas e testadas com dados e reuniões reais em 21-22/07/2026. Falta só o setup real das scheduled tasks nas contas das 8 pessoas (ver "Setup" no fim deste arquivo).

**Pipelines comerciais confirmados nesta conta HubSpot** (não existe pipeline chamado literalmente "Vendas"; o time confirmou que "Vendas" era só um jeito informal de falar de "Oportunidades"):
- `default` = Oportunidades
- `8582978` = Desenvolvimento de Negócios

## Entradas

- Scheduled task roda 1x/dia (9h05, horário de Brasília) para o dono da rotina, cobrindo as reuniões do dia anterior.
- `config/time-vendas.yaml` (lista do time de vendas elegível + domínios internos).

## Passo 1: Janela de tempo

Listar os eventos do Calendar do dono da rotina cobrindo o dia anterior completo, com uma margem de segurança (aproximadamente as últimas 34-40 horas a partir da hora da rodada), não um corte exato de meia-noite a meia-noite.

**Histórico de decisões sobre o agendamento:**
- O documento original definia duas rodadas por dia (12h30 e 18h15) com janelas exatas de manhã/tarde e double-check cruzado, porque, sem dedup, era preciso acertar um recorte específico para não repetir nem perder reunião.
- Com o Passo 4 garantindo dedup de verdade (por marcador na nota), uma janela larga e simples já ficou mais robusta que o double-check original (cobre transcrição atrasada, rodada que falhou, etc.).
- Em 22/07/2026, o time decidiu simplificar ainda mais: **uma única rodada diária, às 9h05 (horário de Brasília)**, processando as reuniões do dia anterior, em vez de duas rodadas por dia. Motivo: com o dedup já garantindo que nada é processado duas vezes, rodar duas vezes por dia parou de agregar valor real; uma rodada pela manhã, depois que a transcrição do Gemini já teve a noite inteira para ficar pronta, é mais simples e cobre o mesmo território. A margem de segurança (~34-40h em vez de um corte exato de 24h) continua existindo para pegar qualquer transcrição que ainda não estivesse pronta na rodada anterior.

## Passo 2: Filtro mestre

Um evento só é candidato a processamento se o **organizador (dono) do evento for a mesma pessoa dona desta rotina** (não apenas alguém do time de vendas em geral).

**Atenção, bug real encontrado e corrigido (21/07/2026):** a versão anterior desta regra checava só "o organizador está na lista de `config/time-vendas.yaml`", sem comparar com quem é o dono da rotina. Isso deixava passar reuniões onde o dono da rotina é apenas convidado, mas o organizador é outra pessoa do time (ex.: a rotina do Marcelo processando uma reunião organizada pela Larissa, só porque ele também foi convidado). Isso quebra a autoria correta (a nota/mensagem saem em nome de quem está rodando a rotina, não de quem é realmente dono da reunião) e, quando a outra pessoa também tiver rotina própria, geraria processamento redundante da mesma reunião por duas rotinas diferentes.

Se o organizador não for o dono da rotina (seja porque é outra pessoa do time, seja porque é o cliente), **não processar automaticamente**: no caso de outra pessoa do time, a reunião é dela processar pela própria rotina; no caso do cliente ser organizador, é exatamente o caso de uso do modo manual (skill `resumo-reuniao` usada diretamente por alguém colando a transcrição).

## Passo 3: Regra de qualificação

Um evento que passou pelo filtro mestre só é processado se atender a **pelo menos um** destes critérios:

1. **Tem participante externo**: algum convidado do evento com e-mail fora dos domínios listados em `dominios_internos` no `config/time-vendas.yaml` (hoje: `piposaude.com.br` e `pipo.ai`; comparação case-insensitive no domínio). O `pipo.ai` foi incluído porque, na prática, pessoas do time às vezes recebem convite nesse domínio em vez de `piposaude.com.br` (mesma pessoa, domínio diferente) e isso não deve contar como participante externo.
2. **É um Pré-Kickoff**: o título do evento contém "pré-kickoff" ou "pre-kickoff" (case e acento-insensível), mesmo sem nenhum participante externo. Padrão de título assumido (a confirmar com o time): `Pré-Kickoff - {Empresa}` ou `Pré-Kickoff: {Empresa}`. Qualquer variação que tenha "pré-kickoff" seguido do nome da empresa é aceita; o essencial é conseguir extrair o nome da empresa do restante do título.

Se **nenhum** dos dois critérios for atendido (reunião interna, 1:1, etc.), **ignorar o evento**: não resumir, não registrar, não notificar.

## Passo 4: Checar duplicata (dedup)

Antes de fazer qualquer outro trabalho (evita gastar tempo com transcrição e classificação de algo que já foi processado): buscar em `notes` por `hs_note_body CONTAINS_TOKEN {eventId}`. Não é preciso restringir a busca a uma Company específica: o ID do evento do Calendar já é único globalmente.

- Achou uma nota com o marcador `Ref. evento Calendar: {eventId}` → esse evento já foi processado; **pular**, não fazer mais nada com ele.
- Não achou → seguir para o Passo 5.

**Limitação conhecida:** notas criadas antes desta automação existir, ou antes da correção do formato do marcador (ver Passo 9), não têm essa linha e não serão reconhecidas como "já processadas". Isso inclui a nota de teste da Cordeiro Guindastes criada durante o desenvolvimento (`113371229342`), que ainda usa o formato antigo (comentário HTML, não indexado pela busca do HubSpot); ela não seria pega pelo dedup se o evento fosse reprocessado. Só afeta esse histórico pontual, não o funcionamento daqui para frente.

## Passo 5: Localizar a transcrição no Drive

**Primeiro, checar o próprio evento do Calendar.** Em alguns casos (confirmado com um evento real de hoje) o Gemini já anexa o documento de notas direto no campo `attachments` do evento, com título "Anotações do Gemini" e um `fileUrl` do Google Docs. Se o evento já tiver esse anexo, use-o diretamente; é mais direto e confiável do que buscar no Drive.

Se o evento não tiver esse anexo (caso mais comum nos testes até agora, ex.: Cordeiro Guindastes, Promon, DOT), busque no Drive. O Gemini nomeia o arquivo de notas/transcrição assim (confirmado com arquivos reais): `{Título do evento} - AAAA/MM/DD HH:MM GMT-03:00 - Anotações do Gemini`.

Buscar no Drive (`search_files`) um Google Doc cujo título contenha uma parte distintiva do título do evento **e** "Anotações do Gemini" (ex.: `title contains 'Cordeiro' and title contains 'Gemini'`). Se vier mais de um resultado, desambiguar pela data/hora embutida no título do arquivo, comparando com a data/hora do evento.

- **Nenhum resultado** → a transcrição ainda não está pronta (o Gemini pode demorar). Não processar esta reunião nesta rodada; ela será pega automaticamente numa rodada seguinte, já que o Passo 1 usa uma janela larga.
- **Achou** → ler o conteúdo do documento (`read_file_content`). Esse conteúdo (Resumo + Próximas etapas + Detalhes + Transcrição, gerados pelo Gemini) é o que vai para a skill `resumo-reuniao` como "a transcrição da reunião". Não é preciso ler só a seção de transcrição bruta: o resumo estruturado que o próprio Gemini já gera é uma fonte válida e, na prática, mais fácil de processar do que a transcrição palavra-por-palavra.

## Passo 6: Classificar e resumir (chamar o cérebro)

Chamar a skill `resumo-reuniao` com:
- a transcrição (conteúdo do Passo 5),
- o título do evento e a data da reunião,
- a lista de participantes/e-mails do evento (Passo 3),
- o estágio do negócio no HubSpot, se já tiver sido possível identificar a empresa/negócio nesse ponto (sinal opcional de reforço para a classificação).

Retorno esperado: o tipo classificado (um dos 12, ou o fallback geral) e o resumo já formatado.

## Passo 7: Identificação da empresa

Ordem de prioridade:

1. **Domínio do participante externo.** Se houver um ou mais convidados externos, use o domínio do e-mail deles para depois buscar a Company no HubSpot por domínio (busca em si é o Passo 8). Esse é o método principal e mais confiável, porque resolve empresas homônimas no HubSpot.
   - Se todos os convidados externos compartilham o mesmo domínio, use esse domínio.
   - Se houver mais de um domínio externo distinto, **não escolha um arbitrariamente nem assuma ambiguidade de cara**. Na prática (validado com calendário e HubSpot reais), é comum a mesma empresa aparecer com dois domínios diferentes (ex.: "Cordeiro Guindastes" com `cordeiroguindastes.com.br` e `grupocordeiro.srv.br`; "Fundação Butantan" com `butantan.gov.br` e `fundacaobutantan.org.br`). No Passo 8, busque a Company no HubSpot por **cada** domínio candidato: se todos resolverem na mesma Company, não há ambiguidade real. Se resolverem em Companies diferentes, verifique o histórico de negócios de cada uma (Passo 8.2): a que tiver negócio aberto batendo com o contexto da reunião é a conta ativa; a outra costuma ser um registro duplicado/incompleto. Só sinalize para revisão manual se ambas (ou nenhuma) tiverem negócio para desempatar.
2. **Nome no título do evento**, como fallback: usado quando não há convidado externo identificável por domínio (o caso central é o Pré-Kickoff, que é interno). Extraia o nome da empresa do título removendo o prefixo do tipo de reunião (ex.: de "Pré-Kickoff - Cordeiro Guindastes" extrair "Cordeiro Guindastes").

Se não for possível identificar a empresa por nenhum dos dois métodos, não invente. Trate como um caso a resolver manualmente (mesma lógica do modo manual: perguntar antes de registrar).

## Passo 8: Buscar Company e negócios abertos no HubSpot

1. **Buscar a Company** pelo(s) domínio(s) candidato(s) do Passo 7 (`companies`, filtro `domain EQ {dominio}`). Empresas homônimas são reais nesta conta (ex.: existem 3 Companies chamadas "Arizona" com domínios diferentes); é por isso que a busca é sempre por domínio, nunca por nome, quando há domínio disponível.
   - **Domínio do e-mail não bate com o domínio cadastrado na Company** (validado com um caso real: a Promon usa e-mails `@promon.com.br`, mas a Company no HubSpot está cadastrada com domínio `promonengenharia.com.br`): se a busca por domínio vier vazia, não desista, tente buscar por nome (`query`, usando a parte reconhecível do domínio ou do título do evento como termo, ex.: "Promon"). Se vier exatamente um resultado plausível, siga com ele. Se vier mais de um, sinalize a ambiguidade em vez de escolher sozinho.
   - Sem domínio disponível (fallback do Passo 7, caso Pré-Kickoff) → buscar Company por nome (`query: {nome}`). Se vier mais de um resultado, não escolher sozinho: sinalizar a ambiguidade.
   - Nenhum resultado por domínio nem por nome → não inventar Company; tratar como caso a resolver manualmente.
2. **Buscar negócios abertos** associados a essa Company: `deals`, `filterGroups: [{filters: [{pipeline IN [default, 8582978]}, {hs_is_closed EQ false}], associatedWith: [{companies EQUAL [companyId]}]}]`. Usar a propriedade padrão `hs_is_closed` (True/False, mantida automaticamente pelo HubSpot para qualquer pipeline) em vez de listar manualmente quais dealstages contam como "fechado" por pipeline; é mais robusto.
   - 0 negócios abertos → a nota será associada **somente à Company** (fallback da seção 10 da spec original).
   - 1+ negócios abertos → associar a Company e **todos** os negócios abertos encontrados.

## Passo 9: Criar a observação (nota) no HubSpot

Usa o objeto `notes`, criado com `manage_crm_objects` (`createRequest`):

- `hs_note_body`: o resumo do Passo 6, convertido para o HTML simples que o HubSpot já usa nas notas reais desta conta: `<h2><strong>Tipo | DD/MM/AAAA</strong></h2>` para o título, `<p><strong>Rótulo:</strong> valor</p>` para campos, `<ul><li>...</li></ul>` para listas. Validado contra uma nota real já existente na conta (Arizona, Reunião de Diagnóstico de 20/07/2026), que segue exatamente esse padrão.
- `hs_timestamp`: data/hora da reunião (do evento do Calendar), não a data de criação da nota.
- **Marcador de dedup**: uma linha discreta (fonte pequena, cor cinza) no **fim** do corpo, depois de "Participantes da reunião": `<p style="font-size:11px;color:#999;margin-top:12px;">Ref. evento Calendar: {id}</p>`. Testado e confirmado: um comentário HTML (`<!-- ... -->`) **não funciona** aqui, porque o HubSpot indexa a busca a partir do texto visível da nota, que descarta comentários HTML. O marcador precisa ser texto real, mesmo que estilizado para ficar discreto.
- **Associações**: sempre a Company; e cada negócio aberto identificado no Passo 8, se houver.

Sobre confirmação: a ferramenta `manage_crm_objects` exige aprovação explícita (mostrar a mudança proposta e esperar confirmação) antes de criar qualquer registro. **Em qualquer teste manual, sempre mostrar a nota proposta (corpo + associações) e esperar aprovação explícita antes de criar de verdade.** No fluxo automático rodando via scheduled task agendada, sem humano no loop, a criação prossegue sem essa pausa interativa (mesmo comportamento já assumido para o Slack na seção 11 da spec: "envio automático, sem confirmação, é o comportamento desejado do fluxo agendado"). Isso só deve valer depois que o fluxo estiver validado em piloto real. Na prática (22/07/2026), isso exige duas coisas: (1) o prompt da scheduled task precisa dizer explicitamente que ela está autorizada a passar `confirmationStatus: CONFIRMED`; (2) as ferramentas `manage_crm_objects` e `slack_send_message` (e as de leitura mais usadas: `search_crm_objects`, `list_events`, `read_file_content`, `search_files`, `get_properties`, `list_scheduled_tasks`) precisam estar na allowlist de permissões do Claude Code (`.claude/settings.json` do projeto), senão a execução agendada trava pedindo aprovação sem ninguém para responder.

## Passo 10: Notificar o Slack

Canal `#reunioes-vendas` (ID `C0A2FRE5F6E`). Mencionar o dono da reunião pelo Slack user ID real (`<@USERID>`, buscar via `slack_search_users` pelo e-mail/nome; não usar @nome em texto puro). Modelo de mensagem (seção 11 da spec original), com links diretos para a nota, a Company e o(s) Deal(s) criados/associados no Passo 9:

```
📝 Nova observação registrada no HubSpot
<@USERID>, o resumo da sua reunião foi registrado.
Reunião: [título do evento]
Tipo: [tipo identificado]
Data: [DD/MM/AAAA]
Registrado em: empresa [nome] + negócio(s) [nome(s)]
🔗 [link da nota] [link da empresa] [link do(s) negócio(s)]
```

**Nota sobre autoria em modo automático:** cada scheduled task roda com a conta Slack de quem é dono daquela reunião, então a mensagem sai postada pela própria pessoa mencionada (ela se menciona), o que é o comportamento esperado do fluxo agendado (seção 11: "cada pessoa posta com o próprio Slack").

## Validado ponta a ponta (21-22/07/2026)

Reuniões reais processadas de ponta a ponta (transcrição, cérebro, HubSpot, Slack), além de um backfill histórico de 34 reuniões (22 notas criadas, sem Slack) cobrindo 01/05 a 21/07:

- **Pipo Saúde + Cordeiro Guindastes** (16/07/2026, Diagnóstico): empresa e negócio identificados resolvendo uma ambiguidade real de domínio pelo histórico de negócio.
- **Pipo Saúde + Promon | Consultoria de Benefícios** (15/07/2026, Briefing de Consultoria de Benefícios): domínio do e-mail (`promon.com.br`) não batia com o domínio cadastrado na Company (`promonengenharia.com.br`); resolvido por busca de nome (Passo 8).
- **Pipo Saúde + DOT | Kickoff** (17/07/2026, Kickoff): Company com 36 negócios no histórico, mas 0 abertos nos pipelines comerciais (todos fechados ou em outro pipeline como Apólices); nota associada só à Company, confirmando que esse fallback é o comportamento normal, não um erro, para clientes já implantados.
- **Pipo Saúde + Conexa Saúde** (21/07/2026, Briefing de Consultoria de Benefícios): teste de ponta a ponta rodado manualmente (reunião organizada pela Larissa, processada excepcionalmente pela conta do Marcelo enquanto ela ainda não tinha rotina própria).

Em todos os casos a transcrição foi localizada automaticamente (anexo no próprio evento do Calendar ou busca no Drive por título, Passo 5), sem link fornecido manualmente.

**Descoberta real sobre a execução agendada em si (22/07/2026):** as duas primeiras execuções agendadas (antes desta seção existir) dispararam no horário certo mas não completaram nenhum trabalho, porque as ferramentas usadas (`manage_crm_objects`, `slack_send_message`, etc.) não estavam na allowlist de permissões do Claude Code, e sem humano presente para aprovar, a execução travava. Corrigido adicionando essas ferramentas ao `.claude/settings.json` do projeto (ver Passo 9). Recomendação: depois de criar ou alterar uma scheduled task, sempre rodar "Run now" uma vez para confirmar que ela completa sem travar em nenhum prompt de aprovação, antes de deixá-la rodando sozinha.

## Setup: scheduled tasks (execução agendada)

- Cada uma das 8 pessoas do time de vendas cria **1 scheduled task** na própria conta Claude, às 9h05 (horário de Brasília), invocando esta skill. Decisão de 22/07/2026: uma rodada diária (não mais duas) é suficiente, já que o dedup (Passo 4) garante que nada é processado duas vezes; rodar de manhã dá a noite inteira para a transcrição do Gemini ficar pronta.
- Cada scheduled task precisa dos conectores conectados com a **própria conta da pessoa**: Google Calendar, Google Drive, HubSpot, Slack. Isso preserva a privacidade (cada fluxo só acessa a própria agenda) e a autoria correta (nota e mensagem saem em nome de quem é dono da reunião), sem precisar gerenciar tokens brutos de API.
- As ferramentas usadas pela skill (leitura e escrita, listadas no Passo 9) precisam estar na allowlist de `.claude/settings.json` para a scheduled task não travar pedindo aprovação sem ninguém para responder.
- Ver `README.md` na raiz do repo para o passo a passo de conexão de contas e criação da scheduled task.

## Limitações conhecidas

- Notas criadas manualmente antes do go-live (ou com o formato antigo de marcador) não são reconhecidas pelo dedup.
- Ambiguidade de empresa/negócio quando não há histórico de negócio para desempatar entre Companies homônimas: fica sinalizado para revisão manual, não decidido sozinho.
- O padrão de título do Pré-Kickoff ainda não foi oficialmente adotado pelo time; a skill assume `Pré-Kickoff - {Empresa}` até isso ser confirmado.
