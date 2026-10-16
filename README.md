# Design Docs com IA: Sistema de Webhooks de Notificação de Pedidos

> Enunciado original: [ENUNCIADO.md](./ENUNCIADO.md) · Transcrição da reunião: [TRANSCRICAO.md](./TRANSCRICAO.md)

## Sobre o desafio

Uma empresa que opera um Order Management System em produção (Node.js + TypeScript, MySQL via Prisma) vai construir um **Sistema de Webhooks de Notificação de Pedidos**. A decisão técnica foi tomada numa reunião de cerca de 55 minutos entre tech lead, PM, engenheiros e segurança, mas nada foi registrado além da transcrição. O desafio é transformar essa transcrição e o código existente num pacote de design docs acionável o suficiente para o time começar a implementação: PRD, RFC, FDD, ADRs e um tracker de rastreabilidade, sem alterar nenhuma linha da aplicação.

A regra central é não inventar: cada requisito, decisão ou restrição precisa apontar para uma fala da reunião (`[hh:mm] Nome`) ou para um arquivo do código. Também é preciso separar o que foi decidido do que foi descartado ou adiado e manter cada documento no seu nível (produto, arquitetura, decisão, implementação), sem repetir conteúdo entre eles. A IA é a ferramenta principal de produção, e este README registra como ela foi usada: ferramentas, workflow, prompts e as correções feitas ao longo do caminho.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
| --- | --- |
| **Claude Code**, sessão principal (Claude Opus 5.5) | Orquestrador. Leu o enunciado, a transcrição e o código, montou o plano em fases, preparou as instruções de cada fase e conferiu o resultado no repositório antes do commit, com scripts próprios e releitura |
| **Claude Code**, subagentes Claude Opus | Executores. A cada fase, a sessão principal lançou um subagente que seguiu as instruções, escreveu o documento e o validador da fase e devolveu um relatório para conferência |
| **Scripts de validação em Python** (só biblioteca padrão), gerados pelos subagentes | Um por etapa: catálogo, ADRs, RFC, FDD, PRD e tracker. Conferem se cada `[hh:mm] Nome` existe na transcrição, se cada trecho entre aspas é literal, se cada caminho existe ou está marcado como novo, além de links, IDs e números. Cada um tem regras do seu documento: tamanho e ausência de detalhe de FDD no RFC, JSON válido e linhas de código citadas no FDD, ausência de detalhe técnico e metas só com números da reunião no PRD, cobertura e percentuais do enunciado no tracker. Foram testados por mutação (erros injetados numa cópia). Ficam na pasta local do plano e **não são versionados** |

## Workflow adotado

**Ordem das fases.** Antes de qualquer documento, pedi um plano em fases, que ficou numa pasta local fora do versionamento. A ordem é a sugerida no enunciado, com uma etapa a mais no começo: catalogar a transcrição fala a fala, com ID e origem `[hh:mm] Nome` em cada item, e mapear o código com caminho e linha, para que os documentos partam de uma base conferida e não de um resumo da reunião. Os ADRs vêm em seguida porque as decisões são o esqueleto do RFC e do FDD. O PRD fica por último entre os grandes documentos porque, com o resto pronto, vira uma consolidação no nível de produto. O tracker fecha a rastreabilidade.

**Andamento**

| Fase | Resultado | Status |
| --- | --- | --- |
| 0. Setup | Branch `docs/design-docs-webhooks`, plano local em fases, este README iniciado | ✅ |
| 1. Catálogo da transcrição | 151 falas cobertas, mapa do código, 32 ambiguidades levadas a decisão | ✅ |
| 2. ADRs | 7 ADRs e o índice | ✅ |
| 3. RFC | Proposta com alternativas descartadas, questões em aberto e links para os ADRs | ✅ |
| 4. FDD | 8 contratos, 11 códigos `WEBHOOK_*`, 18 pontos de integração com o código | ✅ |
| 5. PRD | 15 requisitos funcionais, 14 não funcionais, 9 itens fora de escopo | ✅ |
| 6. Tracker | 299 linhas, 262 de 262 itens cobertos | ✅ |
| 7. Consolidação deste README | Narrativa revisada contra o diário de bordo | ✅ |
| 8. Revisão final e entrega | Checklist de aceite item a item e push | ⏳ |

**Interação com a IA.** Toda fase seguiu o mesmo arranjo: eu dava o comando na sessão principal (por exemplo, "pode começar a fase 4, lança o agente separado"), ela lançava um subagente com as instruções da fase, e o subagente executava, rodava o validador e devolvia um relatório. Antes do commit, a sessão principal conferia o resultado de forma independente, com script próprio, releitura e falas checadas na transcrição, e corrigia o que estava fora do escopo do subagente, como a citação do ADR-001 apontada na fase 3 (iteração 9).

**Escrita incremental.** Depois do travamento da fase 1 (iteração 1), toda geração passou a gravar primeiro um esqueleto com os títulos e depois uma seção por vez, cada uma salva antes da próxima.

**Tracker desde a fase 2.** Cada fase deixou um rascunho com as linhas do seu documento (ADRs 45, RFC 30, FDD 112, PRD 75). A fase 6 consolidou esse rascunho contra o texto final dos documentos.

**Validação em duas camadas.** O script de cada etapa checa o que é mecânico (a fala existe, a citação é literal, o arquivo existe, o item tem linha no tracker), mas não diz se a fala sustenta a frase. Para isso houve releitura cética de cada documento contra a transcrição e, no tracker, amostragem com semente registrada, com varredura da classe inteira quando uma linha sorteada falhava.

**Checkpoints humanos.** A IA não decidiu sozinha o que a reunião deixou ambíguo.

- **Ambiguidades.** A pré-análise e o catálogo levantaram 32 pontos com mais de uma leitura, cada um com opções e recomendação. Decidi quatro: onde barrar um payload acima de 64KB (no worker, direto para a DLQ, sem travar a mudança de status), como manter a secret antiga válida por 24h na rotação (duas assinaturas no mesmo header, a validar na revisão de segurança), que erro devolver para URL sem https (um código `WEBHOOK_` próprio) e o que acontece ao remover um endpoint (soft delete, preservando histórico e DLQ). As outras 28, como a contagem "1 envio inicial + 5 retentativas" e se a criação do pedido gera evento, aprovei em bloco ("ok para o resto"). Nos documentos, essas escolhas aparecem como interpretação ou proposta, nunca como fala da reunião.
- **Tamanho.** Depois de ler o FDD commitado, pedi para enxugar (iteração 12). O PRD já foi escrito com uma referência de tamanho por causa disso.
- **Combinados do início.** Branch própria; plano local, sem commit; `ENUNCIADO.md` versionado; sem templates do curso (os documentos seguem as seções mínimas do enunciado e o formato MADR); e este README atualizado ao fim de cada fase, para registrar prompts e iterações quando aconteciam.
- **Push** só com a minha aprovação, na fase 8.

## Prompts customizados

Os dois primeiros são comandos que digitei, copiados como estão. Os outros seis são as instruções de geração que o subagente de cada fase recebeu: partem do prompt sugerido no plano daquela fase, com as restrições acrescentadas antes de executar, e estão na versão efetivamente usada. Os caminhos `plano/...` que aparecem neles são da pasta local do plano, que não é versionada.

### 1. Planejamento em fases

Pede um plano completo antes de qualquer documento, a partir do que o enunciado avalia.

```text
quero q vc crie um plano completo para implementarmos esse excercicio, veja oq precisamos fazer em
@ENUNCIADO.md e crie um plano separada e em phases para a execucao completa doq precisamos entregar,
detalhes, pode criar uma pasta pra salvar esse plano + fases q vc vai criar, esses arquivos do plano
nao vao ser comitados e só vao servir de guia para nossa implementacao. pode criar o plano.
```

### 2. Delegação de fase com conferência

Definiu o arranjo de todas as fases: um subagente executa e a sessão principal confere.

```text
pode começar a fase 0, usa branch, mas lance um a gente opus pra executar e depois vc confere se foi executada corretamente.
```

### 3. Extração e classificação da transcrição

Gerou o catálogo. Impede que a IA resuma a reunião ou deduza requisitos: obriga a separar decidido de sugerido e adiado de descartado, e a registrar ideias propostas e depois substituídas (como o `customer_id` vindo do JWT), onde um resumo genérico erra.

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

Aplicado a cada um dos 7 ADRs, um por vez. Impede alternativa inventada apresentada como discutida, consequência genérica, detalhe de FDD no ADR e interpretação decidida depois apresentada como fala da reunião.

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

Impede que o RFC vire um FDD resumido (payload, colunas, códigos de erro), que uma questão adiada vire requisito e que uma leitura da análise apareça como decisão da reunião.

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

Impede caminho inventado, import de símbolo que não é exportado (`TxClient`, `transitions`), `NotFoundError` para um código `WEBHOOK_*`, biblioteca nova e proposta apresentada como decisão.

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

Impede que detalhe de implementação (tabela, código de erro, status HTTP) suba para o documento de produto, que uma meta use número que ninguém disse na reunião e que algo descartado, adiado ou em observação volte como requisito.

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

Impede horário estimado, fala que não sustenta o item, arquivo ainda a criar como `CODIGO`, duas origens na mesma célula e origem inventada para o que não tem.

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

| # | Fase / documento | O que a IA gerou | Problema | Ajuste |
| --- | --- | --- | --- | --- |
| 1 | Fase 1 / execução do subagente | Tentativa de gerar o catálogo inteiro da transcrição (151 falas) num único passo | Cerca de 25 minutos sem gravar o catálogo nem os outros insumos; só o script de validação ficou no disco. Percebemos pelos arquivos de saída sem modificação | Interrompi e relancei com escrita incremental: catálogo em 4 blocos de horário, cada entregável salvo antes do próximo. A regra passou a valer para todas as fases |
| 2 | Fase 1 / catálogo vs. pré-análise | A pré-análise do planejamento, com uma fonte para cada decisão e linhas de código para cada integração | Fontes na fala da proposta, não na do fechamento (HMAC e secret por endpoint em [09:20] e [09:21] Sofia, em vez do "Decidido" de [09:22] Sofia); o "total 5 tentativas" de [09:48] Larissa usado como apoio da contagem "1 + 5", quando é a origem da dúvida; 14 decisões dadas como fechadas sem marcador de fechamento; intervalo de linhas errado em `order.status.ts` | A passada linha a linha e a conferência de cada arquivo com `sed`/`grep` expuseram as divergências. Fontes corrigidas no catálogo, mapa do código refeito com uma seção de correções e 11 ambiguidades novas levadas a decisão |
| 3 | Fase 2 / ADR-001 e ADR-004 | Contexto e alternativas dos ADRs de outbox e de HMAC | Frases além da fala: um contra da alternativa Redis que ninguém disse; a transação do `changeStatus` como "a mais pesada do sistema", quando o Bruno disse "já é pesada"; "todo cliente sério tem biblioteca pra isso" ampliado para "em todas as linguagens que os clientes usam", texto que vinha do próprio plano | Conferi cada contra e cada paráfrase com a fala. Removi o argumento sem origem e troquei as paráfrases pela citação literal |
| 4 | Fase 2 / ADR-003, ADR-004 e ADR-007 | Contexto e decisão com interpretações misturadas às falas | Apresentados como fato: "sem limite de tempo prenderia o worker" (a Sofia só perguntou o timeout), "a secret não aparece em nenhuma outra resposta" e "endpoint **ativo**" (a fala é "webhook do customer"). A hipótese "5 envios no total" estava entre aspas, como se fosse fala | Checagem de literalidade das aspas e releitura das frases sem citação. A pergunta literal da Sofia no lugar da primeira, as outras duas marcadas como interpretação posterior à reunião, aspas removidas da hipótese |
| 5 | Fase 2 / ADR-003 vs. ADR-005 | Retry com "no máximo 6 envios por evento" | Inconsistente com o ADR-005: o mesmo evento vai para cada endpoint inscrito, cada um com o seu retry | Achado na leitura cruzada dos números entre os ADRs. Corrigido para "6 envios de cada evento para cada endpoint" |
| 6 | Fase 2 / ADR-002 e validador | ADR citando `webhook.processor.ts`; validador dos ADRs, também gerado pela IA | Arquivo inexistente sem a marca "(novo, a criar)". O script não pegou porque só olhava caminhos começando com `src/`, `prisma/` ou `tests/` | Achado num grep de nomes de arquivo entre crases. Caminho completo com a marca e script ampliado para nomes soltos, confirmado por teste de mutação |
| 7 | Fase 3 / RFC (tamanho) | Primeiro rascunho completo do RFC | 2.830 palavras sem a tabela de metadados, 66% acima do teto de 1.700 que o próprio prompt fixava; só a proposta tinha 741 | Três rodadas de corte: coluna redundante nas alternativas, a ALT-04 (opcional, continua no ADR-002), riscos que repetiam impactos e detalhes de implementação. Fechou em 1.698 no estilo `wc -w` (1.516 sem os símbolos de Markdown) |
| 8 | Fase 3 / RFC (fidelidade) | Proposta, questões em aberto e impactos | Frases mais fortes que a fala: um "POST" que ninguém disse; o "overengineering" do Diego, dito sobre Redis Cluster, usado contra qualquer infraestrutura nova; a ordem por pedido sem a ressalva do retry. Os próprios cortes apagaram qualificadores: "Talvez próxima fase" virou "Próxima fase", e o "por enquanto" da Sofia sumiu da autorização do CRUD | Releitura frase a frase contra a transcrição e os 7 ADRs, repetida depois de cada rodada de corte. Falas literais e qualificadores restaurados |
| 9 | Fase 3 / RFC e ADR-001 | Contexto do RFC dizendo que o Bruno chamou a transação de "já pesada" | A fala é "já é pesada". A citação não literal vinha do ADR-001 e tinha passado pela revisão da fase 2 | Acusado pelo validador do RFC, que confere se cada trecho entre aspas existe literalmente na transcrição. Corrigido no RFC; o ADR-001, fora do escopo daquela fase, foi corrigido na conferência da sessão principal |
| 10 | Fase 4 / FDD (coerência interna) | FDD gerado seção a seção: fluxos, 8 contratos, matriz de erros e 18 pontos de integração | Contradições internas (o arquivo do worker "cria o Prisma client", mas o fluxo usava a instância que o processo já tem; "Só quatro arquivos existentes mudam", com sete listados), três itens sem nenhuma fonte e um trecho entre aspas que não era fala de ninguém. O validador, também gerado pela IA, dava falsos positivos, como ler `@prisma/client` como o caminho `prisma/client` | Texto corrigido e fonte acrescentada a cada item. Script ajustado e testado por mutação (13 erros injetados, 13 acusados). A assinatura HMAC do exemplo de entrega foi recalculada à parte e confere |
| 11 | Fase 4 / FDD (fidelidade) | Contratos, fluxos e riscos já sem falhas no validador | "O `customerId` vem no body, não do JWT ([09:32] Larissa)", quando ela disse "no body ou no path"; um argumento do Diego sobre vários workers aplicado ao paralelismo dentro de um worker; a rota de rotação atribuída ao Bruno; "um customer tem poucos endpoints" como fato; um risco com probabilidade exagerada; três propostas sem a marca | Fala literal, com a escolha do body marcada como proposta. Extrapolação reescrita, atribuição corrigida, suposição transformada em premissa explícita, probabilidade rebaixada e marcas acrescentadas. A leitura cruzada com os ADRs e o RFC não achou contradição |
| 12 | Fase 4 / FDD (tamanho, pedido meu) | FDD já commitado: 1.523 linhas e 14.341 palavras, 707 linhas em blocos de código (274 de TypeScript, com o corpo inteiro do publisher e do worker, e 173 de JSON, com o mesmo erro repetido em cada contrato) | Ao ler o documento, pedi: "tem muito exemplo de código [...] deixe mais enxuto". A prosa também repetia as tabelas logo abaixo e os ADRs | Enxugamento seção a seção. O TypeScript caiu para 27 linhas (assinatura do publisher, constantes do worker, `sign` e o diff do `changeStatus`), os schemas Zod viraram tabela e os contratos ganharam um único JSON de erro comum. Fechou em 951 linhas e 10.221 palavras, com 270 linhas de código. IDs, citações e a assinatura HMAC do exemplo foram preservados; o validador foi ajustado e acusou 10 de 10 erros injetados |
| 13 | Fase 5 / PRD (tamanho) | Primeiro rascunho completo do PRD | 3.433 palavras no `wc -w`, 23% acima da referência de 2.800 definida depois do FDD. Colunas de pouco valor e um "Incluso" que repetia em lista a tabela de requisitos | Quatro rodadas de corte sem tirar ID nem fonte: colunas redundantes fora, cenário já coberto por requisito removido, critérios fundidos, testes em lista. Fechou em 2.836 (2.266 sem os símbolos de Markdown) |
| 14 | Fase 5 / PRD (fidelidade) | Requisitos, critérios de aceitação e testes | Três trechos entre aspas não literais, um deles atribuindo à Sofia a expressão de uma pergunta do Marcos; "status gravado implica evento registrado", quando o filtro só registra evento se há webhook inscrito; responsáveis inventados ("Marcos e time"); o critério de "menos de 10s" ligado a testes do FDD que não medem latência. Os cortes deixaram o "Incluso" vazio e um critério truncado | `grep` de cada citação logo depois de gravar a seção e releitura das 63 falas citadas. Falas literais, "para os webhooks inscritos", atribuição removida, mapeamento corrigido. O validador novo do PRD acusou 22 de 22 erros injetados |
| 15 | Fase 6 / tracker (consolidação) | Linhas do rascunho, alimentado desde a fase 2 | Resumos que juntavam falas diferentes com uma só origem. Pior caso: a linha do cadastro, com "cliente vem na requisição, não do login", apontava para [09:31] Marcos, que disse o contrário ("Customer_id implícito do JWT"); quem corrigiu foi [09:32] Larissa. Tipos sem vocabulário fixo. O validador acusava `tests/orders.test.ts`, que o FDD cita como molde, como arquivo novo | Uma origem por linha, com sufixo (`PRD-FR-02b` aponta para [09:32] Larissa). Vocabulário de 23 tipos com legenda no topo. Script corrigido e testado por mutação (32 de 32) |
| 16 | Fase 6 / tracker (sentido) | Tracker sem falhas no validador: 287 linhas, 100% dos itens cobertos | O script confere se a fala existe, não se ela sustenta o item. Numa amostra de 30 linhas (semente 20261016), 2 decisões apontavam para [09:10] Larissa, que só diz "Worker em polling, 2s". Uma segunda amostra, de 25 linhas (semente 16102026), achou mais 2 do mesmo tipo | Varredura da classe inteira, e não só das linhas sorteadas: decisões compostas passaram para o resumo de [09:48] Larissa, ficaram só com o que a fala de fechamento diz ou ganharam linha com sufixo. Uma varredura por conceito (10s, 24h, ADMIN, JWT etc.) sinalizou 38 linhas: 13 corrigidas, 25 já sustentadas ou marcadas. Fechou com 86,6% das linhas em `TRANSCRICAO` e as 40 linhas `CODIGO` conferidas contra o código |
| 17 | Fase 7 / este README | O README escrito aos poucos, ao fim de cada fase | Na consolidação, três trechos entre aspas desta tabela não batiam com o texto registrado no diário de bordo: "criava o Prisma client" (o texto era "cria"), "só quatro arquivos mudam" (faltava "existentes") e a frase do `customerId` sem o "não do JWT". O relato do travamento da fase 1 trazia um detalhe sem registro | Conferi cada trecho entre aspas do README contra a transcrição e o diário, e cada prompt contra a versão registrada. Troquei os trechos pela forma literal e removi o detalhe sem registro |

**Total: 17 ajustes registrados, em 14 ciclos de geração → revisão → correção.** Conto como ciclo cada rodada registrada no diário de bordo em que um problema real foi encontrado e corrigido antes de seguir. Por etapa: catálogo 2 (o travamento e a revisão da pré-análise), ADRs 2 (a revisão da fase 2 e a correção do ADR-001 na conferência da fase 3), RFC 2, FDD 3 (incluindo o enxugamento), PRD 2, tracker 2 e README 1. Ficam de fora o planejamento, o setup e as três conferências da sessão principal que não acharam problema nos documentos.

## Como navegar a entrega

| Ordem | Arquivo | Pergunta que responde | Status |
| --- | --- | --- | --- |
| 1 | [`docs/PRD.md`](./docs/PRD.md) | Por que e o quê? Problema, público, escopo, requisitos e métricas | ✅ |
| 2 | [`docs/RFC.md`](./docs/RFC.md) | Como pretendemos resolver e o que ainda está em aberto? | ✅ |
| 3 | [`docs/adrs/README.md`](./docs/adrs/README.md) | Quais decisões foram tomadas e onde está cada uma? (índice e convenção) | ✅ |
| 4 | `docs/adrs/ADR-001` a `ADR-007` (links abaixo) | Por que decidimos exatamente assim? | ✅ |
| 5 | [`docs/FDD.md`](./docs/FDD.md) | Como construir, em detalhe? Fluxos, contratos, erros e integração com o código | ✅ |
| 6 | [`docs/TRACKER.md`](./docs/TRACKER.md) | De onde veio cada coisa? | ✅ |
| Origem | [`TRANSCRICAO.md`](./TRANSCRICAO.md) | O que foi dito na reunião? (não alterado) | ✅ |
| Origem | [`ENUNCIADO.md`](./ENUNCIADO.md) | O que o desafio pede? | ✅ |

ADRs: [001 Outbox no MySQL](./docs/adrs/ADR-001-outbox-transacional-no-mysql.md) · [002 Worker com polling](./docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md) · [003 Retry e DLQ](./docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) · [004 HMAC-SHA256](./docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) · [005 At-least-once](./docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) · [006 Reuso dos padrões](./docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) · [007 Payload e filtro](./docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md)
