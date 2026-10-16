# Tracker de Rastreabilidade

Liga cada item rotulado do [PRD](./PRD.md), do [RFC](./RFC.md), dos [ADRs](./adrs/README.md) e do [FDD](./FDD.md) à sua origem: uma fala da [transcrição](../TRANSCRICAO.md) ou um arquivo que já existe no código.

**Como ler.** Cada linha é um item, identificado pelo rótulo que ele tem no próprio documento (conversão abaixo). `Fonte` diz de onde ele vem: `TRANSCRICAO`, com o `[hh:mm] Nome` da fala que sustenta o item (em geral a que fechou a decisão), ou `CODIGO`, com o caminho de um arquivo que existe hoje; linha e símbolo, quando ajudam, ficam no resumo. Cada linha tem uma origem só. Quando um item tem duas origens relevantes, ganha uma segunda linha com sufixo (`FDD-INT-01b`). Os arquivos marcados (novo, a criar) nos documentos, como `src/worker.ts`, `src/modules/webhooks/*`, a migration nova e `tests/webhooks.test.ts`, nunca são fonte `CODIGO`: os itens sobre eles apontam para a fala que os motivou. No resumo, *(proposta)* marca proposta de implementação, *(interpretação)* marca leitura posterior à reunião e *(plausível, não discutida)* marca alternativa que ninguém levantou; as três apontam para a fala ou o arquivo que as motiva.

**Legenda: rótulo no documento → ID no tracker, e o tipo**

- **PRD** (`docs/PRD.md`): `MET` → `PRD-MET`, Métrica · `OOS` → `PRD-OOS`, Fora de Escopo · `RF` → `PRD-FR`, Requisito Funcional · `RNF` → `PRD-NFR`, Requisito Não Funcional (o `RNF-12`, reservado, é Restrição) · `DEC` → `PRD-DEC`, Decisão · `DEP` → `PRD-DEP`, Dependência · `R` → `PRD-RISK`, Risco · `CA` → `PRD-AC`, Critério de Aceite.
- **RFC** (`docs/RFC.md`): `PROP` → `RFC-PROP`, Proposta · `ALT` → `RFC-ALT`, Alternativa Descartada · `OQ` → `RFC-OQ`, Questão em Aberto · `PC` → `RFC-PC`, Ponto para Confirmação · `IMP` → `RFC-IMP`, Impacto · `R` → `RFC-RISK`, Risco.
- **ADRs** (`docs/adrs/ADR-00N-*.md`): `ADR-00N`, Decisão · `ADR-006a` a `ADR-006i` (padrões reaproveitados, tabela do ADR-006), Integração · `ADR-00N-ALT-n` (alternativa n), Alternativa Descartada · `ADR-00N-CONS-1` (trade-off explícito), Trade-off.
- **FDD** (`docs/FDD.md`): `OT` → `FDD-OT`, Objetivo · `DADOS` → `FDD-DADOS`, Modelo de Dados · `FLUXO` → `FDD-FLUXO`, Fluxo · `CT` → `FDD-CONTRATO`, Contrato · `ERR` → `FDD-ERR`, Erro · `RES` → `FDD-RES`, Resiliência · `OBS` → `FDD-OBS`, Observabilidade · `INT` → `FDD-INT`, Integração · `CAT` → `FDD-CAT`, Critério de Aceite · `R` → `FDD-RISK`, Risco.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-MET-01 | docs/PRD.md | Métrica | Tempo da mudança de status até a chegada da notificação abaixo de 10s, com até 2s de espera do polling *(interpretação)* | TRANSCRICAO | [09:02] Marcos |
| PRD-MET-02 | docs/PRD.md | Métrica | 3 de 3 clientes solicitantes integrados (Atlas Comercial, MaxDistribuição e Nova Cargo): meta *(proposta)* deste PRD, não dita na reunião | TRANSCRICAO | [09:00] Marcos |
| PRD-MET-03 | docs/PRD.md | Métrica | Feature em produção até o fim de novembro, prazo pedido pela Atlas | TRANSCRICAO | [09:45] Marcos |
| PRD-MET-03b | docs/PRD.md | Métrica | Estimativa de 3 sprints para a entrega, com a revisão de segurança no fim | TRANSCRICAO | [09:47] Larissa |
| PRD-MET-04 | docs/PRD.md | Métrica | Janela coberta pelas retentativas: 14h36m, leitura das "quase 15 horas" *(interpretação)* | TRANSCRICAO | [09:17] Diego |
| PRD-MET-05 | docs/PRD.md | Métrica | 0 mudanças de status gravadas sem evento para os webhooks inscritos | TRANSCRICAO | [09:40] Bruno |
| PRD-MET-06 | docs/PRD.md | Métrica | Consultas periódicas dos clientes à listagem de pedidos: a medir, sem meta *(proposta)* | TRANSCRICAO | [09:00] Marcos |
| PRD-MET-07 | docs/PRD.md | Métrica | Eventos que esgotam as tentativas e picos por cliente: a medir, sem meta, base para decidir email e rate limiting *(proposta)* | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-01 | docs/PRD.md | Fora de Escopo | Webhooks de entrada, do cliente para o OMS: descartado ("Só saindo da gente pra eles") | TRANSCRICAO | [09:02] Marcos |
| PRD-OOS-02 | docs/PRD.md | Fora de Escopo | Email ao cliente após falhas seguidas: adiado ("Talvez próxima fase") | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de Escopo | Rate limiting de saída por cliente: em observação ("observar e decidir depois") | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de Escopo | Dashboard visual para o cliente: descartado desta feature, é projeto do time de frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-05 | docs/PRD.md | Fora de Escopo | Arquivamento dos eventos entregues (~30 dias): adiado, "fora do escopo dessa feature" | TRANSCRICAO | [09:08] Diego |
| PRD-OOS-06 | docs/PRD.md | Fora de Escopo | Vários workers em paralelo e ordem global: adiado ("problema do futuro") | TRANSCRICAO | [09:13] Diego |
| PRD-OOS-07 | docs/PRD.md | Fora de Escopo | Entrega exactly-once: descartada | TRANSCRICAO | [09:25] Diego |
| PRD-OOS-08 | docs/PRD.md | Fora de Escopo | Itens do pedido na notificação: descartado ("pra não inflar") | TRANSCRICAO | [09:43] Diego |
| PRD-OOS-09 | docs/PRD.md | Fora de Escopo | Restringir a configuração, hoje aberta a qualquer usuário autenticado: adiado ("Mais pra frente") | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Notificar o cliente quando o status de um pedido dele muda (evento `order.status_changed`) | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL e status de interesse; a plataforma gera a secret e a devolve no cadastro | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02b | docs/PRD.md | Requisito Funcional | O cliente do webhook vem na requisição, não do login (o JWT é do usuário operador) | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover webhook, que deixa de ser notificado | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Listar os webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Notificar cada webhook só nos status que ele escolheu | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Ativar e desativar webhook sem removê-lo: derivado do "estado ativo"; mudá-lo pela edição é *(interpretação)* | TRANSCRICAO | [09:21] Bruno |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Assinar cada notificação com HMAC-SHA256 do corpo e a secret do webhook | TRANSCRICAO | [09:22] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Rotacionar a secret pela API; a anterior vale por mais 24h e depois morre | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Retentar entregas com falha em 1m/5m/30m/2h/12h, lido como 1 envio inicial + 5 retentativas *(interpretação)* | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Esgotadas as tentativas, guardar o evento na DLQ | TRANSCRICAO | [09:15] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Reprocessar manualmente, por endpoint admin, um evento da DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-FR-12b | docs/PRD.md | Requisito Funcional | Reprocessamento só com perfil ADMIN e com registro de quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-13 | docs/PRD.md | Requisito Funcional | Consultar as últimas 100 entregas de um webhook: sucesso ou falha, payload, resposta e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-14 | docs/PRD.md | Requisito Funcional | Recusar URL que não seja HTTPS, com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-FR-15 | docs/PRD.md | Requisito Funcional | Identificar o evento em cada notificação (`X-Event-Id`), para deduplicação | TRANSCRICAO | [09:25] Diego |
| PRD-FR-15b | docs/PRD.md | Requisito Funcional | Identificar o webhook em cada notificação (`X-Webhook-Id`) | TRANSCRICAO | [09:44] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Desempenho: notificação em menos de 10s após a mudança de status | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Isolamento: cliente lento ou fora do ar não trava nem desfaz mudanças de status | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Consistência: status gravado implica evento registrado para os inscritos; se o registro falhar, o status não muda | TRANSCRICAO | [09:06] Diego |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Timeout: sem resposta em 10s, a entrega conta como falha e vai para retentativa | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Limite de 64KB por notificação; acima é erro, sem truncar; ir direto para a DLQ é *(interpretação)* | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Transporte: TLS obrigatório, entrega só por HTTPS | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Garantia at-least-once: o evento pode chegar repetido, e o cliente deduplica | TRANSCRICAO | [09:26] Larissa |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Ordem só por pedido e com um único worker; a ultrapassagem sob retry é *(interpretação)* | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Disponibilidade: a entrega roda fora do processo da API e não para quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Infraestrutura: nada novo, só o MySQL existente | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-11 | docs/PRD.md | Requisito Não Funcional | Auditoria: todo reprocessamento registra quem o pediu | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-12 | docs/PRD.md | Restrição | ID reservado: a revisão de segurança antes do deploy é processo, não atributo do sistema, e está em DEP-02 | TRANSCRICAO | [09:46] Sofia |
| PRD-NFR-13 | docs/PRD.md | Requisito Não Funcional | Stack: a atual, sem logger nem ferramenta nova | TRANSCRICAO | [09:29] Bruno |
| PRD-NFR-14 | docs/PRD.md | Requisito Não Funcional | Resiliência: janela de retentativas de 14h36m *(interpretação)*, que cobre manutenções de 2h | TRANSCRICAO | [09:17] Diego |
| PRD-DEC-01 | docs/PRD.md | Decisão | Evento registrado no MySQL atual, na mesma transação da mudança de status (ADR-001) | TRANSCRICAO | [09:48] Larissa |
| PRD-DEC-02 | docs/PRD.md | Decisão | Processo separado da API busca eventos a cada 2s (ADR-002) | TRANSCRICAO | [09:48] Larissa |
| PRD-DEC-02b | docs/PRD.md | Decisão | Instância única, com ordem garantida só por pedido (ADR-002) | TRANSCRICAO | [09:13] Larissa |
| PRD-DEC-03 | docs/PRD.md | Decisão | Retentativas em 1m/5m/30m/2h/12h, depois DLQ com reprocessamento por ADMIN (ADR-003) | TRANSCRICAO | [09:48] Larissa |
| PRD-DEC-04 | docs/PRD.md | Decisão | HMAC-SHA256 com secret por webhook, rotacionável com 24h de convivência (ADR-004) | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | At-least-once com `X-Event-Id`; a deduplicação fica com o cliente (ADR-005) | TRANSCRICAO | [09:26] Larissa |
| PRD-DEC-06 | docs/PRD.md | Decisão | Reuso da stack e dos padrões do projeto (ADR-006) | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | docs/PRD.md | Decisão | Notificação com o pedido como estava na mudança de status, renderizada ao registrar (snapshot) (ADR-007) | TRANSCRICAO | [09:52] Bruno |
| PRD-DEC-07b | docs/PRD.md | Decisão | Filtro por status aplicado ao registrar o evento: sem inscrito, nada é registrado (ADR-007) | TRANSCRICAO | [09:34] Bruno |
| PRD-DEC-07c | docs/PRD.md | Decisão | Notificação enxuta, sem os itens do pedido (ADR-007) | TRANSCRICAO | [09:43] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | MySQL que o OMS já usa | TRANSCRICAO | [09:07] Diego |
| PRD-DEP-02 | docs/PRD.md | Dependência | Revisão de segurança da Sofia sobre HMAC e geração de secret, de pelo menos dois dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-03 | docs/PRD.md | Dependência | Clientes deduplicando pelo `X-Event-Id` do lado deles | TRANSCRICAO | [09:25] Diego |
| PRD-DEP-03b | docs/PRD.md | Dependência | Clientes verificando a assinatura do lado deles | TRANSCRICAO | [09:20] Sofia |
| PRD-DEP-04 | docs/PRD.md | Dependência | Portal do desenvolvedor documentando a integração, com a deduplicação "bem destacado" | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-05 | docs/PRD.md | Dependência | Usuários do OMS que representam o cliente, com o login atual | TRANSCRICAO | [09:32] Marcos |
| PRD-DEP-06 | docs/PRD.md | Dependência | Operação de um processo a mais, separado da API | TRANSCRICAO | [09:11] Diego |
| PRD-RISK-01 | docs/PRD.md | Risco | Atraso faz a Atlas migrar para o concorrente (Média/Alto) | TRANSCRICAO | [09:00] Marcos |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente fora do ar por mais de 14h36m perde a entrega automática; o PM achou "aceitável" (Baixa/Médio) | TRANSCRICAO | [09:17] Marcos |
| PRD-RISK-03 | docs/PRD.md | Risco | Cliente processa o mesmo evento duas vezes (Média/Médio) | TRANSCRICAO | [09:25] Diego |
| PRD-RISK-04 | docs/PRD.md | Risco | Cliente vaza a secret, como já aconteceu num log dele (Baixa/Alto) | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-05 | docs/PRD.md | Risco | Eventos acumulam sem arquivamento e a entrega fica lenta (Média/Médio) | TRANSCRICAO | [09:07] Bruno |
| PRD-RISK-06 | docs/PRD.md | Risco | Rajada de mudanças de status sobrecarrega o cliente (Média/Médio) | TRANSCRICAO | [09:38] Diego |
| PRD-RISK-07 | docs/PRD.md | Risco | Eventos do mesmo pedido chegam fora de ordem (Baixa/Baixo) | TRANSCRICAO | [09:13] Larissa |
| PRD-RISK-08 | docs/PRD.md | Risco | Qualquer usuário autenticado gerencia webhooks de qualquer cliente, aceito "Por enquanto" (Média/Médio) | TRANSCRICAO | [09:37] Sofia |
| PRD-RISK-09 | docs/PRD.md | Risco | Endpoint lento atrasa a fila de notificações de todos os clientes, e a meta de 10s pode não valer enquanto isso (Média/Médio) | TRANSCRICAO | [09:42] Diego |
| PRD-AC-01 | docs/PRD.md | Critério de Aceite | Cadastro válido cria o webhook e a resposta traz a secret gerada pela plataforma | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-01b | docs/PRD.md | Critério de Aceite | Cadastro com URL http é recusado com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-AC-02 | docs/PRD.md | Critério de Aceite | Pedido vira status inscrito e, com os endpoints respondendo dentro do prazo, a notificação chega em menos de 10s (identificador e assinatura vêm de PRD-FR-15 e PRD-FR-08) | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-03 | docs/PRD.md | Critério de Aceite | Mudança para status fora do filtro do webhook não gera notificação para ele | TRANSCRICAO | [09:33] Marcos |
| PRD-AC-04 | docs/PRD.md | Critério de Aceite | Cliente em manutenção de 2h recebe por retentativa; fora a janela toda de 14h36m *(interpretação)*, o evento vai para a DLQ | TRANSCRICAO | [09:16] Diego |
| PRD-AC-05 | docs/PRD.md | Critério de Aceite | Reprocessamento negado a quem não é ADMIN; o ADMIN reentrega com o mesmo `X-Event-Id` *(interpretação)* e fica registrado | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-06 | docs/PRD.md | Critério de Aceite | Histórico de um webhook com mais de 100 entregas mostra as 100 mais recentes | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-07 | docs/PRD.md | Critério de Aceite | Depois da rotação, secret antiga e nova funcionam por 24h; depois, só a nova | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-08 | docs/PRD.md | Critério de Aceite | Webhook desativado ou removido não é notificado | TRANSCRICAO | [09:33] Bruno |
| PRD-AC-09 | docs/PRD.md | Critério de Aceite | Cliente lento ou fora do ar não atrasa a mudança de status | TRANSCRICAO | [09:04] Bruno |
| RFC-PROP-01 | docs/RFC.md | Proposta | Captura: evento gravado na outbox dentro da transação do `changeStatus`; commit e rollback valem para status e evento | TRANSCRICAO | [09:06] Diego |
| RFC-PROP-02 | docs/RFC.md | Proposta | Despacho: worker separado da API, com entry-point `src/worker.ts` (novo, a criar), que lê os pendentes a cada 2s | TRANSCRICAO | [09:48] Larissa |
| RFC-PROP-03 | docs/RFC.md | Proposta | Entrega segura: HMAC-SHA256 do corpo no `X-Signature`, secret por endpoint e rotação com a antiga válida por 24h | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-03b | docs/RFC.md | Proposta | URL só HTTPS, e payload acima do limite é erro, não truncamento | TRANSCRICAO | [09:23] Sofia |
| RFC-PROP-04 | docs/RFC.md | Proposta | Confiabilidade: at-least-once com `X-Event-Id`, retry 1m/5m/30m/2h/12h, DLQ em tabela separada e replay só por ADMIN | TRANSCRICAO | [09:48] Larissa |
| RFC-PROP-05 | docs/RFC.md | Proposta | Configuração e API: cadastrar webhook com URL e lista de status, com a secret gerada pela plataforma | TRANSCRICAO | [09:31] Marcos |
| RFC-PROP-05b | docs/RFC.md | Proposta | Editar, remover e listar os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| RFC-PROP-05c | docs/RFC.md | Proposta | Ver as últimas 100 entregas de um webhook | TRANSCRICAO | [09:34] Marcos |
| RFC-PROP-05d | docs/RFC.md | Proposta | O customer vem na requisição, não do JWT | TRANSCRICAO | [09:32] Larissa |
| RFC-PROP-06 | docs/RFC.md | Proposta | Padrões do projeto: webhook como módulo igual aos outros, em `src/modules/webhooks/` (novo, a criar), com `AppError`, códigos de erro, error middleware, Pino e Zod (`requireRole`, UUID e Prisma por processo: ADR-006f, ADR-006i e ADR-006h) | TRANSCRICAO | [09:30] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa Descartada | HTTP síncrono no `changeStatus`: cliente lento travaria mudanças de status; cliente fora do ar não justifica rollback | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa Descartada | Redis Streams ou outra fila externa: mais infraestrutura, "overengineering" para um time pequeno | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa Descartada | Trigger de banco: MySQL sem LISTEN/NOTIFY, avisar o worker exigiria improviso; o polling de 2s já atende | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa Descartada | Retry indefinido: evento "pendurado pra sempre se o cliente sumiu" | TRANSCRICAO | [09:15] Diego |
| RFC-ALT-09 | docs/RFC.md | Alternativa Descartada | Exactly-once: exige coordenação dos dois lados e é muito mais complexo | TRANSCRICAO | [09:25] Diego |
| RFC-OQ-01 | docs/RFC.md | Questão em Aberto | Rate limiting de saída por cliente: "observar e decidir depois" | TRANSCRICAO | [09:39] Larissa |
| RFC-OQ-02 | docs/RFC.md | Questão em Aberto | Vários workers sem perder a ordem por pedido (partição por `order_id` ou lock pessimista): "problema do futuro" | TRANSCRICAO | [09:13] Diego |
| RFC-OQ-03 | docs/RFC.md | Questão em Aberto | Email ao cliente após falhas seguidas: "Talvez próxima fase", depois de medir o impacto | TRANSCRICAO | [09:37] Larissa |
| RFC-OQ-04 | docs/RFC.md | Questão em Aberto | Endurecer a autorização do CRUD de webhooks: "Mais pra frente a gente pode endurecer" | TRANSCRICAO | [09:37] Sofia |
| RFC-OQ-04b | docs/RFC.md | Questão em Aberto | Hoje `User` não tem vínculo com `Customer` (model `User`, L25-38): qualquer usuário autenticado gerencia webhooks de qualquer customer | CODIGO | prisma/schema.prisma |
| RFC-OQ-05 | docs/RFC.md | Questão em Aberto | Retenção da outbox, arquivando as entregues "depois de 30 dias ou assim": fora do escopo da feature | TRANSCRICAO | [09:08] Diego |
| RFC-PC-01 | docs/RFC.md | Ponto para Confirmação | Contagem lida como 1 envio inicial + 5 retentativas (14h36m, perto das "quase 15 horas"); como 5 envios no total, 2h36m *(interpretação)* | TRANSCRICAO | [09:17] Diego |
| RFC-PC-02 | docs/RFC.md | Ponto para Confirmação | Formato da assinatura: duas assinaturas, nova e antiga, durante as 24h da rotação, e timestamp do envio fora do HMAC, a validar na revisão de segurança *(proposta)* | TRANSCRICAO | [09:21] Sofia |
| RFC-PC-03 | docs/RFC.md | Ponto para Confirmação | Mesmo com um único worker, um evento em backoff pode ser ultrapassado pelo seguinte do mesmo pedido: limitação conhecida *(proposta)* | TRANSCRICAO | [09:12] Diego |
| RFC-PC-04 | docs/RFC.md | Ponto para Confirmação | Forma de guardar a secret decidida na revisão de segurança; secret mostrada só na criação e na rotação *(proposta)* | TRANSCRICAO | [09:46] Sofia |
| RFC-PC-04b | docs/RFC.md | Ponto para Confirmação | O HMAC exige a secret recuperável, ao contrário da senha, guardada só como hash (`passwordHash`, L28) | CODIGO | prisma/schema.prisma |
| RFC-PC-05 | docs/RFC.md | Ponto para Confirmação | Replay recebe o id da DLQ, que fica exposto no log e no histórico de entregas, sem endpoint novo de listagem *(proposta)* | TRANSCRICAO | [09:18] Diego |
| RFC-IMP-01 | docs/RFC.md | Impacto | A transação do `changeStatus` ganha a leitura dos webhooks inscritos e a inserção na outbox | TRANSCRICAO | [09:40] Bruno |
| RFC-IMP-01b | docs/RFC.md | Impacto | O contrato de `PATCH /orders/:id/status` (`router.patch('/:id/status'`, L19-23) não muda | CODIGO | src/modules/orders/order.routes.ts |
| RFC-IMP-02 | docs/RFC.md | Impacto | Um processo a mais para operar, consultando o MySQL a cada 2s | TRANSCRICAO | [09:11] Diego |
| RFC-IMP-03 | docs/RFC.md | Impacto | Clientes deduplicam pelo `X-Event-Id` | TRANSCRICAO | [09:25] Diego |
| RFC-IMP-03b | docs/RFC.md | Impacto | Clientes verificam a assinatura do lado deles | TRANSCRICAO | [09:20] Sofia |
| RFC-IMP-03c | docs/RFC.md | Impacto | Documentação da integração no portal do desenvolvedor | TRANSCRICAO | [09:26] Marcos |
| RFC-RISK-01 | docs/RFC.md | Risco | Outbox acumula eventos e o worker fica lento; mitigação: índices, leitura só dos pendentes e OQ-05 | TRANSCRICAO | [09:07] Bruno |
| RFC-RISK-02 | docs/RFC.md | Risco | Worker parado: eventos acumulam sem entrega, mas o evento gravado não se perde | TRANSCRICAO | [09:06] Diego |
| RFC-RISK-03 | docs/RFC.md | Risco | Eventos do mesmo pedido fora de ordem, por retry ou por mais de um worker | TRANSCRICAO | [09:12] Diego |
| RFC-RISK-04 | docs/RFC.md | Risco | Vazamento de secret pelo cliente; mitigação: secret por endpoint e rotação | TRANSCRICAO | [09:22] Diego |
| RFC-RISK-05 | docs/RFC.md | Risco | Cliente fora do ar por mais de 14h36m perde a entrega automática; DLQ e replay manual, aceito pelo PM | TRANSCRICAO | [09:17] Marcos |
| RFC-RISK-06 | docs/RFC.md | Risco | Atraso faz a Atlas migrar para o concorrente; mitigação: 3 sprints e prazo confirmado com a Atlas | TRANSCRICAO | [09:00] Marcos |
| ADR-001 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Decisão | Padrão outbox no MySQL existente, com o evento inserido na mesma transação da mudança de status, atômica com ela | TRANSCRICAO | [09:48] Larissa |
| ADR-001-ALT-1 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Alternativa Descartada | Disparo HTTP síncrono no `changeStatus`: "qualquer cliente lento vai travar mudança de status pra outros pedidos" | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-2 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Alternativa Descartada | Redis Streams ou outra fila externa: mais infraestrutura, "overengineering" para um time pequeno | TRANSCRICAO | [09:07] Diego |
| ADR-001-ALT-3 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Alternativa Descartada | Gravar o evento depois do commit, fora da transação *(plausível, não discutida)*: "perde a garantia toda" | TRANSCRICAO | [09:41] Diego |
| ADR-001-CONS-1 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Trade-off | Mudança de status acoplada ao sucesso da inserção na outbox e tabela sem arquivamento, em troca de consistência sem infra nova | TRANSCRICAO | [09:40] Bruno |
| ADR-002 | docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md | Decisão | Worker em processo separado da API, em polling de 2s (a instância única está no ADR-002-CONS-1) | TRANSCRICAO | [09:48] Larissa |
| ADR-002-ALT-1 | docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md | Alternativa Descartada | Trigger de banco ou notificação reativa: MySQL sem LISTEN/NOTIFY, e a trigger não avisa processo externo | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-2 | docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md | Alternativa Descartada | Worker dentro do processo da API: "se a API reinicia, perde o worker" | TRANSCRICAO | [09:11] Diego |
| ADR-002-ALT-3 | docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md | Alternativa Descartada | Vários workers em paralelo (adiada): perde a ordem; partição por `order_id` ou lock é "problema do futuro" | TRANSCRICAO | [09:13] Diego |
| ADR-002-CONS-1 | docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md | Trade-off | Até 2s de espera, consultas periódicas e um único worker (ordem só por pedido), em troca de simplicidade sem infra nova | TRANSCRICAO | [09:13] Larissa |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Decisão | Retry com backoff 1m/5m/30m/2h/12h, lido como 1 envio inicial + 5 retentativas (14h36m) *(interpretação)*, e depois a DLQ | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-1 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Alternativa Descartada | Retry indefinido: o evento pode "ficar pendurado pra sempre se o cliente sumiu" | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-2 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Alternativa Descartada | 3 tentativas: mataria o evento em cerca de 30 minutos, sem cobrir a manutenção de duas horas | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-3 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Alternativa Descartada | DLQ como status "failed" na própria outbox: a tabela separada deixa a outbox limpa e serve de evidência | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-1 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Trade-off | Cliente fora por mais de 14h36m depende de replay manual, sem aviso, em troca de uma janela finita e previsível | TRANSCRICAO | [09:17] Marcos |
| ADR-004 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo no `X-Signature`, secret por endpoint gerada pela plataforma, rotação com a antiga válida por 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-004-ALT-1 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa Descartada | Secret global da plataforma: "Senão se vaza uma, vaza tudo" | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-2 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa Descartada | Confiar só no TLS, sem assinatura *(plausível, não discutida)*: não prova ao cliente a origem nem a integridade | TRANSCRICAO | [09:19] Sofia |
| ADR-004-ALT-3 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa Descartada | Uma única assinatura durante a rotação *(plausível, não discutida)*: o cliente não teria tempo de migrar | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-4 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa Descartada | Assinar o timestamp junto com o corpo *(plausível, não discutida)*: descarte por *(interpretação)* da decisão literal, HMAC só sobre o corpo; a pergunta vai para a revisão de segurança | TRANSCRICAO | [09:22] Sofia |
| ADR-004-CONS-1 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Trade-off | Secret recuperável, duas assinaturas por 24h e revisão de segurança, em troca de vazamento isolado e verificação padrão | TRANSCRICAO | [09:46] Sofia |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | At-least-once com `X-Event-Id` para deduplicação do lado do cliente | TRANSCRICAO | [09:26] Larissa |
| ADR-005-ALT-1 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Alternativa Descartada | Exactly-once: "exigiria coordenação dos dois lados e fica muito mais complexo" | TRANSCRICAO | [09:25] Diego |
| ADR-005-ALT-2 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Alternativa Descartada | At-most-once, sem reenvio *(plausível, não discutida)*: contradiz o retry com backoff decidido | TRANSCRICAO | [09:17] Larissa |
| ADR-005-CONS-1 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Trade-off | A deduplicação fica com o cliente ("joga responsabilidade pro cliente"), em troca de uma entrega que pode reenviar sem risco | TRANSCRICAO | [09:25] Sofia |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso máximo: padrão de módulos, `AppError` e códigos de erro, error middleware, Pino e schemas Zod; webhook como "módulo igual aos outros" | TRANSCRICAO | [09:30] Larissa |
| ADR-006a | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | Módulo `src/modules/webhooks/` (novo, a criar) no molde de `src/modules/orders/`: controller, service, repository, routes e schemas | CODIGO | src/modules/orders/order.controller.ts |
| ADR-006b | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | Erros `WEBHOOK_*` como subclasses de `AppError`, no modelo de `InvalidStatusTransitionError` e `InsufficientStockError` (L45-63) | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006c | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | `errorMiddleware` sem alteração: já trata `AppError` (L15-24) e devolve o formato de erro atual | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006d | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | Logger Pino reaproveitado na API e no worker, sem logger novo | CODIGO | src/shared/logger/index.ts |
| ADR-006e | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | Schemas Zod do módulo com o middleware `validate` (L11-37) | CODIGO | src/middlewares/validate.middleware.ts |
| ADR-006f | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | `requireRole('ADMIN')` (L49-61) no replay da DLQ | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-006g | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | `publishWebhookEvent(tx, ...)` chamado dentro da transação do `changeStatus` (L126-179), sem mudar o construtor do `OrderService` | CODIGO | src/modules/orders/order.service.ts |
| ADR-006h | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | Client do Prisma (`createPrismaClient` e singleton `prisma`, L4-10): o worker cria a sua instância, com a mesma `DATABASE_URL` | CODIGO | src/config/database.ts |
| ADR-006i | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Integração | IDs UUID `@db.Char(36)` (L26 e L75) também nas tabelas novas | CODIGO | prisma/schema.prisma |
| ADR-006-ALT-1 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa Descartada | Injetar o repository de webhook no `OrderService`: "função pura recebendo o tx" basta | TRANSCRICAO | [09:41] Diego |
| ADR-006-ALT-2 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa Descartada | Worker usar o mesmo PrismaClient da API: "PrismaClient é por processo" | TRANSCRICAO | [09:30] Bruno |
| ADR-006-ALT-3 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa Descartada | ID auto-incremental na outbox: "segue o padrão do resto do projeto. Tudo é uuid" | TRANSCRICAO | [09:51] Larissa |
| ADR-006-ALT-4 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa Descartada | Stack própria para os webhooks *(plausível, não discutida)*: contradiz o reuso máximo | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CONS-1 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | Herdar as limitações dos padrões atuais e acoplar pedidos a webhooks, em troca de nenhuma dependência nova | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CONS-1b | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | Limitação herdada: `NotFoundError` fixa o código `NOT_FOUND` (L27-31), por isso `WEBHOOK_NOT_FOUND` exige subclasse própria de `AppError` | CODIGO | src/shared/errors/http-errors.ts |
| ADR-007 | docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md | Decisão | Payload do evento renderizado na inserção na outbox (snapshot); formato enxuto e filtro na inserção estão em ADR-007-ALT-2 e ADR-007-ALT-3 | TRANSCRICAO | [09:52] Bruno |
| ADR-007-ALT-1 | docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md | Alternativa Descartada | Guardar só o `order_id` e renderizar no envio: "Senão tem caso esquisito" se o pedido mudar | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-2 | docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md | Alternativa Descartada | Payload com os items do pedido: "Não manda items pra não inflar" | TRANSCRICAO | [09:43] Diego |
| ADR-007-ALT-3 | docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md | Alternativa Descartada | Filtrar na hora do envio: filtrar na inserção "Economiza linha na tabela" | TRANSCRICAO | [09:34] Bruno |
| ADR-007-CONS-1 | docs/adrs/ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md | Trade-off | Dados duplicados, leitura a mais na transação e filtro só para eventos futuros, em troca de eventos fiéis ao momento da mudança | TRANSCRICAO | [09:52] Larissa |
| FDD-OT-01 | docs/FDD.md | Objetivo | Latência do commit até a 1ª tentativa: até 2s de polling mais a fila do lote; ponta a ponta abaixo de 10s | TRANSCRICAO | [09:02] Marcos |
| FDD-OT-02 | docs/FDD.md | Objetivo | Atomicidade: 0 mudanças de status commitadas sem a linha na outbox para cada endpoint ativo inscrito | TRANSCRICAO | [09:40] Bruno |
| FDD-OT-03 | docs/FDD.md | Objetivo | Isolamento: 0 chamadas HTTP no `changeStatus`; a transação ganha só 1 leitura e 1 inserção em lote | TRANSCRICAO | [09:04] Bruno |
| FDD-OT-04 | docs/FDD.md | Objetivo | Retry finito: até 6 envios de cada evento para cada endpoint (1 + 5 retentativas *(interpretação)*), numa janela de 14h36m; depois, DLQ | TRANSCRICAO | [09:17] Larissa |
| FDD-OT-05 | docs/FDD.md | Objetivo | Tempo máximo por chamada: 10s por tentativa; acima disso, falha com retry | TRANSCRICAO | [09:42] Diego |
| FDD-OT-06 | docs/FDD.md | Objetivo | Payload até 64KB; acima disso é erro, que vai para a DLQ sem retry e sem bloquear a mudança de status *(interpretação)* | TRANSCRICAO | [09:24] Larissa |
| FDD-OT-07 | docs/FDD.md | Objetivo | 100% das entregas assinadas com HMAC-SHA256 | TRANSCRICAO | [09:22] Sofia |
| FDD-OT-07b | docs/FDD.md | Objetivo | 100% das URLs em `https` | TRANSCRICAO | [09:23] Sofia |
| FDD-OT-08 | docs/FDD.md | Objetivo | Sem stack nova: 0 dependências npm e 0 serviços de infraestrutura novos | TRANSCRICAO | [09:29] Bruno |
| FDD-DADOS-01 | docs/FDD.md | Modelo de Dados | Enum `WebhookOutboxStatus` com PENDING, PROCESSING, DELIVERED e FAILED (pendente, processando, entregue, falhou) | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-02 | docs/FDD.md | Modelo de Dados | `webhook_endpoints`: url, secret, secret anterior e validade, events, active, deletedAt e FK para customer | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-03 | docs/FDD.md | Modelo de Dados | `webhook_outbox`: uma linha por evento e endpoint, `eventId` compartilhado, payload snapshot, attempts, nextAttemptAt e índice (status, createdAt) | TRANSCRICAO | [09:06] Diego |
| FDD-DADOS-03b | docs/FDD.md | Modelo de Dados | `orderId` da outbox sem FK *(proposta)*: `delete` (L181-192) apaga pedidos CANCELLED, e uma FK quebraria esse delete ou apagaria a evidência da DLQ | CODIGO | src/modules/orders/order.service.ts |
| FDD-DADOS-04 | docs/FDD.md | Modelo de Dados | `webhook_deliveries`: uma linha por processamento, com resposta truncada, durationMs, errorCode e deadLetterId | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-05 | docs/FDD.md | Modelo de Dados | `webhook_dead_letter`: payload, motivo, lastErrorCode, failedAt, replayedAt e replayedById | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-06 | docs/FDD.md | Modelo de Dados | Relações inversas em `Customer` (L40-54) e `User` (L25-38); nenhuma coluna existente muda | CODIGO | prisma/schema.prisma |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Criação do evento: `publishWebhookEvent` na transação do `changeStatus`, filtro por status, um `eventId`, uma linha por endpoint e rollback se falhar | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Worker: recuperação na partida, polling de 2s dos pendentes mais antigos, lote de 10 *(proposta)*, envio sequencial com timeout e parada controlada | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Retry: esperas de 1m/5m/30m/2h/12h; o 6º envio com falha vai para a DLQ; erros determinísticos sem retry | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | DLQ numa transação, com `deadLetterId` no histórico; o replay "Recoloca na outbox como pendente", com o mesmo `eventId` | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Rotação: secret anterior válida por 24h, com duas assinaturas na janela *(proposta)* | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Desativação e remoção lógica: sem eventos novos; pendentes vão para a DLQ com `WEBHOOK_ENDPOINT_INACTIVE`, sem retry *(proposta)* | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | `POST /api/v1/webhooks` com url e lista de status; 201 com a secret gerada pela plataforma, mostrada só aqui | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-01b | docs/FDD.md | Contrato | `customerId` vem na requisição, não do JWT; usar o body, e não o path, é *(proposta)* | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | `GET /api/v1/webhooks?customerId=`: webhooks de um customer, sem a secret, com paginação *(proposta)* | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | `PATCH /api/v1/webhooks/:id`: edita url, events e active | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | `DELETE /api/v1/webhooks/:id`: 204, com remoção lógica *(proposta)* | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | `POST /api/v1/webhooks/:id/secret/rotate` (path *(proposta)*): nova secret e `previousSecretExpiresAt` em 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | `GET /api/v1/webhooks/:id/deliveries`: até 100 entregas em ordem decrescente, com payload e `deadLetterId` | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay`: só ADMIN, 202; 409 em replay repetido *(proposta)* | TRANSCRICAO | [09:18] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Saída: `POST` para a URL do cliente com o payload em snake_case (`event_id`, `event_type`, `order_id`, `from_status`, `to_status`, `total_cents`...) | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-08b | docs/FDD.md | Contrato | Headers da saída: `Content-Type`, `X-Event-Id`, `X-Timestamp` (para o cliente detectar replay) e `X-Signature` | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-08c | docs/FDD.md | Contrato | Header `X-Webhook-Id`, para o cliente com vários cadastros saber qual recebeu o envio | TRANSCRICAO | [09:44] Sofia |
| FDD-ERR-01 | docs/FDD.md | Erro | `WEBHOOK_NOT_FOUND` (404), subclasse de `AppError`, não de `NotFoundError` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-01b | docs/FDD.md | Erro | `NotFoundError` fixa o código `NOT_FOUND` (L27-31), por isso `WEBHOOK_NOT_FOUND` estende `AppError` | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERR-02 | docs/FDD.md | Erro | `WEBHOOK_INVALID_URL` (400), estende `BadRequestError`: o Zod valida o formato e o service recusa o que não é `https` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-04 | docs/FDD.md | Erro | `WEBHOOK_DEAD_LETTER_NOT_FOUND` (404, nome *(proposta)*): id da DLQ inexistente no replay | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-05 | docs/FDD.md | Erro | `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED` (409, estende `ConflictError`): linha já reprocessada, um replay por linha *(proposta)* | TRANSCRICAO | [09:36] Sofia |
| FDD-ERR-10 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_TIMEOUT`: sem resposta em 10s, com retry | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-11 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_HTTP_ERROR`: resposta fora de 2xx, com retry (só 2xx como sucesso é *(proposta)*) | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-12 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_NETWORK_ERROR`: DNS, conexão recusada ou TLS, com retry ("Se o cliente tá offline") | TRANSCRICAO | [09:14] Larissa |
| FDD-ERR-13 | docs/FDD.md | Erro | `WEBHOOK_PAYLOAD_TOO_LARGE`: corpo acima de 64KB é erro e vai direto para a DLQ, sem retry *(interpretação)* | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-14 | docs/FDD.md | Erro | `WEBHOOK_SECRET_REQUIRED`: endpoint sem secret utilizável vai direto para a DLQ; guarda defensiva *(proposta)*, inalcançável enquanto a secret for gerada no cadastro (o código foi citado só como exemplo de nome) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-15 | docs/FDD.md | Erro | `WEBHOOK_MAX_ATTEMPTS_EXCEEDED`: motivo da DLQ quando o 6º envio falha | TRANSCRICAO | [09:17] Larissa |
| FDD-ERR-16 | docs/FDD.md | Erro | `WEBHOOK_ENDPOINT_INACTIVE`: endpoint desativado ou removido; o evento vai para a DLQ sem retry e pode ser reprocessado depois de reativar o endpoint *(proposta)* | TRANSCRICAO | [09:21] Bruno |
| FDD-RES-01 | docs/FDD.md | Resiliência | Transação do `changeStatus` sem chamada de rede: só 1 leitura e 1 inserção em lote | TRANSCRICAO | [09:04] Bruno |
| FDD-RES-02 | docs/FDD.md | Resiliência | Isolamento de processo: reiniciar a API não interrompe as entregas, e vice-versa | TRANSCRICAO | [09:11] Diego |
| FDD-RES-03 | docs/FDD.md | Resiliência | Polling a cada 2s, sem sobreposição de ciclos *(proposta)* | TRANSCRICAO | [09:09] Diego |
| FDD-RES-04 | docs/FDD.md | Resiliência | Lote pequeno, 10 linhas *(proposta)*, processadas uma a uma por `createdAt` | TRANSCRICAO | [09:08] Diego |
| FDD-RES-05 | docs/FDD.md | Resiliência | Timeout de 10s por envio, com `AbortSignal.timeout` | TRANSCRICAO | [09:42] Diego |
| FDD-RES-06 | docs/FDD.md | Resiliência | Backoff 1m/5m/30m/2h/12h, até 6 envios de cada evento para cada endpoint, janela de 14h36m *(interpretação da contagem)* | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-07 | docs/FDD.md | Resiliência | DLQ em tabela separada, para esgotamento ou erro determinístico | TRANSCRICAO | [09:18] Diego |
| FDD-RES-08 | docs/FDD.md | Resiliência | Replay manual só por ADMIN, com o mesmo `eventId` e o contador de envios zerado *(proposta)* | TRANSCRICAO | [09:36] Sofia |
| FDD-RES-09 | docs/FDD.md | Resiliência | Idempotência no cliente: at-least-once, e o cliente descarta repetições pelo `X-Event-Id` | TRANSCRICAO | [09:25] Diego |
| FDD-RES-10 | docs/FDD.md | Resiliência | Na partida, linhas `PROCESSING` voltam a `PENDING` *(proposta)*: com um único worker, só sobram de execução interrompida | TRANSCRICAO | [09:12] Diego |
| FDD-RES-11 | docs/FDD.md | Resiliência | Parada controlada em `SIGINT`/`SIGTERM`, no molde do `shutdown` do servidor (L13-21) | CODIGO | src/server.ts |
| FDD-RES-11b | docs/FDD.md | Resiliência | O worker é uma entry-point nova "Tipo o que a gente já tem em src/server.ts" | TRANSCRICAO | [09:11] Larissa |
| FDD-RES-12 | docs/FDD.md | Resiliência | Entrega, estado da outbox e linha da DLQ na mesma transação *(proposta)*, no padrão do `$transaction` (L131) | CODIGO | src/modules/orders/order.service.ts |
| FDD-RES-13 | docs/FDD.md | Resiliência | Sem seguir redirect, e um 3xx conta como falha *(proposta)*, derivado da regra de só `https` | TRANSCRICAO | [09:23] Sofia |
| FDD-RES-14 | docs/FDD.md | Resiliência | Limite de 64KB medido no worker; acima, DLQ sem retry e sem afetar o status *(interpretação)* | TRANSCRICAO | [09:24] Larissa |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Métrica de latência do evento até o fim da 1ª tentativa, com meta abaixo de 10s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Métrica de backlog, os pendentes com `nextAttemptAt` vencido, com alerta *(proposta)* | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | Métrica de idade do pendente vencido mais antigo, alerta acima de 60s *(proposta)*: o evento não pode ficar pendurado | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-04 | docs/FDD.md | Observabilidade | Métrica de sucesso na 1ª tentativa por webhook *(proposta)*, a partir do sucesso ou falha de cada entrega | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-05 | docs/FDD.md | Observabilidade | Métrica de DLQ aberta por motivo, com alerta a cada entrada nova *(proposta)* | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-06 | docs/FDD.md | Observabilidade | Métrica de duração das respostas ("tempo de resposta") por webhook | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-07 | docs/FDD.md | Observabilidade | Métrica de envios por evento entregue, para acompanhar a janela de retry *(proposta)* | TRANSCRICAO | [09:17] Larissa |
| FDD-OBS-08 | docs/FDD.md | Observabilidade | Métrica de volume por endpoint, insumo para decidir o rate limiting | TRANSCRICAO | [09:39] Diego |
| FDD-OBS-09 | docs/FDD.md | Observabilidade | Heartbeat do worker em log, sem biblioteca nova *(proposta)* | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-10 | docs/FDD.md | Observabilidade | Eventos de log em Pino do enqueue ao replay; o de replay leva `userId` para auditoria (os demais são *(proposta)*) | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-11 | docs/FDD.md | Observabilidade | Secret, secret anterior e `X-Signature` fora do log, ampliando `redactPaths` (L4-11) *(proposta)* | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-12 | docs/FDD.md | Observabilidade | Tracing por IDs de correlação a partir do `requestId` (L6-7): `orderId`, `eventId`, `outboxId` e `deadLetterId` | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-01 | docs/FDD.md | Integração | `changeStatus` (L126-179) chama `publishWebhookEvent` entre L167 e L169; `TxClient` (L24) não é exportado, usar `Prisma.TransactionClient` | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-01b | docs/FDD.md | Integração | Inserir na `webhook_outbox` dentro da transação do `changeStatus`: "Se a outbox falhar de inserir, rollback" | TRANSCRICAO | [09:40] Bruno |
| FDD-INT-02 | docs/FDD.md | Integração | Status aceitos no filtro derivados de `allowedTransitions` (L16-18); `transitions` (L3-10) não é exportado | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | Enum e 4 models novos no padrão de `Order` (`@@index`, `@@map`); `Customer` e `User` só ganham relações inversas | CODIGO | prisma/schema.prisma |
| FDD-INT-04 | docs/FDD.md | Integração | Erros `WEBHOOK_*` como subclasses de `AppError`, `BadRequestError` e `ConflictError` | CODIGO | src/shared/errors/http-errors.ts |
| FDD-INT-04b | docs/FDD.md | Integração | Seguir o padrão de erros existente (`AppError`, classes específicas, códigos) com códigos `WEBHOOK_*` | TRANSCRICAO | [09:28] Bruno |
| FDD-INT-05 | docs/FDD.md | Integração | `authenticate` (L27-47) em todas as rotas; `requireRole('ADMIN')` (L49-61) no router de admin | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-05b | docs/FDD.md | Integração | No replay, "a gente reaproveita o requireRole que já existe" | TRANSCRICAO | [09:36] Larissa |
| FDD-INT-06 | docs/FDD.md | Integração | `errorMiddleware` (L14-65) e `validate` sem alteração | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-06b | docs/FDD.md | Integração | O middleware de erro centralizado "Vai pegar nossos erros sem precisar mudar nada" | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-07 | docs/FDD.md | Integração | Mesmo `logger` (L32), com child no worker; `redactPaths` ampliado *(proposta)* | CODIGO | src/shared/logger/index.ts |
| FDD-INT-07b | docs/FDD.md | Integração | Logger Pino, que "já tá no projeto inteiro", sem nada novo | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-08 | docs/FDD.md | Integração | O worker usa o `prisma` do seu processo (L10) e o mesmo `env`, que também exige `JWT_SECRET` | CODIGO | src/config/database.ts |
| FDD-INT-08b | docs/FDD.md | Integração | PrismaClient próprio no worker: "Mesmo banco, mesma DATABASE_URL, mas instância nova" | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-09 | docs/FDD.md | Integração | `src/server.ts` (bootstrap, shutdown e sinais) como molde de `src/worker.ts` (novo, a criar) | CODIGO | src/server.ts |
| FDD-INT-09b | docs/FDD.md | Integração | Entry-point nova do worker no molde de `src/server.ts` | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-10 | docs/FDD.md | Integração | `buildControllers` (L26-53) monta repository, service e controller de webhooks; rotas `/webhooks` e `/admin/webhooks` | CODIGO | src/app.ts |
| FDD-INT-11 | docs/FDD.md | Integração | Módulo de pedidos como molde (routes, schemas, controller, repository); contrato de `PATCH /orders/:id/status` inalterado | CODIGO | src/modules/orders/order.routes.ts |
| FDD-INT-11b | docs/FDD.md | Integração | "Webhook vai seguir igual": módulo em `src/modules` com controller, service, repository, routes e schemas | TRANSCRICAO | [09:27] Bruno |
| FDD-INT-12 | docs/FDD.md | Integração | `paginated()` (L22-24) na resposta da listagem de webhooks (CT-02) | CODIGO | src/shared/http/response.ts |
| FDD-INT-13 | docs/FDD.md | Integração | Scripts `worker` e `dev:worker` ao lado de `dev` e `start`; nenhuma dependência nova | CODIGO | package.json |
| FDD-INT-13b | docs/FDD.md | Integração | Script "npm run worker" para a entry-point nova | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-14 | docs/FDD.md | Integração | `beforeEach` (L8-16) apaga as 4 tabelas novas antes de `orderStatusHistory`; factories nos testes novos | CODIGO | tests/setup.ts |
| FDD-INT-15 | docs/FDD.md | Integração | Colunas camelCase e FKs (L50-69, L110-125) como referência; a migration nova é separada e aditiva | CODIGO | prisma/migrations/20260519182739_init/migration.sql |
| FDD-INT-16 | docs/FDD.md | Integração | `CustomerService.getById` (L23-27) valida o customer no cadastro | CODIGO | src/modules/customers/customer.service.ts |
| FDD-INT-17 | docs/FDD.md | Integração | Mesmo MySQL 8.0 (`image: mysql:8.0`, L3), nenhum serviço novo | CODIGO | docker-compose.yml |
| FDD-INT-17b | docs/FDD.md | Integração | "Outbox no MySQL existente resolve", sem subir infraestrutura | TRANSCRICAO | [09:07] Diego |
| FDD-INT-18 | docs/FDD.md | Integração | O worker lê a mesma `DATABASE_URL` (L5) | CODIGO | .env.example |
| FDD-CAT-01 | docs/FDD.md | Critério de Aceite | Com 2 endpoints inscritos, o `PATCH` de status gera 2 linhas `PENDING` com o mesmo `eventId` e o payload de 9 campos | TRANSCRICAO | [09:06] Diego |
| FDD-CAT-02 | docs/FDD.md | Critério de Aceite | Com `publishWebhookEvent` forçado a falhar: 500 e rollback de pedido, histórico, estoque e outbox | TRANSCRICAO | [09:40] Bruno |
| FDD-CAT-03 | docs/FDD.md | Critério de Aceite | Status não inscrito, endpoint inativo ou removido e criação de pedido não geram linha na outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-CAT-04 | docs/FDD.md | Critério de Aceite | Cadastro: URL http dá `WEBHOOK_INVALID_URL`; outros erros de schema, `VALIDATION_ERROR`; customer inexistente, 404 | TRANSCRICAO | [09:23] Sofia |
| FDD-CAT-05 | docs/FDD.md | Critério de Aceite | Secret só na resposta do `POST`; ausente no `GET`, no `PATCH` e nos logs | TRANSCRICAO | [09:31] Marcos |
| FDD-CAT-06 | docs/FDD.md | Critério de Aceite | Entrega com 2xx: `DELIVERED`, os 5 headers e `X-Signature` verificável com a secret | TRANSCRICAO | [09:22] Sofia |
| FDD-CAT-07 | docs/FDD.md | Critério de Aceite | Timeout: `WEBHOOK_DELIVERY_TIMEOUT` e próxima tentativa em 1m | TRANSCRICAO | [09:42] Diego |
| FDD-CAT-08 | docs/FDD.md | Critério de Aceite | Falhas seguidas gravam esperas de 1m, 5m, 30m, 2h e 12h | TRANSCRICAO | [09:17] Larissa |
| FDD-CAT-09 | docs/FDD.md | Critério de Aceite | 6ª falha: `FAILED`, DLQ com `WEBHOOK_MAX_ATTEMPTS_EXCEEDED` e nenhum 7º envio | TRANSCRICAO | [09:15] Diego |
| FDD-CAT-10 | docs/FDD.md | Critério de Aceite | Payload de 65.537 bytes: DLQ com `WEBHOOK_PAYLOAD_TOO_LARGE`, sem chamada HTTP *(interpretação)* | TRANSCRICAO | [09:24] Larissa |
| FDD-CAT-11 | docs/FDD.md | Critério de Aceite | Endpoint inativo ou removido, inclusive no replay: DLQ com `WEBHOOK_ENDPOINT_INACTIVE`, sem retry *(proposta)* | TRANSCRICAO | [09:21] Bruno |
| FDD-CAT-12 | docs/FDD.md | Critério de Aceite | Replay: OPERATOR recebe 403; ADMIN, 202 com o mesmo `eventId` e log com `userId`; repetido, 409; inexistente, 404 | TRANSCRICAO | [09:36] Sofia |
| FDD-CAT-13 | docs/FDD.md | Critério de Aceite | Rotação: duas assinaturas verificáveis durante 24h *(proposta)*; depois, só a nova | TRANSCRICAO | [09:21] Sofia |
| FDD-CAT-14 | docs/FDD.md | Critério de Aceite | Histórico com no máximo 100 itens, em ordem decrescente | TRANSCRICAO | [09:34] Marcos |
| FDD-CAT-15 | docs/FDD.md | Critério de Aceite | Linhas `PROCESSING` de execução interrompida voltam a `PENDING` e saem com o mesmo `eventId` *(proposta)* | TRANSCRICAO | [09:24] Diego |
| FDD-CAT-16 | docs/FDD.md | Critério de Aceite | A suíte atual passa sem alteração (inclusive o teste de `PATCH /status`, L59-87); só `tests/setup.ts` muda | CODIGO | tests/orders.test.ts |
| FDD-CAT-17 | docs/FDD.md | Critério de Aceite | `DELETE` devolve 204 e o webhook some da lista; `PATCH`, `DELETE` e rotação depois dão 404 | TRANSCRICAO | [09:33] Bruno |
| FDD-RISK-01 | docs/FDD.md | Risco | A outbox cresce sem arquivamento e a leitura fica lenta (Média/Médio) | TRANSCRICAO | [09:07] Bruno |
| FDD-RISK-02 | docs/FDD.md | Risco | Worker parado: eventos se acumulam sem entrega, mas sem perda (Média/Alto) | TRANSCRICAO | [09:06] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Endpoint lento ocupa a fila única até 10s por envio e atrasa os outros clientes (Média/Médio) | TRANSCRICAO | [09:42] Diego |
| FDD-RISK-04 | docs/FDD.md | Risco | Cliente processa o mesmo evento mais de uma vez (Média/Médio) | TRANSCRICAO | [09:25] Diego |
| FDD-RISK-05 | docs/FDD.md | Risco | Eventos do mesmo pedido fora de ordem sob retry (Baixa/Baixo) | TRANSCRICAO | [09:13] Larissa |
| FDD-RISK-06 | docs/FDD.md | Risco | Vazamento de secret (Baixa/Alto) | TRANSCRICAO | [09:22] Diego |
| FDD-RISK-07 | docs/FDD.md | Risco | Falha na inserção da outbox derruba a mudança de status, por decisão (Baixa/Alto) | TRANSCRICAO | [09:40] Bruno |
| FDD-RISK-08 | docs/FDD.md | Risco | Qualquer usuário autenticado gerencia webhooks de qualquer customer, aceito "Por enquanto sim" (Média/Médio) | TRANSCRICAO | [09:37] Sofia |
| FDD-RISK-08b | docs/FDD.md | Risco | `User` não tem vínculo com `Customer` (model `User`, L25-38) | CODIGO | prisma/schema.prisma |
| FDD-RISK-09 | docs/FDD.md | Risco | Rajada de mudanças de status sobrecarrega o cliente (Média/Médio) | TRANSCRICAO | [09:38] Diego |
| FDD-RISK-10 | docs/FDD.md | Risco | URL `https` apontando para endereço interno da rede do OMS (Baixa/Alto) *(interpretação: risco não discutido; só a regra do `https` foi decidida)* | TRANSCRICAO | [09:23] Sofia |
| FDD-RISK-11 | docs/FDD.md | Risco | Cliente fora do ar por mais de 14h36m perde a entrega automática (Baixa/Médio) | TRANSCRICAO | [09:17] Marcos |
| FDD-RISK-12 | docs/FDD.md | Risco | Duas instâncias do worker por engano quebram a ordem e podem enviar a mesma linha duas vezes (Baixa/Médio) | TRANSCRICAO | [09:12] Diego |

## Resumo de cobertura

Números gerados pelo script de validação do tracker (Python, só biblioteca padrão), que também confere cada fala na transcrição e cada caminho no repositório.

- **Linhas:** 300 (PRD 85, RFC 39, ADRs 46, FDD 130), das quais 37 são uma origem a mais de um item (sufixo de letra, como `FDD-INT-01b`).
- **`TRANSCRICAO`:** 260 linhas (86,7%), apontando para 66 falas distintas; toda `Localização` existe na transcrição como `[hh:mm] Nome`.
- **`CODIGO`:** 40 linhas, apontando para 22 arquivos distintos; todos existem no repositório e nenhum é arquivo novo da feature.
- **Cobertura dos itens rotulados:** PRD 76/76 · RFC 30/30 · ADRs 45/45 · FDD 112/112; total 263/263 (100,0%).
- **Rótulos sem linha no tracker:** 0. **Linhas sem rótulo no documento:** 0.
