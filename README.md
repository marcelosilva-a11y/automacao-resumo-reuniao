# Automação de Resumos de Reunião: Pipo Saúde

Automação que lê reuniões comerciais no Google Calendar, resume a transcrição gerada pelo Gemini, registra uma observação no HubSpot (associada à empresa e aos negócios abertos) e notifica o time no Slack.

Baseado em `especificacao-automacao-resumos.md` (o documento original do time), com uma mudança de arquitetura explicada abaixo.

## Arquitetura

Diferente do documento original (que previa GitHub Actions com credenciais brutas por pessoa), esta automação roda via **scheduled tasks do Claude Code**, uma por pessoa, usando os mesmos conectores (Google Calendar, Google Drive, HubSpot, Slack) que cada pessoa já usa no dia a dia. Vantagens: nenhuma credencial bruta de API para gerenciar, autoria correta automática (cada rotina roda com a conta de quem é dona da reunião).

**Limitação importante:** scheduled tasks do Claude Code rodam enquanto o app está aberto; se estiver fechado no horário marcado, a tarefa roda no próximo lançamento do app, não exatamente às 12h30/18h15. Isso é diferente de uma execução 100% desatendida na nuvem (como seria com GitHub Actions). Na prática, isso funciona bem se cada pessoa mantém o Claude Code aberto no horário comercial, mas é bom o time saber disso, para não estranhar se uma rodada atrasar em um dia em que o app ficou fechado.

## Estrutura do repositório

```
automacao-resumo-reuniao/
├── README.md                              # este arquivo
├── especificacao-automacao-resumos.md     # documento original do time
├── config/
│   └── time-vendas.yaml                   # lista das 8 pessoas + domínios internos
├── docs/
│   ├── instrucoes-anotacoes-reunioes-pipo.md  # regras originais de cada tipo de reunião
│   ├── snippet-briefing-cotacao.md            # campos do template de Briefing de Cotação
│   └── exemplo-diagnostico.md                 # exemplo validado, calibra o nível de detalhe
└── .claude/
    └── skills/
        ├── resumo-reuniao/                # o "cérebro": classifica + resume
        │   ├── SKILL.md
        │   └── templates/                 # 1 arquivo por tipo de reunião + fallback
        └── processar-reunioes-automatico/ # orquestra tudo: Calendar → Drive → cérebro → HubSpot → Slack
            └── SKILL.md
```

## Modo manual (rede de segurança)

Útil quando o cliente é quem organiza a reunião (a transcrição fica no Drive do cliente, fora do alcance da automação): cole a transcrição no chat do Claude e peça para usar a skill `resumo-reuniao`. Ela classifica o tipo de reunião e devolve o resumo pronto para colar como observação no HubSpot; se faltar contexto (qual é a empresa, por exemplo), ela pergunta antes de "inventar".

## Setup por pessoa (as 8 pessoas do time de vendas)

Cada pessoa faz isso na própria conta do Claude Code:

1. **Conectar as contas**: Google Calendar, Google Drive, HubSpot e Slack, todos com a própria conta corporativa da pessoa (não uma conta central).
2. **Clonar/abrir este repositório** no Claude Code, para que as skills (`resumo-reuniao` e `processar-reunioes-automatico`) fiquem disponíveis.
3. **Criar duas scheduled tasks** (uma para cada horário do documento original):
   - **12h30** (horário de Brasília): prompt = "Use a skill processar-reunioes-automatico para varrer minhas reuniões recentes."
   - **18h15** (horário de Brasília): mesmo prompt.
   - Não é necessário diferenciar o prompt entre as duas rodadas: a janela de tempo e o dedup já cuidam de processar cada reunião qualificada exatamente uma vez, então rodar o mesmo prompt duas vezes por dia funciona sem duplicar nem perder nada.
4. **Manter o Claude Code aberto** perto desses horários (ver limitação na seção Arquitetura).

## Manutenção

- **Lista do time** (`config/time-vendas.yaml`): editar ao entrar/sair alguém do time, ou ao adicionar um domínio interno alternativo (hoje: `piposaude.com.br` e `pipo.ai`).
- **Padrão de título do Pré-Kickoff**: a skill assume `Pré-Kickoff - {Empresa}`; se o time adotar um padrão diferente, atualizar a seção correspondente em `.claude/skills/processar-reunioes-automatico/SKILL.md`.
- **Pipelines comerciais**: confirmados como `default` (Oportunidades) e `8582978` (Desenvolvimento de Negócios); não existe pipeline "Vendas" nesta conta HubSpot.
- **Templates de resumo** (`docs/instrucoes-anotacoes-reunioes-pipo.md`): se o time atualizar as regras de algum tipo de reunião, atualizar o template correspondente em `.claude/skills/resumo-reuniao/templates/`.

## Piloto antes de ativar as 8 pessoas

Antes de qualquer pessoa criar suas scheduled tasks, valide manualmente com reuniões reais: peça para rodar `processar-reunioes-automatico` uma vez, revise a nota e a mensagem de Slack geradas, ajuste o que precisar, e só depois disso replique o setup para o resto do time.
