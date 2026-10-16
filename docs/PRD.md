# PRD: Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| Product Manager | Marcos |
| Redação | João Paulo Constantino, com apoio de IA |
| Status | Em revisão |
| Data | 2026-10-16 |
| Documentos | [RFC](./RFC.md) · [FDD](./FDD.md) · [ADRs](./adrs/README.md) |

**Convenções.** `[hh:mm] Nome` aponta para a [transcrição](../TRANSCRICAO.md). *(interpretação)* marca uma leitura posterior à reunião, registrada no documento citado ao lado. `RF`, `RNF` e `OOS` são IDs globais; os demais são deste PRD.

## 1. Resumo e contexto da feature

Os clientes B2B passam a ser avisados em menos de 10s ([09:02] Marcos) quando o status de um pedido deles muda, por uma chamada HTTPS assinada para a URL que cadastraram. É um pedido formal da Atlas Comercial, da MaxDistribuição e da Nova Cargo ([09:00] Marcos). A Atlas quer a feature "pra fim de novembro" ([09:45] Marcos), e a estimativa é de 3 sprints, com a revisão de segurança no fim ([09:47] Larissa).

## 2. Problema e motivação

- **Polling caro.** Para saber se um pedido mudou, os clientes "ficam batendo no GET /orders de tempos em tempos", o que deixa a integração "lenta e cara pra eles" ([09:00] Marcos).
- **Tempo real.** Para eles, "qualquer coisa abaixo de 10 segundos" basta, sem ficar "atualizando manualmente" ([09:02] Marcos).
- **Risco comercial.** A Atlas sinalizou que pode "migrar pro nosso concorrente" se a feature não sair "até fim do trimestre" ([09:00] Marcos).
- **Lacuna no produto.** O OMS não tem eventos nem notificação externa: `src/modules/` só tem `auth`, `customers`, `orders`, `products` e `users`, e acompanhar um pedido exige consultá-lo (`src/modules/orders/order.routes.ts`).

## 3. Público-alvo e cenários de uso

| Persona | Necessidade | Fonte |
| --- | --- | --- |
| Cliente B2B integrador (os três solicitantes) | Saber quando o status dos seus pedidos muda, sem polling | [09:00] Marcos |
| Usuário do OMS que representa o cliente | Cadastrar e gerenciar os webhooks pela API, com o login atual | [09:32] Marcos |
| Administrador (perfil ADMIN) | Reprocessar eventos que esgotaram as tentativas | [09:36] Sofia |

**Cenários**

1. **Cadastro seletivo.** A Atlas inscreve uma URL HTTPS só em SHIPPED e DELIVERED ([09:33] Marcos) e guarda a secret devolvida no cadastro ([09:31] Marcos).
2. **Notificação verificada.** Um pedido vira SHIPPED; em menos de 10s a Atlas recebe a notificação, confere a assinatura ([09:20] Sofia) e descarta repetições pelo identificador do evento ([09:25] Diego).
3. **Manutenção planejada.** O cliente fica 2h fora do ar, como já aconteceu ([09:16] Diego), e recebe o evento numa retentativa quando volta.
4. **Reprocessamento.** Um ADMIN reprocessa um evento que esgotou as tentativas, e fica registrado quem fez ([09:18] Diego, [09:36] Sofia).
5. **Secret vazada.** O cliente vaza a secret num log dele, como já ocorreu ([09:22] Diego), pede outra pela API e tem 24h para migrar ([09:21] Sofia).

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Fonte |
| --- | --- | --- | --- | --- |
| MET-01 | Notificação "em tempo real" | Tempo da mudança de status até a chegada da notificação | **< 10s**, com até 2s de espera do polling *(interpretação, [ADR-002](./adrs/ADR-002-worker-em-processo-separado-com-polling.md))* | [09:02] Marcos, [09:10] Larissa |
| MET-02 | Atender quem pediu | Clientes solicitantes integrados | **3 de 3** *(proposta deste PRD)* | [09:00] Marcos |
| MET-03 | Cumprir o prazo | Feature em produção | **Até o fim de novembro**, em 3 sprints | [09:45] Marcos, [09:47] Larissa |
| MET-04 | Tolerar cliente fora do ar | Janela coberta pelas retentativas | **14h36m** *(interpretação, [ADR-003](./adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md))* | [09:17] Diego, [09:16] Diego |
| MET-05 | Nenhuma mudança sem evento | Mudanças de status gravadas sem evento para os webhooks inscritos | **0**: "Não pode ter caso de status mudar e evento não sair" | [09:40] Bruno, [09:41] Diego |
| MET-06 | Integração deixar de ser "lenta e cara" | Consultas periódicas dos clientes à listagem de pedidos | A medir; linha de base a coletar | [09:00] Marcos |
| MET-07 | Decidir email (OOS-02) e rate limiting (OOS-03) com dados | Eventos que esgotam as tentativas; picos de notificações por cliente | A medir; linha de base a coletar | [09:37] Larissa, [09:39] Diego |

MET-02, MET-06 e MET-07 são propostas deste PRD: a reunião não fixou a meta de MET-02, e as outras duas ficam sem meta.

## 5. Escopo (incluso e fora de escopo)

### Incluso

Notificação assinada de cada mudança de status, configuração pela API, retentativas com DLQ e reprocessamento, e histórico de entregas: RF-01 a RF-15 e RNF-01 a RNF-14. A criação do pedido não notifica, porque o ponto decidido é a mudança de status ([09:40] Bruno) *(interpretação, [ADR-007](./adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md))*.

### Fora de escopo

Nenhum item abaixo é requisito. Os que ficaram em aberto viraram questões (OQ) no [RFC](./RFC.md).

| ID | Item | Situação | Fonte |
| --- | --- | --- | --- |
| OOS-01 | Webhooks de entrada, do cliente para o OMS | **Descartado**: "Só saindo da gente pra eles" | [09:02] Marcos |
| OOS-02 | Email ao cliente após falhas seguidas | **Adiado**: "Talvez próxima fase, depois que a gente medir o impacto" (OQ-03) | [09:37] Larissa |
| OOS-03 | Rate limiting de saída por cliente | **Em observação**: "observar e decidir depois" (OQ-01) | [09:39] Larissa |
| OOS-04 | Dashboard visual para o cliente | **Descartado desta feature**: "Painel é projeto separado do time de frontend" | [09:40] Larissa |
| OOS-05 | Arquivamento dos eventos entregues ("depois de 30 dias ou assim") | **Adiado**: "fora do escopo dessa feature" (OQ-05) | [09:08] Diego |
| OOS-06 | Vários workers em paralelo e ordem global | **Adiado**: "problema do futuro, não agora" (OQ-02) | [09:13] Diego |
| OOS-07 | Entrega exactly-once | **Descartado**: "exigiria coordenação dos dois lados" | [09:25] Diego |
| OOS-08 | Itens do pedido na notificação | **Descartado**: "Não manda items pra não inflar" | [09:43] Diego |
| OOS-09 | Restringir a configuração, hoje aberta a qualquer usuário autenticado | **Adiado**: "Mais pra frente a gente pode endurecer" (OQ-04) | [09:36] Marcos, [09:37] Sofia |

## 6. Requisitos funcionais

Prioridade: todos obrigatórios; a reunião não priorizou requisitos.

| ID | Requisito | Fonte |
| --- | --- | --- |
| RF-01 | Notificar o cliente quando o status de um pedido dele muda (evento `order.status_changed`) | [09:00] Marcos, [09:43] Diego |
| RF-02 | Cadastrar webhook com cliente, URL e status de interesse; a plataforma gera a secret e a devolve no cadastro. O cliente vem na requisição, não do login | [09:31] Marcos, [09:32] Larissa |
| RF-03 | Editar webhook | [09:33] Bruno |
| RF-04 | Remover webhook, que deixa de ser notificado | [09:33] Bruno |
| RF-05 | Listar os webhooks de um cliente | [09:33] Bruno |
| RF-06 | Notificar cada webhook só nos status que ele escolheu | [09:33] Marcos, [09:33] Bruno |
| RF-07 | **Derivado:** ativar e desativar webhook sem removê-lo. A reunião só cita o "estado ativo"; mudá-lo pela edição (RF-03) é *(interpretação, [FDD](./FDD.md))* | [09:21] Bruno |
| RF-08 | Assinar cada notificação com HMAC-SHA256 do corpo e a secret do webhook, para o cliente verificar origem e integridade | [09:22] Sofia, [09:19] Sofia |
| RF-09 | Rotacionar a secret pela API; a anterior vale por mais 24h e depois "morre" | [09:21] Sofia |
| RF-10 | Retentar entregas com falha, com esperas de 1m/5m/30m/2h/12h: 1 envio inicial + 5 retentativas *(interpretação, [ADR-003](./adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md))* | [09:17] Larissa |
| RF-11 | Esgotadas as tentativas, guardar o evento na DLQ "pra debug e reprocessamento" | [09:15] Diego, [09:18] Diego |
| RF-12 | Reprocessar manualmente um evento da DLQ, só com perfil ADMIN e com registro de quem fez | [09:18] Diego, [09:36] Sofia |
| RF-13 | Consultar as últimas 100 entregas de um webhook: sucesso ou falha, payload, resposta e tempo de resposta | [09:34] Marcos |
| RF-14 | Recusar URL que não seja HTTPS, com erro de validação | [09:23] Sofia |
| RF-15 | Identificar em cada notificação o evento (`X-Event-Id`), para deduplicação, e o webhook (`X-Webhook-Id`) | [09:25] Diego, [09:44] Sofia |

## 7. Requisitos não funcionais

| ID | Requisito | Fonte |
| --- | --- | --- |
| RNF-01 | **Desempenho:** notificação em menos de 10s após a mudança de status | [09:02] Marcos, [09:10] Larissa |
| RNF-02 | **Isolamento:** cliente lento ou fora do ar não trava nem desfaz mudanças de status | [09:04] Bruno |
| RNF-03 | **Consistência:** status gravado implica evento registrado para os webhooks inscritos; se o registro falhar, o status não muda | [09:06] Diego, [09:40] Bruno |
| RNF-04 | **Timeout:** sem resposta em 10s, a entrega conta como falha e vai para retentativa | [09:42] Diego |
| RNF-05 | **Limite:** até 64KB por notificação; acima disso é erro, sem truncar: o evento vai direto para a DLQ, sem bloquear o status *(interpretação, [ADR-003](./adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md))* | [09:24] Larissa, [09:23] Sofia |
| RNF-06 | **Transporte:** "TLS obrigatório", entrega só por HTTPS | [09:23] Sofia |
| RNF-07 | **Garantia:** at-least-once; o evento pode chegar repetido, e o cliente deduplica | [09:26] Larissa, [09:24] Diego |
| RNF-08 | **Ordem:** só por pedido e com um único worker, sem ordem global; sob retry, um evento pode ser ultrapassado pelo seguinte *(interpretação, PC-03 do [RFC](./RFC.md))* | [09:13] Larissa, [09:12] Diego |
| RNF-09 | **Disponibilidade:** a entrega roda fora do processo da API e não para quando a API reinicia | [09:11] Diego |
| RNF-10 | **Infraestrutura:** nada novo, só o MySQL existente | [09:07] Diego |
| RNF-11 | **Auditoria:** todo reprocessamento registra quem o pediu | [09:36] Sofia |
| RNF-12 | **Reservado:** a revisão de segurança é processo, não atributo do sistema; está em DEP-02 | [09:46] Sofia |
| RNF-13 | **Stack:** a atual, sem logger nem ferramenta nova | [09:29] Bruno, [09:30] Larissa |
| RNF-14 | **Resiliência:** janela de retentativas de 14h36m, que cobre manutenções de 2h | [09:17] Diego, [09:16] Diego |

## 8. Decisões e trade-offs principais

| ID | Decisão | Trade-off aceito | ADR |
| --- | --- | --- | --- |
| DEC-01 | Evento registrado no MySQL atual, na mesma transação da mudança de status | O status depende desse registro, e os eventos acumulam sem arquivamento | [ADR-001](./adrs/ADR-001-outbox-transacional-no-mysql.md) |
| DEC-02 | Processo separado da API busca eventos a cada 2s, em instância única | Até 2s de espera, sem escala horizontal e com ordem só por pedido | [ADR-002](./adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| DEC-03 | Retentativas em 1m/5m/30m/2h/12h, depois DLQ com reprocessamento por ADMIN | Cliente fora por mais de 14h36m depende de ação manual | [ADR-003](./adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) |
| DEC-04 | HMAC-SHA256 com secret por webhook, rotacionável com 24h de convivência | Secret recuperável guardada, duas assinaturas na rotação (a validar) e revisão de segurança | [ADR-004](./adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) |
| DEC-05 | At-least-once com `X-Event-Id` | A deduplicação fica com o cliente | [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| DEC-06 | Reuso da stack e dos padrões do projeto | Herda as limitações atuais, como autorização só por perfil | [ADR-006](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |
| DEC-07 | Notificação enxuta, sem itens, com o pedido como estava na mudança e filtrada por status ao registrar | Dados do pedido copiados em cada evento; mudar o filtro só afeta eventos futuros | [ADR-007](./adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md) |

## 9. Dependências

| ID | Dependência | Fonte |
| --- | --- | --- |
| DEP-01 | MySQL que o OMS já usa | [09:07] Diego |
| DEP-02 | Revisão de segurança da Sofia sobre "HMAC e geração de secret", de pelo menos dois dias úteis antes do deploy, dentro das 3 sprints (no lugar do RNF-12) | [09:46] Sofia, [09:47] Larissa, [09:49] Sofia |
| DEP-03 | Clientes verificando a assinatura e deduplicando pelo `X-Event-Id` | [09:20] Sofia, [09:25] Diego |
| DEP-04 | Portal do desenvolvedor documentando a integração, com a deduplicação "bem destacado" | [09:26] Marcos, [09:40] Marcos |
| DEP-05 | Usuários do OMS que representam o cliente, com o login atual | [09:32] Marcos |
| DEP-06 | Operação de um processo a mais, separado da API | [09:11] Diego |

## 10. Riscos e mitigação

Probabilidade e impacto são avaliação deste PRD, não falas da reunião; quando o risco também está no [FDD](./FDD.md), a avaliação é a mesma.

| ID | Risco | Prob. | Impacto | Mitigação | Fonte |
| --- | --- | --- | --- | --- | --- |
| R-01 | Atraso faz a Atlas migrar para o concorrente | Média | Alto | 3 sprints; Marcos confirma o prazo com a Atlas ([09:47] Marcos) | [09:00] Marcos |
| R-02 | Cliente fora do ar por mais de 14h36m perde a entrega automática | Baixa | Médio | DLQ e reprocessamento por ADMIN; o PM achou "aceitável" | [09:17] Marcos |
| R-03 | Cliente processa o mesmo evento duas vezes | Média | Médio | Mesmo identificador em toda repetição; deduplicação no portal (DEP-04) | [09:25] Diego, [09:25] Sofia |
| R-04 | Cliente vaza a secret | Baixa | Alto | Secret por webhook, rotação com 24h e revisão de segurança | [09:22] Diego, [09:21] Sofia |
| R-05 | Eventos acumulam sem arquivamento e a entrega fica lenta | Média | Médio | Leitura só dos pendentes; arquivamento adiado (OOS-05) | [09:07] Bruno, [09:08] Diego |
| R-06 | Rajada de mudanças sobrecarrega o cliente | Média | Médio | Medir o volume (MET-07); rate limiting em observação (OOS-03) | [09:38] Diego |
| R-07 | Eventos do mesmo pedido chegam fora de ordem | Baixa | Baixo | Instância única; os clientes "só querem saber se cada pedido deles mudou" | [09:13] Larissa, [09:14] Marcos |
| R-08 | Qualquer usuário autenticado gerencia webhooks de qualquer cliente | Média | Médio | Aceito "Por enquanto"; restrição adiada (OOS-09) | [09:32] Bruno, [09:37] Sofia |
| R-09 | Um endpoint lento atrasa a fila de notificações de todos os clientes, e a meta de 10s pode não valer enquanto isso | Média | Médio | Timeout de 10s por chamada, monitoramento do atraso e, se preciso, paralelizar o envio, o que depende da OQ-02 do [RFC](./RFC.md) | [09:42] Diego, [09:12] Diego |

## 11. Critérios de aceitação

Comportamento observável pelo cliente ou pelo administrador. A coluna **Cobre** liga cada critério aos requisitos e ao item do [FDD](./FDD.md) que o verifica.

| ID | Dado | Quando | Então | Cobre |
| --- | --- | --- | --- | --- |
| CA-01 | Usuário autenticado | Cadastra webhook com status de interesse | Com URL HTTPS, o webhook é criado e a resposta traz a secret gerada; com http, recusa com erro de validação | RF-02, RF-14, RNF-06 · CAT-04, CAT-05 |
| CA-02 | Webhook inscrito em SHIPPED, com os endpoints respondendo dentro do prazo (R-09) | Um pedido vira SHIPPED | A notificação chega em menos de 10s, com `X-Event-Id` e assinatura que confere com a secret | RF-01, RF-08, RF-15, RNF-01 · CAT-01, CAT-06, OBS-01 |
| CA-03 | Webhook inscrito só em SHIPPED e DELIVERED | O pedido vira PAID | Nada é enviado a ele | RF-06 · CAT-03 |
| CA-04 | Cliente em manutenção de 2h | Ele volta | Uma retentativa entrega o evento; se ficar fora toda a janela de 14h36m, o evento vai para a DLQ | RF-10, RF-11, RNF-14 · CAT-08, CAT-09 |
| CA-05 | Evento na DLQ | Um usuário sem perfil ADMIN e depois um ADMIN pedem o reprocessamento | O primeiro é negado; o segundo reentrega com o mesmo `X-Event-Id` *(interpretação, [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-x-event-id.md))* e registra quem pediu | RF-12, RNF-11 · CAT-12 |
| CA-06 | Webhook com mais de 100 entregas | O cliente consulta o histórico | Vê as 100 mais recentes, com os dados do RF-13 | RF-13 · CAT-14 |
| CA-07 | Secret recém-rotacionada | O cliente valida as notificações | Antiga e nova funcionam por 24h; depois, só a nova | RF-09 · CAT-13 |
| CA-08 | Webhook desativado ou removido | Um pedido vira status de interesse dele | Nada é enviado a ele | RF-04, RF-07 · CAT-03, CAT-11, CAT-17 |
| CA-09 | Cliente lento ou fora do ar | Um pedido dele muda de status | A mudança conclui sem esperar o cliente | RNF-02 · CAT-01, OT-03 |

## 12. Estratégia de testes e validação

- **Revisão de design:** o design da feature, com Bruno e Diego, antes de codar ([09:50] Larissa).
- **Testes automatizados:** CA-01 a CA-09, pelos itens do FDD indicados na seção 11, dentro das 3 sprints, que incluem "testes ponta a ponta" ([09:46] Larissa).
- **Revisão de segurança:** "HMAC e geração de secret", incluindo PC-02 e PC-04 do [RFC](./RFC.md), antes do deploy (DEP-02; [09:46] Sofia).
- **Validação com os clientes:** Marcos confirma o prazo com a Atlas ([09:47] Marcos), atualiza os clientes ([09:49] Marcos) e documenta a integração no portal ([09:40] Marcos).
- **Acompanhamento:** MET-01, MET-02 e MET-06 depois da entrega; MET-07 embasa a decisão sobre email e rate limiting (OOS-02, OOS-03; [09:37] Larissa, [09:39] Diego).

A reunião não combinou piloto, beta nem liberação gradual, e este PRD não os propõe.
