# Instruções para estruturação das anotações das reuniões

> Fonte original da Pipo, referenciada na seção 9 da especificação técnica. Copiada aqui para rastreabilidade. Os templates em `.claude/skills/resumo-reuniao/templates/` são a versão operacional (usada pela skill); se o time atualizar este documento, atualize os templates junto.

Ao receber a transcrição de uma reunião, identifique primeiro o tipo de reunião com base nas categorias já definidas neste projeto. Em seguida, produza a anotação seguindo as orientações abaixo.

## Regras gerais

- Escreva de forma objetiva, organizada e adequada para registro no HubSpot.
- Não invente, presuma ou complete informações que não estejam disponíveis na transcrição.
- Utilize títulos e macrotópicos para facilitar a leitura.
- Evite repetir a mesma informação em diferentes partes da anotação.
- Priorize as informações relevantes para o processo comercial, para a tomada de decisão do cliente e para a continuidade da oportunidade.
- Todas as anotações devem conter uma seção de próximos passos, com as ações definidas, os respectivos responsáveis e os prazos, quando mencionados.
- O resumo deve ter, preferencialmente, o equivalente a uma página de Word. Somente quando houver um volume maior de informações realmente relevantes, poderá chegar ao equivalente a duas páginas de Word.

## Título da anotação

Toda anotação deve começar com um título que represente a temática ou o tipo da reunião, acompanhado da data em que ela aconteceu.

Exemplos:

**Pré-Kickoff | 20/07/2026**

**Apresentação da Plataforma | 20/07/2026**

**Briefing de Cotação | 20/07/2026**

O título deve aparecer em uma fonte ou tamanho um pouco maior do que o restante da observação no HubSpot, para ficar visualmente destacado.

## Participantes da reunião

Ao final de todas as anotações, depois de todos os resumos, tópicos e próximos passos, inclua a seção:

**Participantes da reunião:**

Liste todas as pessoas que participaram da reunião e que possam ser identificadas na transcrição. Quando possível, inclua também a empresa, o cargo ou o time de cada participante, e também o email se achar.

Essa deve ser sempre a última informação da observação.

Não invente nomes, cargos ou participantes. Caso não seja possível identificar algum dado, não o inclua.

## Reunião não identificada

Caso não seja possível identificar com segurança o tipo de reunião a partir da transcrição, não force uma classificação.

Nesse caso, utilize um título que represente a temática principal discutida na reunião e faça um resumo geral e organizado dos principais assuntos abordados.

O resumo geral deve destacar:

- Principais assuntos discutidos
- Informações relevantes compartilhadas
- Dores, necessidades ou dúvidas mencionadas
- Decisões e alinhamentos realizados
- Próximos passos, responsáveis e prazos
- Participantes da reunião

O objetivo é garantir que nenhuma reunião fique sem registro no HubSpot, mesmo quando ela não se encaixar claramente em um dos tipos previamente definidos.

## Reunião de Briefing de Cotação

Utilize como base o snippet de briefing de cotação disponibilizado neste projeto (ver `docs/snippet-briefing-cotacao.md`).

Mantenha os campos e a ordem apresentados no snippet. Quando uma informação não for mencionada na transcrição, mantenha o respectivo campo em branco.

As seções relacionadas aos benefícios devem aparecer somente quando fizerem parte do escopo da cotação. Por exemplo, se o cliente não solicitar uma cotação de seguro de vida, não inclua a seção de Seguro de Vida. A mesma regra vale para plano odontológico, plano de saúde e outros benefícios.

Depois das informações sobre os benefícios, acrescente:

**Principais objetivos do cliente com o estudo de mercado:**

Registre o que o cliente espera analisar, comparar, reduzir, melhorar ou alcançar por meio da cotação.

**Próximos passos:**

Registre as entregas combinadas, informações pendentes, responsáveis e prazos mencionados.

**Participantes da reunião:**

Liste os participantes identificados na transcrição.

## Reunião de Diagnóstico

Inicie as anotações com os seguintes campos relacionados ao plano de saúde. Quando alguma dessas informações não estiver disponível, mantenha o campo em branco.

**Operadora atual:**

**Corretora atual:**

**Data de renovação:**

**Número de vidas:**

Depois, divida as anotações nos seguintes macrotópicos. As seções de dores devem ser organizadas **por temática, em subtópicos** (não em parágrafo único), para facilitar a leitura.

**Dores e características com a corretora atual:**

Registre tanto os problemas quanto as características do serviço prestado pela corretora atual: modelo de remuneração, escopo de trabalho operacional, postura (proativa ou reativa), suporte a casos graves / concierge, serviços incluídos no pacote, tecnologia, comunicação e relacionamento. Ou seja, além das dores, caracterize o que a corretora atual entrega hoje.

**Dores e características do plano de saúde e demais benefícios:**

Registre tanto os problemas quanto as características do plano e dos benefícios: sinistralidade e break-even, maiores sinistros, estrutura de subsídio, tempo e histórico de contrato, ticket médio, telemedicina e usabilidade, coparticipação, benefícios adicionais (ex.: gympass/total pass) e a experiência dos colaboradores. Inclua aqui as caracterizações do plano (como a estrutura de subsídio), e não no contexto geral.

**Dores e contexto geral:**

Faça um compilado das demais dores, necessidades, objetivos, prioridades e características importantes do cenário do cliente. É aqui que entra o contexto organizacional — por exemplo, redução de quadro, metas estratégicas do CEO, perfil e cultura da empresa, e a expectativa central do cliente em relação a uma nova corretora.

**Pontos de atenção e oportunidades de negociação:**

Registre os fatos de negociação relevantes (concorrentes na disputa e suas propostas, prazos críticos de renovação e comunicação) e, de forma **breve**, sugestões de oportunidades — sempre ancoradas no que foi efetivamente discutido na reunião. Não invente diferenciais ou argumentos que não tenham aparecido na conversa. Mantenha esta seção enxuta (poucas sugestões, direto ao ponto).

**Próximos passos:**

Registre o que o vendedor, o cliente e outras pessoas envolvidas ficaram responsáveis por fazer, com responsáveis e prazos quando mencionados.

**Participantes da reunião:**

Liste os participantes identificados na transcrição.

## Reunião de Apresentação de Cotação

Produza um resumo objetivo contendo:

**Cenários apresentados:**

Resuma as principais opções, valores ou alternativas apresentadas.

**Percepções e feedbacks do cliente:**

Registre como o cliente reagiu à cotação, o que agradou, o que não fez sentido e quais opções despertaram maior interesse.

**Principais insights e comentários:**

Registre dúvidas, objeções, comparações, restrições, pedidos de ajustes e critérios de decisão mencionados.

**Próximos passos:**

Registre ajustes solicitados, novas análises, informações pendentes, responsáveis e prazos.

**Participantes da reunião:**

Liste os participantes identificados na transcrição.

## Reunião de Negociação

Faça um resumo dos principais pontos negociados, incluindo valores, subsídios, perks, condições comerciais, contrapartidas, concessões solicitadas, condições aprovadas ou recusadas e eventuais impedimentos para o fechamento.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Reunião de Follow-up Comercial

Faça um resumo do andamento da oportunidade, destacando atualizações do processo de decisão, dúvidas, pendências, novos envolvidos, concorrentes, possíveis bloqueios, prazo esperado para a decisão e nível de interesse demonstrado pelo cliente.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Reunião com Parceiro

Faça um resumo dos principais assuntos discutidos entre a Pipo, o cliente e o parceiro.

Destaque apresentações realizadas, informações compartilhadas, dúvidas respondidas, condições discutidas, negociações e posicionamentos do cliente ou do parceiro.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Apresentação do Time de Saúde

Resuma os principais pontos apresentados pelo time de saúde e registre as dúvidas, comentários, necessidades, interesses e feedbacks do cliente em relação ao modelo de atendimento, programas de saúde, uso de dados e suporte aos colaboradores.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Apresentação da Plataforma

Resuma as principais funcionalidades apresentadas e registre as dúvidas, necessidades operacionais, integrações, funcionalidades de maior interesse, feedbacks e possíveis limitações identificadas pelo cliente.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Briefing de Consultoria de Benefícios

Registre as informações coletadas para a elaboração da consultoria e, principalmente, os temas, comparações, recortes, indicadores e pontos que o cliente pediu para aparecer no estudo.

Finalize com as informações pendentes, próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Apresentação da Consultoria de Benefícios

Resuma os principais resultados apresentados e registre os insights, comentários, dúvidas, feedbacks e prioridades apontados pelo cliente durante a apresentação.

Destaque oportunidades ou temas que o cliente demonstrou interesse em aprofundar.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua os participantes da reunião.

## Reunião de Kickoff

Faça um resumo dos principais alinhamentos realizados para o início da parceria, incluindo contexto, participantes dos times, responsabilidades, cronograma, etapas da implantação, informações pendentes, expectativas do cliente e pontos de atenção.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua a seção final com todos os participantes da reunião.

## Reunião de Pré-Kickoff

Reunião interna da Pipo, sem participação do cliente. Identificada pelo nome do evento (o time deve seguir um padrão no título, incluindo o nome da empresa).

Faça um resumo geral da passagem de bastão entre o time comercial e os demais times da Pipo.

Registre o contexto do cliente, histórico da negociação, produtos contratados, principais dores, expectativas, promessas ou condições comerciais acordadas, stakeholders, riscos e pontos de atenção para a implantação e o relacionamento.

Finalize com os próximos passos, responsáveis e prazos.

Depois, inclua a seção final com todos os participantes da reunião.

---

## Anexo — Exemplo de referência (Reunião de Diagnóstico)

O exemplo abaixo foi validado pelo time e serve como referência de formato e nível de detalhe para o tipo Diagnóstico. Note a organização das dores por subtópicos temáticos, a caracterização (não só as dores) da corretora e do plano, o contexto organizacional na seção geral, e a seção enxuta de pontos de atenção e oportunidades.

**Reunião de Diagnóstico | 20/07/2026**

**Operadora atual:** SulAmérica (saúde, odonto e vida no mesmo contrato)
**Corretora atual:** Gallagher (que adquiriu a DASA)
**Data de renovação:** 1º de setembro de 2026
**Número de vidas:** ~450 vidas (titulares + dependentes); ~350 funcionários

**Dores e características com a corretora atual:**
- Modelo de remuneração: fee mensal direto à corretora (~R$19/vida), para evitar incentivo de comissão atrelada à fatura.
- Falta de trabalho operacional: RH precisa que a corretora assuma a corretagem do dia a dia (auditoria de contas, aprovação de senhas), pois o time interno é enxuto.
- Ausência de postura proativa: a corretora apresenta dados nas reuniões, sem trazer estratégias junto.
- Falta de concierge / médico de referência para direcionar casos graves.
- Serviços atuais no pacote: inclui teleatendimento (Conexa) dentro do fee.

**Dores e características do plano de saúde e demais benefícios:**
- Sinistralidade: 71% (break-even de 80%).
- Maior sinistro: dependente de ex-colaborador em extensão.
- Subsídio escalonado: titular 100%, 1º dependente 50%, 2º 20%, 3º em diante integral pelo colaborador.
- Contrato antigo (~10 anos), ticket médio abaixo da média de mercado.
- Telemedicina com ~30% de usabilidade, preferida por não ter coparticipação.

**Dores e contexto geral:**
- Redução de quadro (de 500 para 350 funcionários).
- Meta estratégica do CEO de reduzir para ~100 colaboradores, o que preocupa a sustentação das condições.
- Perfil de empresa que valoriza inovação; colaboradores com longa casa.
- Expectativa central: corretora que assuma o direcionamento de casos graves e o papel de concierge.

**Pontos de atenção e oportunidades de negociação:**
- Concorrente na disputa (proposta com mais serviços por valor menor por vida).
- Prazo apertado de renovação e de comunicação de troca de corretora.
- Oportunidades sugeridas: reforçar o modelo de fee e o diferencial de concierge/time de saúde; avaliar assumir a telemedicina dentro do benefício. Breve e ancorado no que foi discutido.

**Próximos passos:**
- Ações com responsáveis e prazos conforme combinado na reunião.

**Participantes da reunião:**
- Lista de participantes com empresa, cargo/time e e-mail quando identificáveis.
