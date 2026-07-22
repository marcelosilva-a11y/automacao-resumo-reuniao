# Automação de Resumos de Reunião: Pipo Saúde

Automação que lê reuniões comerciais no Google Calendar, resume a transcrição gerada pelo Gemini, registra uma observação no HubSpot (associada à empresa e aos negócios abertos) e notifica o time no Slack.

Baseado em `especificacao-automacao-resumos.md` (o documento original do time), com uma mudança de arquitetura explicada abaixo.

## Arquitetura

Diferente do documento original (que previa GitHub Actions com credenciais brutas por pessoa), esta automação roda via **scheduled tasks do Claude Code**, uma por pessoa, usando os mesmos conectores (Google Calendar, Google Drive, HubSpot, Slack) que cada pessoa já usa no dia a dia. Vantagens: nenhuma credencial bruta de API para gerenciar, autoria correta automática (cada rotina roda com a conta de quem é dona da reunião).

**Limitação importante:** scheduled tasks do Claude Code rodam enquanto o app está aberto; se estiver fechado no horário marcado, a tarefa roda no próximo lançamento do app, não exatamente às 9h05. Isso é diferente de uma execução 100% desatendida na nuvem (como seria com GitHub Actions). Na prática, isso funciona bem se cada pessoa mantém o Claude Code aberto de manhã, mas é bom o time saber disso, para não estranhar se uma rodada atrasar em um dia em que o app ficou fechado.

**Outra pegadinha real (encontrada em 22/07/2026):** a primeira execução de uma scheduled task pode travar pedindo aprovação de ferramenta (Calendar/Drive/HubSpot/Slack), mesmo com tudo conectado. Sem ninguém para responder, ela simplesmente não completa nenhum trabalho (dispara no horário, mas não cria nota nem manda mensagem, silenciosamente). É essencial adicionar as ferramentas usadas pela automação à allowlist de permissões (`.claude/settings.json` do projeto) antes de deixar a scheduled task rodando sozinha, e testar com "Run now" pelo menos uma vez para confirmar que ela completa sem travar.

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
3. **Criar uma scheduled task, às 9h05** (horário de Brasília): prompt = "Use a skill processar-reunioes-automatico para varrer as reuniões do dia anterior." Uma rodada diária é suficiente (decisão de 22/07/2026): o dedup já garante que nada é processado duas vezes, e rodar de manhã dá a noite inteira para a transcrição do Gemini ficar pronta.
4. **Adicionar as ferramentas da automação à allowlist de permissões** (`.claude/settings.json` do projeto): `search_crm_objects`, `manage_crm_objects`, `get_properties` (HubSpot), `list_events`, `get_event` (Calendar), `read_file_content`, `search_files`, `get_file_metadata` (Drive), `slack_send_message`, `slack_search_users` (Slack), `list_scheduled_tasks`. Sem isso, a scheduled task trava pedindo aprovação sem ninguém para responder.
5. **Rodar "Run now" pelo menos uma vez** para confirmar que a tarefa completa sem travar em nenhum prompt de aprovação, antes de deixá-la rodando sozinha.
6. **Manter o Claude Code aberto** perto das 9h05 (ver limitação na seção Arquitetura).

## Manutenção

- **Lista do time** (`config/time-vendas.yaml`): editar ao entrar/sair alguém do time, ou ao adicionar um domínio interno alternativo (hoje: `piposaude.com.br` e `pipo.ai`).
- **Padrão de título do Pré-Kickoff**: a skill assume `Pré-Kickoff - {Empresa}`; se o time adotar um padrão diferente, atualizar a seção correspondente em `.claude/skills/processar-reunioes-automatico/SKILL.md`.
- **Pipelines comerciais**: confirmados como `default` (Oportunidades) e `8582978` (Desenvolvimento de Negócios); não existe pipeline "Vendas" nesta conta HubSpot.
- **Templates de resumo** (`docs/instrucoes-anotacoes-reunioes-pipo.md`): se o time atualizar as regras de algum tipo de reunião, atualizar o template correspondente em `.claude/skills/resumo-reuniao/templates/`.

## Piloto antes de ativar as 8 pessoas

Antes de qualquer pessoa criar suas scheduled tasks, valide manualmente com reuniões reais: peça para rodar `processar-reunioes-automatico` uma vez, revise a nota e a mensagem de Slack geradas, ajuste o que precisar, e só depois disso replique o setup para o resto do time.
