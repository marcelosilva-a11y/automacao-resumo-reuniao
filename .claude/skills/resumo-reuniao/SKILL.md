---
name: resumo-reuniao
description: Classifica uma reunião comercial da Pipo Saúde em um dos 12 tipos comerciais e gera o resumo pronto para virar uma observação (nota) no HubSpot, a partir da transcrição da reunião. Use sempre que alguém colar uma transcrição de reunião comercial (cliente, Pré-Kickoff, Kickoff, consultoria de benefícios, etc.) e pedir para resumir, classificar ou gerar a observação/nota de HubSpot. Também é usada internamente pelo fluxo automático (skill processar-reunioes-automatico).
---

# Resumo de Reunião (o "cérebro")

Esta skill tem duas responsabilidades, sempre nesta ordem: **(1) classificar** a reunião em um dos 12 tipos comerciais e **(2) gerar o resumo** no template certo. É usada tanto por uma pessoa colando uma transcrição no chat quanto pelo fluxo automático.

## Entradas

Sempre exigir ou perguntar por (não presumir o que faltar):

- **Transcrição da reunião** (texto completo, gerado pelo Gemini).
- **Título do evento** do Calendar, se disponível.
- **Data da reunião** (DD/MM/AAAA).
- Opcional, quando vier do fluxo automático: **estágio do negócio no HubSpot** (sinal extra de classificação) e **lista de participantes/e-mails** do evento do Calendar.

Se faltar a transcrição, peça para colarem antes de continuar. Nunca gere um resumo a partir só do título.

Se faltar a empresa/contexto (comum no modo manual, quando o cliente é quem organizou a reunião), pergunte qual é a empresa antes de finalizar, em vez de adivinhar.

## Passo 1: Classificar em um dos 12 tipos

Cruze três sinais: **título do evento**, **conteúdo da transcrição** e, se disponível, **estágio do negócio no HubSpot**. Nenhum sinal isolado é definitivo: o título pode estar genérico ("Reunião com Cliente"), então o conteúdo da conversa manda mais.

| # | Tipo | Pistas típicas |
|---|------|-----------------|
| 1 | Reunião de Diagnóstico | Primeira conversa com o cliente; mapeia dores, plano atual, headcount, motivo da insatisfação. Ainda não há cotação em jogo. |
| 2 | Reunião de Briefing de Cotação | Cliente descreve o que precisa para a Pipo montar uma cotação: benefícios desejados (Saúde, Odontológico, Seguro de Vida, Outros Benefícios), regras de elegibilidade, coparticipação, rede desejada. |
| 3 | Reunião de Apresentação de Cotação | Pipo apresenta uma cotação/proposta já pronta (valores, planos, operadoras) ao cliente. |
| 4 | Reunião de Negociação | Discussão de condições comerciais de uma proposta já apresentada: desconto, prazo, condições de pagamento, ajustes de escopo para fechar. |
| 5 | Reunião de Follow-up Comercial | Acompanhamento de uma oportunidade em aberto que não é diagnóstico, briefing, apresentação nem negociação (ex.: alinhamento de status, tirar dúvida pontual). |
| 6 | Reunião com Parceiro | O interlocutor principal é um parceiro/corretor (não o cliente final) tratando de uma conta ou fluxo de parceria. |
| 7 | Apresentação do Time de Saúde | O time de saúde/wellness da Pipo se apresenta ou apresenta seu escopo de atuação ao cliente. |
| 8 | Apresentação da Plataforma | Demonstração do produto/plataforma da Pipo (funcionalidades, app, portal do RH). |
| 9 | Briefing de Consultoria de Benefícios | Cliente/prospect fornece contexto para a Pipo montar uma consultoria de benefícios (benchmark), não uma cotação. |
| 10 | Apresentação de Consultoria de Benefícios | Pipo apresenta o resultado da consultoria de benefícios (benchmark, recomendações). |
| 11 | Kickoff | Reunião de início de implantação após o fechamento do negócio, com o cliente. |
| 12 | Pré-Kickoff | Reunião interna (sem participante externo) preparando o Kickoff de uma empresa específica, identificada pelo nome da empresa no título do evento. |

Se o estágio do negócio no HubSpot estiver disponível, use-o como reforço (ex.: estágio de "Cotação enviada" reforça o tipo 3; estágio de "Fechado ganho" recente reforça Kickoff/Pré-Kickoff), mas não decida só por isso, pois nomes de estágio variam por pipeline e podem estar desatualizados.

**Regra de não forçar:** se depois de olhar os três sinais não der para cravar o tipo com segurança, **não invente**. Use um título temático curto que descreva o assunto real da reunião e gere o resumo no `templates/_geral.md` (assuntos, informações, dores, decisões, próximos passos, participantes). Toda reunião qualificada tem que sair com um resumo. Nunca deixe sem registrar por falta de certeza no tipo.

Declare o tipo identificado logo no topo do resumo, para o vendedor poder corrigir se a classificação estiver errada.

## Passo 2: Gerar o resumo

Use o template do tipo classificado em `templates/`:

| # | Tipo | Arquivo |
|---|------|---------|
| 1 | Reunião de Diagnóstico | `templates/01-diagnostico.md` |
| 2 | Reunião de Briefing de Cotação | `templates/02-briefing-cotacao.md` |
| 3 | Reunião de Apresentação de Cotação | `templates/03-apresentacao-cotacao.md` |
| 4 | Reunião de Negociação | `templates/04-negociacao.md` |
| 5 | Reunião de Follow-up Comercial | `templates/05-followup-comercial.md` |
| 6 | Reunião com Parceiro | `templates/06-parceiro.md` |
| 7 | Apresentação do Time de Saúde | `templates/07-apresentacao-time-saude.md` |
| 8 | Apresentação da Plataforma | `templates/08-apresentacao-plataforma.md` |
| 9 | Briefing de Consultoria de Benefícios | `templates/09-briefing-consultoria-beneficios.md` |
| 10 | Apresentação de Consultoria de Benefícios | `templates/10-apresentacao-consultoria-beneficios.md` |
| 11 | Kickoff | `templates/11-kickoff.md` |
| 12 | Pré-Kickoff | `templates/12-pre-kickoff.md` |
| — | Não identificado (fallback) | `templates/_geral.md` |

Os templates 1 e 2 (Diagnóstico e Briefing de Cotação) têm campos estruturados fixos: preencha-os quando mencionados e deixe em branco (não omita o campo) quando não identificados na transcrição; essa é a única exceção à regra geral de "omitir o que não apareceu". Nos demais templates, se uma seção inteira não tiver conteúdo real na conversa, omita a seção.

Referências de apoio em `docs/`: `docs/instrucoes-anotacoes-reunioes-pipo.md` (fonte original completa), `docs/snippet-briefing-cotacao.md` (campos do Briefing de Cotação) e `docs/exemplo-diagnostico.md` (exemplo validado, calibra nível de detalhe do Diagnóstico).

Regras que valem para **todos** os templates, sem exceção:

- **Foque no que o cliente disse, não no que a Pipo apresentou.** A nota existe para acompanhar a jornada e as decisões do cliente, não para documentar o discurso comercial de quem vende. Cite o que a Pipo apresentou em uma linha de contexto (ex.: "Pipo apresentou sua proposta de valor e capacidades técnicas"), sem detalhar pilares, funcionalidades ou cases um a um. O grosso do resumo deve ser: necessidades, reações, dúvidas, objeções, dores e decisões do cliente. Isso vale especialmente nos templates de apresentação (Time de Saúde, Plataforma, Consultoria de Benefícios) e no fallback geral, onde é fácil escorregar para narrar o pitch do vendedor.
- **Não inventar, presumir ou completar** informação que não esteja na transcrição. Se uma seção do template não tiver conteúdo real na conversa, omita a seção; não preencha com genérico.
- **Desconfiar de nomes próprios foneticamente estranhos** (operadoras, corretoras, empresas). O Gemini transcreve por voz e erra nomes próprios com frequência (ex.: "Pivida"/"P vida" no lugar de Hapvida). Se um nome não bater com uma operadora/corretora conhecida do mercado, use o contexto (outras menções na própria transcrição, grafia mais plausível) para corrigir. Se não der para ter certeza, registre o nome como aparece e sinalize a incerteza, em vez de inventar uma grafia com confiança falsa.
- Escrever de forma objetiva e organizada, adequada para colar como observação no HubSpot (títulos, macrotópicos, sem repetição).
- Priorizar o que é relevante para o processo comercial e a continuidade da oportunidade.
- **Não usar travessão (—) no meio de frases ou parágrafos.** Usar vírgula, ponto, dois-pontos, ponto e vírgula ou parênteses conforme o caso. Isso vale para o resumo gerado e para qualquer lista (ex.: participantes: usar vírgula entre nome, empresa, cargo e e-mail, não travessão).
- **Toda** observação tem seção de **Próximos passos** (ação, responsável, prazo; só quando mencionados na reunião).
- Tamanho-alvo: ~1 página; até 2 páginas somente se houver volume real de informação relevante.
- **Título em destaque** (negrito/cabeçalho): `[Tipo de Reunião] | DD/MM/AAAA`.
- **Última seção, sempre**: `Participantes da reunião`: nome, empresa, cargo/time e e-mail quando identificável a partir da transcrição ou da lista de convidados. Não inventar participantes que não apareceram.
- "Oportunidades sugeridas" (quando o template tiver essa seção): manter breve e ancorado no que apareceu de fato na reunião; não citar diferenciais que ninguém mencionou.
- Formatação: HTML simples é aceito (o corpo vai para o campo `hs_note_body` do HubSpot); use `<h3>`, `<p>`, `<ul>/<li>`, `<strong>` em vez de markdown puro quando o resumo for consumido pelo fluxo automático.

## Saída

Entregue:
1. O tipo classificado (e uma frase de confiança: certo, ou "não deu para cravar, usei resumo geral").
2. O resumo formatado, pronto para colar/enviar como nota do HubSpot.

Se estiver rodando dentro do fluxo automático (`processar-reunioes-automatico`), pare aqui e devolva esses dois itens para a skill chamadora. Não crie a nota nem envie mensagens; isso é responsabilidade da skill automática.
