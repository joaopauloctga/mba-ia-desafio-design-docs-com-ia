# Design Docs com IA: Sistema de Webhooks de Notificação de Pedidos

> Enunciado original: [ENUNCIADO.md](./ENUNCIADO.md) · Transcrição da reunião: [TRANSCRICAO.md](./TRANSCRICAO.md)

<!-- README em construção: atualizado ao final de cada fase; consolidação final na fase 7. -->

## Sobre o desafio

O ponto de partida é a transcrição de uma reunião técnica de cerca de 55 minutos. Nela, tech lead, PM, engenheiros e segurança decidem como construir um **Sistema de Webhooks de Notificação de Pedidos** para um Order Management System já em produção (Node.js + TypeScript, MySQL via Prisma). Nada foi registrado além da transcrição. A tarefa é transformar essa conversa, junto com o código existente, num pacote de design docs acionável (PRD, RFC, FDD, ADRs e um tracker de rastreabilidade) sem alterar nenhuma linha da aplicação.

O difícil não é gerar texto. É garantir que cada requisito, decisão e restrição tenha origem identificável na reunião ou no código, separar o que foi decidido do que foi descartado ou adiado, e manter cada documento na sua altura, sem repetir conteúdo entre eles. A IA foi a ferramenta principal de produção. Meu papel foi o de maestro: definir o processo, escrever os prompts, revisar criticamente o que voltava e corrigir até o resultado ficar consistente.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
| --- | --- |
| **Claude Code** (Claude Opus 5.5), sessão principal | Orquestrador e revisor. Leu a transcrição e o código, montou o plano de execução em fases, preparou as instruções de cada fase e conferiu diretamente no repositório (git, arquivos, comandos) se o que foi executado batia com o plano |
| **Claude Code**, subagente Opus | Executor das fases. Recebe as instruções da fase, executa os passos e devolve um relatório que o orquestrador confere antes de seguir |
| **Scripts de validação** (Python, só biblioteca padrão, gerados pelo subagente) | Conferem cada documento contra a transcrição e o código: citações `[hh:mm] Nome`, trechos entre aspas, caminhos de arquivo, links, IDs, números canônicos e, no RFC, tamanho e vazamento de detalhe de FDD. Ficam na pasta local do plano e são testados por mutação (erros injetados numa cópia) |

## Workflow adotado

1. **Planejamento antes de escrever.** Antes de gerar qualquer documento, pedi um plano completo dividido em fases. O plano ficou numa pasta local, fora do versionamento, porque é guia de trabalho e não entregável. A primeira etapa dele foi uma pré-análise da transcrição e do código, já com origem `[hh:mm] Nome` ou caminho de arquivo em cada item:
   - decisões fechadas e requisitos;
   - itens fora de escopo e alternativas descartadas;
   - questões em aberto;
   - pontos de integração com o código;
   - ambiguidades a decidir e armadilhas de alucinação a evitar.
2. **Ordem de produção**, seguindo a sugestão do enunciado:
   setup → catálogo da transcrição → ADRs → RFC → FDD → PRD → tracker → README → revisão final.
   As decisões (ADRs) vêm primeiro porque formam o esqueleto do RFC e do FDD. O PRD vem por último entre os grandes documentos porque, com o resto pronto, vira uma consolidação em altura mais alta.
3. **Organização da interação com a IA.** Cada fase segue o mesmo ciclo:
   - eu dou o comando da fase;
   - a sessão principal lança um subagente executor com instruções detalhadas;
   - o subagente executa e entrega um relatório;
   - a sessão principal confere o resultado no repositório.
   Pontos ambíguos da reunião são decididos por mim, não pela IA, em checkpoints explícitos.
4. **Sem templates do curso.** Os documentos seguem as seções mínimas exigidas no enunciado para PRD, RFC e FDD, e o formato MADR para os ADRs.
5. **Processo documentado durante o trabalho.** Este README é atualizado ao final de cada fase, para que prompts e iterações sejam registrados quando acontecem, e não reconstruídos de memória no fim.
6. **Catálogo como base de verdade.** Antes de escrever qualquer documento, a transcrição foi catalogada linha a linha, com um ID por item e a origem `[hh:mm] Nome`, e um script confere se todas as falas estão cobertas e se cada citação existe. Os documentos só usam o que está no catálogo ou no código. O que a reunião deixou ambíguo vira checkpoint de decisão minha, com opções e recomendação, antes de entrar em qualquer documento.

**Andamento**

| Fase | Status |
| --- | --- |
| 0. Setup (branch `docs/design-docs-webhooks`, ambiente, registro do processo) | ✅ |
| 1. Extração e catálogo da transcrição | ✅ |
| 2. ADRs | ✅ |
| 3. RFC | ✅ |
| 4. FDD | ⏳ |
| 5. PRD | ⏳ |
| 6. Tracker | ⏳ |
| 7. Consolidação deste README | ⏳ |
| 8. Revisão final e entrega | ⏳ |

## Prompts customizados

### 1. Planejamento em fases

Primeiro prompt da produção. Pede um plano antes de qualquer documento, para que a IA leia o enunciado inteiro, entenda o que é avaliado e separe o trabalho em etapas verificáveis.

```text
quero q vc crie um plano completo para implementarmos esse excercicio, veja oq precisamos fazer em
@ENUNCIADO.md e crie um plano separada e em phases para a execucao completa doq precisamos entregar,
detalhes, pode criar uma pasta pra salvar esse plano + fases q vc vai criar, esses arquivos do plano
nao vao ser comitados e só vao servir de guia para nossa implementacao. pode criar o plano.
```

### 2. Delegação de fase com conferência

Define o modo de trabalho de todas as fases: um agente executa e outro confere. Isso separa quem produz de quem revisa.

```text
pode começar a fase 0, usa branch, mas lance um a gente opus pra executar e depois vc confere se foi
executada corretamente.
```

### 3. Extração e classificação da transcrição

Usado na passada linha a linha que gerou o catálogo. Impede que a IA resuma a reunião ou deduza requisitos: obriga a separar decidido de sugerido, adiado de descartado, e a registrar as ideias que foram propostas e depois substituídas (como o `customer_id` vindo do JWT), que são exatamente os pontos em que um resumo genérico erra.

```text
Você é um analista técnico extraindo requisitos de uma reunião. Leia TRANSCRICAO.md LINHA A LINHA.
Para CADA fala que contenha decisão, requisito, restrição, alternativa descartada, item fora de escopo,
questão em aberto ou referência ao código, gere uma linha de tabela com:
T-ID | [hh:mm] Nome (exatamente como na transcrição) | Categoria | Resumo de 1 linha | Documento destino.

Regras:
- Use apenas o que está escrito. Não deduza requisitos.
- Quando uma ideia for proposta e depois descartada ou substituída (ex.: customer_id do JWT em [09:31]
  substituído em [09:32]), registre AS DUAS linhas e marque a primeira como "Descartada".
- Diferencie "decidido" (alguém fechou: "Decidido", "Decisão", "Anotado", "Tá decidido") de "sugerido".
- Separe explicitamente o que foi ADIADO (próxima fase) do que foi DESCARTADO (não será feito).
- Não catalogue cumprimentos, entradas e saídas da call.
Ao final, liste as falas que você considera ambíguas ou contraditórias entre si.
```

### 4. ADR com rastreabilidade obrigatória

Aplicado a cada um dos 7 ADRs, um por vez. Impede que a IA invente uma alternativa e a apresente como discutida, escreva consequências genéricas ("aumenta a complexidade"), traga detalhe de FDD para o ADR ou apresente como fala da reunião uma interpretação que eu decidi depois.

```text
Escreva o ADR-00N (<decisão>) no formato MADR em PT-BR, usando o template da fase 2.
Fonte de verdade: TRANSCRICAO.md (conferida fala a fala), plano/insumos/catalogo-transcricao.md,
plano/insumos/ids.md, plano/insumos/decisoes-de-interpretacao.md e plano/insumos/mapa-codigo.md.

Regras inegociáveis:
1. Toda afirmação no Contexto, Decisão e Alternativas deve ter uma citação [hh:mm] Nome que exista
   na transcrição ou um caminho de arquivo real conferido no repositório (linhas do mapa do código).
2. Alternativas discutidas na reunião levam o ID global (ALT-NN), a fala de origem e o motivo do
   descarte com a fala. Alternativa que ninguém levantou só entra rotulada "plausível, não discutida".
3. Interpretação decidida depois da reunião (ex.: contagem 1 + 5 tentativas, duas assinaturas na
   rotação) aparece marcada como interpretação ou proposta a validar, nunca como fala.
4. Consequências negativas reais e específicas desta decisão. Termine com "Trade-off explícito"
   em uma frase, rotulada CONS-1.
5. NÃO inclua detalhes de implementação (DDL, colunas, JSON completo, código, matriz de erros).
   Isso é FDD. A lista de campos do payload é aceitável.
6. Decisores: só quem participou daquela decisão (Diego entrou às [09:05]; Marcos e Sofia saíram
   às [09:50]).
7. Arquivos que ainda não existem (src/worker.ts, src/modules/webhooks/*) sempre com "(novo, a criar)".
8. Números idênticos em todos os ADRs: 2s, 10s, 1m/5m/30m/2h/12h, 14h36m, 64KB, 24h.
No ADR-003, mostre a conta da janela (1m+5m+30m+2h+12h = 14h36m) e explique a leitura
"1 envio inicial + 5 retentativas" citando [09:17] Diego e o conflito com [09:48] Larissa.
Salve cada ADR antes de começar o próximo.
```

### 5. RFC conciso, com altura controlada

Usado para gerar o RFC. Impede que ele vire um FDD resumido (payload, colunas, códigos de erro), que uma questão adiada apareça como requisito e que uma leitura minha da reunião seja apresentada como decisão tomada nela.

```text
Escreva docs/RFC.md em PT-BR. Ele é um documento CONCISO (2 a 4 páginas, ~1.000-1.700 palavras)
submetido à equipe para revisão. Ele propõe e abre para revisão; os ADRs já registram as decisões
(docs/adrs/*) e o FDD vai detalhar a implementação.

Use as seções: Metadados | TL;DR | Contexto e problema | Proposta técnica | Alternativas consideradas |
Questões em aberto | Pontos para confirmação dos revisores | Impacto e riscos | Decisões relacionadas.

Restrições:
- PROIBIDO no RFC: JSON de payload, lista de colunas, matriz de erros, status codes, códigos de erro,
  trechos de código. Se precisar disso, escreva "detalhado no FDD".
- Alternativas: somente as DESCARTADAS NA REUNIÃO, cada uma com o trade-off que motivou o descarte e a
  fala de origem [hh:mm] Nome.
- Questões em aberto: somente pontos que a reunião ADIOU ou NÃO DECIDIU (rate limiting, multi-worker,
  email, endurecer CRUD, retenção). Não transforme nenhum deles em requisito.
- Pontos para confirmação: deixe explícito que NÃO foram discutidos na reunião; são leituras da análise
  que os revisores precisam validar.
- Cada bloco da proposta termina com link para o ADR correspondente.
- Revisores = participantes da reunião.
- Cada item rastreável leva seu ID visível (PROP-NN, ALT-NN, OQ-NN, PC-NN, R-NN, IMP-NN).
Depois de gerar, conte as palavras e reporte.
```

<!-- Próximos: prompts de geração do FDD, PRD e tracker (fases 4-6). -->

## Iterações e ajustes

_Em andamento: cada fase acrescenta aqui os momentos em que a IA gerou algo errado ou superficial e o que foi corrigido._

| # | Fase / documento | O que a IA gerou | Problema | Ajuste |
| --- | --- | --- | --- | --- |
| 1 | Fase 1 / execução do subagente | A primeira execução tentou gerar o catálogo inteiro da transcrição (151 falas) num único passo | Ficou cerca de 25 minutos sem gravar o catálogo nem os outros insumos. Só o script de validação apareceu no disco, segundos antes da interrupção. Percebi que o agente parecia parado e confirmei olhando o horário de modificação dos arquivos | Interrompi e relancei com instrução de escrita incremental: catálogo em 4 blocos de horário, arquivo criado com o primeiro bloco e os demais anexados, cada entregável salvo antes de começar o próximo |
| 2 | Fase 1 / catálogo vs. pré-análise | A pré-análise feita pela IA no planejamento atribuía uma fonte a cada decisão e linhas de código a cada ponto de integração | Algumas fontes apontavam para a fala da proposta, não a do fechamento (HMAC em [09:20] Sofia e secret por endpoint em [09:21] Sofia, em vez do "Decidido" de [09:22] Sofia; headers em [09:45] Diego em vez da lista de [09:44] Diego); o resumo de [09:48] Larissa era citado como apoio da contagem "1 + 5 tentativas", quando o "total 5 tentativas" dele é justamente a origem da dúvida; o redact da secret no log era atribuído a uma fala sobre o log do cliente; e havia intervalo de linhas errado em `order.status.ts` | A passada linha a linha e a conferência de cada arquivo com `sed`/`grep` expuseram as divergências. Corrigi as fontes no catálogo e no registro de IDs, refiz o mapa do código com uma seção de correções e levei 11 ambiguidades novas ao checkpoint de decisão |
| 3 | Fase 2 / ADR-001 e ADR-004 | Os ADRs de outbox e de assinatura HMAC, com contexto, alternativas e consequências | Algumas frases iam além do que foi dito. Um contra da alternativa Redis (a fila externa ficaria fora da transação do MySQL) aparecia dentro de uma alternativa marcada como discutida, mas ninguém disse isso na reunião. A transação do `changeStatus` era descrita como "a mais pesada do sistema", quando o Bruno só disse que ela "já é pesada". E a fala da Sofia, "todo cliente sério tem biblioteca pra isso", tinha virado "biblioteca em todas as linguagens que os clientes usam", texto que vinha do próprio plano | Na revisão crítica, conferi cada contra e cada paráfrase com a fala de origem. Removi o argumento sem origem e troquei as paráfrases pela citação literal |
| 4 | Fase 2 / ADR-003, ADR-004 e ADR-007 | Contexto e decisão com interpretações misturadas às falas | Três trechos apresentavam como fato algo que ninguém disse: "uma chamada sem limite de tempo prenderia o worker" (a Sofia só perguntou qual seria o timeout), "a secret não aparece em nenhuma outra resposta" e "se nenhum endpoint **ativo** quer o status" (a fala é "nenhum webhook do customer"). Além disso, a hipótese "5 envios no total" estava entre aspas, como se fosse fala | Uma checagem automática conferiu se cada trecho entre aspas existe literalmente na transcrição, e reli as frases sem citação. Troquei a primeira pela pergunta literal da Sofia, marquei as outras duas como interpretação posterior à reunião (no caso do "estado ativo", com a fala de apoio [09:21] Bruno) e tirei as aspas da hipótese |
| 5 | Fase 2 / ADR-003 vs. ADR-005 | Decisão de retry com "no máximo 6 envios por evento" | Inconsistente com o ADR-005, em que o mesmo evento (mesmo `event_id`) vai para cada endpoint inscrito, cada um com o seu próprio retry | Apareceu na leitura cruzada dos números entre os ADRs. Corrigi para "no máximo 6 envios de cada evento para cada endpoint" |
| 6 | Fase 2 / ADR-002 e script de validação | O ADR do worker citava o nome alternativo `webhook.processor.ts`; o script `validar_adrs.py`, gerado pela IA, conferia citações, seções e caminhos | O arquivo não existe e estava sem a marca "(novo, a criar)". O script não pegou porque só conferia caminhos começando com `src/`, `prisma/` ou `tests/` | Achei com um grep de nomes de arquivo entre crases. Escrevi o caminho completo com a marca e ampliei o script para nomes soltos. Um teste de mutação (erros injetados numa cópia dos ADRs) confirmou que ele passou a acusar o caso |
| 7 | Fase 3 / RFC (tamanho) | Primeiro rascunho completo do RFC | 2.830 palavras sem a tabela de metadados, 66% acima do teto de 1.700 que o próprio prompt fixava. Só a proposta tinha 741 palavras, as tabelas tinham colunas redundantes e quase toda frase levava citação duplicada | Medi com um script novo e cortei em três rodadas. Tirei a coluna "Por que foi considerada" (a motivação ficou só onde há fala de origem), a ALT-04, que era opcional e continua no ADR-002, os riscos que repetiam impactos ou questões em aberto e os detalhes de implementação que tinham escapado. Fechou em 1.698 na contagem estilo `wc -w` (1.516 sem os símbolos de Markdown das tabelas) |
| 8 | Fase 3 / RFC (fidelidade) | Proposta, questões em aberto e impactos | Frases mais fortes que a fala. Um "POST" que ninguém disse. O "overengineering" do Diego, dito sobre Redis Cluster, virou argumento contra qualquer infraestrutura. "A ordem vale por pedido" aparecia sem a ressalva do retry, e a introdução dos pontos para confirmação dizia que todos estavam nos ADRs. O impacto na transação omitia a leitura que o ADR-007 registra. Os próprios cortes de tamanho também apagaram qualificadores: "Talvez próxima fase" virou "Próxima fase", e o "por enquanto" da Sofia sumiu da autorização do CRUD | Reli cada frase contra a transcrição e cruzei com os 7 ADRs. Troquei pelas falas literais, restaurei os qualificadores e passei a reler o texto depois de cada rodada de corte, não só no fim |
| 9 | Fase 3 / RFC vs. ADR-001 | Contexto do RFC dizendo que o Bruno chamou a transação de "já pesada" | A fala é "já é pesada". A citação não literal veio do ADR-001 (L14) e tinha passado pela revisão da fase 2 | O validador do RFC passou a conferir se cada trecho entre aspas existe literalmente na transcrição. Corrigi no RFC. O agente da fase não podia alterar ADRs, então só apontou o problema; corrigi o ADR-001 na conferência da fase |

## Como navegar a entrega

| Ordem | Arquivo | Pergunta que responde | Status |
| --- | --- | --- | --- |
| 1 | [`docs/PRD.md`](./docs/PRD.md) | Por que e o quê? | ⏳ |
| 2 | [`docs/RFC.md`](./docs/RFC.md) | Como pretendemos resolver e o que ainda está em aberto? | ✅ |
| 3 | [`docs/adrs/`](./docs/adrs/README.md) | Por que decidimos exatamente assim? | ✅ |
| 4 | [`docs/FDD.md`](./docs/FDD.md) | Como construir, em detalhe? | ⏳ |
| 5 | [`docs/TRACKER.md`](./docs/TRACKER.md) | De onde veio cada coisa? | ⏳ |
