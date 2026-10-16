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
| **Scripts de validação** (Python, só biblioteca padrão, gerados pelo subagente) | Conferem cada documento contra a transcrição e o código: citações `[hh:mm] Nome`, trechos entre aspas, caminhos de arquivo, links, IDs, números canônicos; no RFC, tamanho e vazamento de detalhe de FDD; no FDD, se cada exemplo JSON é válido, se as rotas e os códigos `WEBHOOK_*` são só os previstos e se o símbolo citado está mesmo na linha de código indicada; no PRD, se escapou nome de tabela, código de erro, status HTTP ou caminho de arquivo, se algum item fora de escopo virou requisito e se cada meta usa só números ditos na reunião; no tracker, se cada `[hh:mm] Nome` aponta para uma fala que existe, se cada caminho `CODIGO` existe e não é arquivo que a feature ainda vai criar, se todo rótulo dos quatro documentos tem linha e se as metas do enunciado (70% de `TRANSCRICAO`, 5 linhas `CODIGO`, 80% de cobertura) são atingidas. Ficam na pasta local do plano e são testados por mutação (erros injetados numa cópia) |

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
7. **Script para existência, amostra para sentido.** No tracker, o validador garante que cada fala e cada arquivo existem e que nenhum item ficou sem linha, mas não diz se a fala sustenta o item. Para isso, sorteio linhas com semente registrada e confiro cada uma na transcrição ou no código. Quando a amostra acha um problema, varro a classe inteira de linhas parecidas, e não só as sorteadas.

**Andamento**

| Fase | Status |
| --- | --- |
| 0. Setup (branch `docs/design-docs-webhooks`, ambiente, registro do processo) | ✅ |
| 1. Extração e catálogo da transcrição | ✅ |
| 2. ADRs | ✅ |
| 3. RFC | ✅ |
| 4. FDD | ✅ |
| 5. PRD | ✅ |
| 6. Tracker | ✅ |
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

### 6. FDD acionável, ancorado no código

Usado para gerar o FDD, uma seção por vez. Impede que a IA invente caminhos ou trate como importável um símbolo que não é exportado (`TxClient`, `transitions`), use `NotFoundError` para um código `WEBHOOK_*`, traga biblioteca nova ou apresente como decidido o que é proposta minha.

```text
Escreva docs/FDD.md em PT-BR. É o documento de IMPLEMENTAÇÃO: um dev deve conseguir codar a partir dele.
Base obrigatória: docs/adrs/*, docs/RFC.md, o mapa do código e as decisões de interpretação aprovadas.
NÃO contradiga nenhum ADR nem o RFC.

Regras:
1. Seção "Integração com o sistema existente": cite SOMENTE caminhos que existem (confira com ls/grep antes),
   com linha e o símbolo exato (ex.: changeStatus L126-179, requireRole L49-61). Arquivos novos vão numa
   seção separada, marcados "(novo, a criar)". Símbolo não exportado não é "reaproveitado" por import.
2. Contratos: para cada endpoint, request (headers + JSON), response de sucesso (JSON), um response de erro
   no formato real do errorMiddleware, status codes e semântica. API de configuração em camelCase;
   payload de saída para o cliente em snake_case.
3. Matriz de erros: todos os códigos novos com prefixo WEBHOOK_, indicando a classe base existente que cada
   um estende. Lembre que NotFoundError tem código fixo NOT_FOUND.
4. Tudo que NÃO foi decidido na reunião e você precisar propor deve vir marcado "(proposta de implementação)".
5. Não adicione bibliotecas novas. Não use Redis/filas/OpenTelemetry/Prometheus como solução.
6. Observabilidade: subseções explícitas "Métricas", "Logs" e "Tracing".
7. Números exatos: 2s, 10s, 1m/5m/30m/2h/12h, 14h36m, 64KB, 24h, 100.
8. Cada item rastreável leva seu ID visível (FLUXO-NN, CT-NN, ERR-NN, RES-NN, OBS-NN, INT-NN, DADOS-NN,
   CAT-NN, R-NN) e cada afirmação aponta para [hh:mm] Nome ou para um caminho real do código.
Grave o esqueleto primeiro e preencha uma seção por vez.
```

### 7. PRD na altura de produto

Usado para gerar o PRD, uma seção por vez. Impede que detalhe de implementação (tabela, código de erro, status HTTP) suba para o documento de produto, que a IA invente metas como "99,9% de entrega" e que algo descartado, adiado ou deixado em observação na reunião volte como requisito.

```text
Escreva docs/PRD.md em PT-BR consolidando a visão de PRODUTO. ADRs, RFC e FDD já existem: use-os como
fonte, mas suba a altura. Proibido no PRD: nome de tabela, coluna, classe, código de erro, JSON,
status code HTTP. Decisão técnica entra como uma linha com link para o ADR.

Obrigatório:
- As 12 seções do enunciado, nesta ordem.
- ≥ 8 requisitos funcionais, cada um com a fala de origem [hh:mm] Nome; requisito derivado marcado como tal.
- Objetivos com meta numérica usando SOMENTE números ditos na reunião (<10s, 3 clientes, fim de novembro,
  janela de retry de 14h36m). Não invente percentuais de SLA.
- Fora de escopo: liste o que foi DESCARTADO, ADIADO ou fica EM OBSERVAÇÃO (email, rate limiting,
  dashboard, inbound, arquivamento, exactly-once, multi-worker...), com a situação de cada um.
  Nenhum item fora de escopo pode reaparecer como requisito.
- Riscos em tabela com Probabilidade, Impacto e Mitigação.
- Decisões: uma linha por ADR com link, sem repetir o conteúdo.
- Conciso: tabelas diretas, sem repetir em prosa o que já está na tabela.
Grave o esqueleto primeiro e preencha uma seção por vez. Antes de entregar, liste qualquer frase sua
que NÃO tenha origem na transcrição ou no código.
```

### 8. Tracker com uma origem por linha

Usado para montar o tracker, um documento por vez. Impede que a IA estime horários ou aponte uma fala que não sustenta o item, cite como `CODIGO` um arquivo que a feature ainda vai criar, junte duas origens numa célula ou invente uma origem para o que não tem.

```text
Monte docs/TRACKER.md a partir do rascunho incremental e varrendo docs/PRD.md, docs/RFC.md, docs/FDD.md
e docs/adrs/*.md. Para CADA rótulo (RF, RNF, OOS, MET, CA, ALT, OQ, PC, CT, ERR, INT, CAT...) e cada ADR,
garanta uma linha no formato | ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |.
- Localização TRANSCRICAO: exatamente "[hh:mm] Nome" de uma fala que EXISTE e que SUSTENTA o item.
  Abra a transcrição e confira; não estime o horário.
- Localização CODIGO: somente caminhos que existem. Arquivos a criar (src/worker.ts,
  src/modules/webhooks/*) NUNCA são fonte CODIGO.
- Uma origem por linha. Proposta de implementação é ancorada na fala ou no arquivo que a motiva e
  marcada como proposta no Conteúdo.
- Se não encontrar origem para um item, NÃO invente: liste-o à parte em "Itens sem origem" para
  revisarmos (provável remoção do documento).
Grave um documento por vez. No fim, rode o validador e sorteie 25 linhas para conferir a semântica.
```

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
| 10 | Fase 4 / FDD (coerência interna) | O FDD gerado seção a seção, com modelo de dados, fluxos, 8 contratos, matriz de erros e 18 pontos de integração | Partes do texto se contradiziam ou não tinham origem. O arquivo do worker "criava o Prisma client", enquanto o fluxo do worker usava a instância que o processo já tem. A seção de integração dizia "só quatro arquivos mudam" e listava sete. Três itens da resiliência e da matriz de erros estavam sem nenhuma fonte, e um trecho entre aspas ("arquivos do código base") não era fala de ninguém. O próprio validador, também gerado pela IA, acusava falsos positivos, como ler `@prisma/client` como o caminho `prisma/client` | Corrigi o texto, acrescentei a fala ou a linha de código que sustenta cada item e ajustei o script. Testei o validador por mutação (13 erros injetados, 13 acusados) e conferi à parte que a assinatura HMAC do exemplo de entrega é o valor real calculado sobre aquele corpo com aquela secret |
| 11 | Fase 4 / FDD (fidelidade) | Contratos, fluxos e riscos já sem falhas no validador | A releitura cética achou frases que iam além da reunião. "O `customerId` vem no body ([09:32] Larissa)", quando a Larissa disse "no body ou no path" e o body foi escolha do FDD. Um argumento do Diego sobre vários workers aplicado ao paralelismo dentro de um worker. A rota de rotação atribuída ao Bruno, quando o endpoint é da Sofia. "Um customer tem poucos endpoints" como fato. E três propostas sem a marca (paginação da listagem, replay único por linha da DLQ e mudança no redact do log) | Troquei pela fala literal e marquei a escolha como proposta de implementação. Também reescrevi a extrapolação, corrigi a atribuição, transformei a suposição em premissa explícita, com alternativa, e acrescentei as marcas que faltavam. A leitura cruzada com os 7 ADRs e o RFC não achou contradição |
| 12 | Fase 4 / FDD (tamanho) | O FDD já validado e commitado: 1.523 linhas e 14.341 palavras, 707 delas em blocos de código. Eram 274 linhas de TypeScript, com o corpo inteiro do publisher e do ciclo do worker, os schemas Zod, as classes de erro e a composição de rotas, e 173 de JSON, com o mesmo JSON de erro repetido em cada contrato e um histórico de entregas com dois itens completos | Ao ler o documento, achei que tinha "muito exemplo de código". Ele parecia código pronto em vez de especificação, e a prosa repetia o que as tabelas logo abaixo e os ADRs já diziam | Enxuguei uma seção por vez. O TypeScript ficou com 27 linhas: a assinatura de `publishWebhookEvent`, as constantes do worker, a função `sign` e o diff do `changeStatus`. Os schemas Zod viraram tabela, as classes de erro ficaram só na matriz, e a composição de rotas, os scripts e a limpeza dos testes foram para a tabela de integração. Os contratos ganharam um único exemplo de erro no início, o CT-06 ficou com um item e sobrou uma consulta SQL. Resultado: 951 linhas e 10.221 palavras no `wc -w` (8.536 sem os símbolos das tabelas), com 270 linhas de código. IDs, citações, números e a assinatura HMAC do exemplo foram preservados. A checagem do validador que exigia um JSON de erro por contrato passou a exigir o exemplo comum e uma linha 4xx em cada tabela, e o teste por mutação acusou os 10 erros injetados |
| 13 | Fase 5 / PRD (tamanho) | O primeiro rascunho completo do PRD, gerado seção a seção | 3.433 palavras no `wc -w`, 23% acima da referência de cerca de 2.800 que defini para não repetir o problema do FDD. Havia colunas de pouco valor nas tabelas, um "Incluso" que repetia em lista a tabela de requisitos e uma tabela de testes de quatro colunas | Cortei em quatro rodadas sem tirar nenhum ID nem fonte. A categoria dos requisitos não funcionais virou prefixo, as colunas redundantes saíram, a persona do PM foi para as dependências e o cenário de investigação saiu, porque o requisito e o critério já o cobrem. O critério de URL http foi fundido ao de cadastro, e a seção de testes virou lista. Fechou em 2.836 palavras (2.266 sem os símbolos de Markdown das tabelas) |
| 14 | Fase 5 / PRD (fidelidade) | Requisitos, critérios de aceitação e estratégia de testes já sem falhas no validador | Três trechos entre aspas não eram literais, e um deles atribuía à Sofia uma expressão que era da pergunta do Marcos. O requisito de consistência dizia que toda mudança de status gera evento, mas o filtro por status só registra o evento quando há webhook inscrito. A estratégia de testes inventava responsáveis ("Marcos e time") e ampliava a fala da Larissa: "este pacote de documentos" no lugar do "doc de design da feature". O mapeamento dos critérios de aceitação para os testes do FDD apontava itens que não testam aquilo, como o critério de "menos de 10s" ligado a testes que não medem latência. Os próprios cortes de tamanho deixaram o "Incluso" sem conteúdo e um critério com a frase truncada | Conferi cada trecho entre aspas na transcrição logo depois de gravar a seção. Também reli as 63 falas citadas no contexto e cada item do FDD apontado. Troquei pelas falas literais, acrescentei "para os webhooks inscritos", tirei a atribuição inventada, corrigi o mapeamento (com a métrica de latência do FDD) e reescrevi o "Incluso" numa frase. Um validador novo para o PRD (altura, números das metas, fora de escopo longe dos requisitos) passou no teste por mutação: 22 erros injetados, 22 acusados |
| 15 | Fase 6 / tracker (consolidação do rascunho) | As linhas do rascunho, alimentado desde a fase 2 com uma origem por linha | Em várias linhas, o resumo juntava partes de falas diferentes, e a única origem apontada não sustentava tudo. No pior caso, o requisito de cadastro dizia que o cliente vem na requisição, e não do login, e apontava para [09:31] Marcos, que disse o contrário ("Customer_id implícito do JWT"); quem corrigiu foi [09:32] Larissa. O mesmo padrão aparecia no reprocessamento só por ADMIN, no `X-Webhook-Id`, na recusa de URL http e nos headers de saída. O tipo também variava ("Objetivo Técnico", "Critério de Aceitação", "Métrica (proposta, sem meta)"). O validador novo, também gerado pela IA, acusava como arquivo novo o `tests/orders.test.ts`, que o FDD cita só como molde | Separei cada origem numa linha com sufixo (`PRD-FR-02b` aponta para [09:32] Larissa) e fixei um vocabulário de 23 tipos, com legenda no topo do tracker. No script, a lista de arquivos novos passou a ter só os caminhos marcados "(novo". O teste por mutação acusou 32 de 32 erros injetados, cada um pela regra certa: fala inexistente, caminho falso, `src/worker.ts` como `CODIGO`, ID duplicado, cabeçalho alterado, rótulo sem linha, `A-xx` no texto, entre outros |
| 16 | Fase 6 / tracker (amostragem semântica) | O tracker já sem falhas no validador: 287 linhas, 86% `TRANSCRICAO` e 100% dos itens cobertos | O script confere se a fala existe, não se ela sustenta o item. Numa amostra de 30 linhas (semente 20261016), 2 decisões apontavam para [09:10] Larissa, que só diz "Worker em polling, 2s", sem falar de processo separado nem de instância única. Uma segunda amostra, de 25 linhas (semente 16102026), achou mais 2 do mesmo tipo: o contrato de cadastro com o `customerId` no body, ancorado na fala do Marcos que falava em JWT, e a verificação da assinatura atribuída ao Diego | Varri a classe inteira, e não só as linhas sorteadas. As decisões que juntam vários elementos passaram a apontar para o resumo de [09:48] Larissa, que todos confirmaram, ou ganharam linha com sufixo; o ADR-003 e o ADR-007 ficaram só com o que a fala de fechamento diz. Uma varredura por conceito (10s, 24h, ADMIN, assinatura, JWT etc.) nas linhas `TRANSCRICAO` sinalizou 38. Corrigi 13 delas: os blocos de proposta do RFC, o UUID atribuído à Larissa, o https posto na conta de falas que não falam dele (num objetivo do FDD e num critério do PRD) e 7 linhas em que o 64KB indo direto para a DLQ ou a janela de 14h36m estavam sem a marca *(interpretação)*. As outras 25 já traziam a marca ou tinham o núcleo sustentado pela fala. Conferi também as 40 linhas `CODIGO` contra as linhas de código citadas. Resultado: 299 linhas, 86,6% `TRANSCRICAO`, 40 `CODIGO` e 262 de 262 itens cobertos. Nenhum item ficou sem origem, e nenhum documento precisou de correção de âncora |

## Como navegar a entrega

| Ordem | Arquivo | Pergunta que responde | Status |
| --- | --- | --- | --- |
| 1 | [`docs/PRD.md`](./docs/PRD.md) | Por que e o quê? | ✅ |
| 2 | [`docs/RFC.md`](./docs/RFC.md) | Como pretendemos resolver e o que ainda está em aberto? | ✅ |
| 3 | [`docs/adrs/`](./docs/adrs/README.md) | Por que decidimos exatamente assim? | ✅ |
| 4 | [`docs/FDD.md`](./docs/FDD.md) | Como construir, em detalhe? | ✅ |
| 5 | [`docs/TRACKER.md`](./docs/TRACKER.md) | De onde veio cada coisa? | ✅ |
