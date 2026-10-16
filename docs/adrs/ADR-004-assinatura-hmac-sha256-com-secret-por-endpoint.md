# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint

- **Data:** 2026-10-15
- **Decisores:** Sofia (Engenheira de Segurança), Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma), Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager, só na geração da secret pela plataforma, [09:31] Marcos)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (contrato de saída, rotação da secret, modelo de dados) · [PRD](../PRD.md) · [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md) · [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md)

## Status

Aceito. Fechado em [09:22] Sofia ("Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h") e confirmado no resumo de [09:48] Larissa. A forma de manter a secret antiga válida durante as 24h (duas assinaturas) é **proposta a validar na revisão de segurança** ([09:46] Sofia).

## Contexto

- "a gente tá expondo eventos com dados de pedidos pra um endpoint fora da nossa infra. O cliente tem que conseguir validar que a requisição veio realmente da gente, e que ninguém adulterou o payload no meio" ([09:19] Sofia).
- Vazamento de secret já aconteceu: "A gente já teve cliente que vazou secret em log de aplicação dele uma vez" ([09:22] Diego).
- A configuração de cada webhook guarda url, secret, customer e estado ativo ([09:21] Bruno, confirmado com "Sim" em [09:21] Sofia).
- O cliente cadastra o webhook pela API, e a "secret é gerada pela gente e devolvida na criação" ([09:31] Marcos).
- No código, o logger Pino já mascara dados sensíveis (`src/shared/logger/index.ts:4-11`: `authorization`, `cookie`, `password`, `passwordHash`, `token`, `accessToken`), mas nenhum campo de secret. A senha de usuário é guardada só como hash (`passwordHash`, `prisma/schema.prisma:28`).
- A Sofia quer revisar "HMAC e geração de secret" com calma, com pelo menos dois dias úteis antes do deploy ([09:46] Sofia).

## Decisão

**Assinar cada entrega com HMAC-SHA256 calculado sobre o corpo do request e enviado no header `X-Signature`, usando uma secret única por endpoint, gerada pela plataforma e rotacionável via API, com a secret antiga válida por mais 24h em paralelo.**

- Algoritmo HMAC-SHA256: "o padrão de mercado, todo cliente sério tem biblioteca pra isso" ([09:20] Sofia).
- A assinatura vai no header `X-Signature` e o cliente a verifica do lado dele ([09:20] Sofia). A base do cálculo é o corpo do request ([09:22] Sofia).
- O `X-Timestamp` (momento do envio) vai no request "pra cliente conseguir detectar replay attack se quiser" ([09:44] Diego), mas não entra no cálculo, porque a decisão fala só do corpo ([09:22] Sofia).
- Secret única por endpoint: "Não é uma secret global da nossa plataforma" ([09:21] Sofia).
- A secret é gerada pela plataforma e devolvida na criação ([09:31] Marcos) e, na rotação, a nova é devolvida ao cliente que a pediu pela API ([09:21] Sofia). Não mostrá-la em nenhuma outra resposta (listagem, consulta) é interpretação adotada depois da reunião, a partir de "devolvida na criação"; o contrato fica no [FDD](../FDD.md).
- Rotação via API: "Quando ele rotaciona, a antiga fica válida por 24 horas em paralelo, pra ele ter tempo de migrar os sistemas dele. Depois disso, a antiga morre" ([09:21] Sofia).
- Durante as 24h, o worker envia **duas assinaturas** no `X-Signature`, uma com a secret nova e outra com a antiga, e o cliente aceita qualquer uma. Nenhuma fala diz como a secret antiga "fica válida" num envio de saída. Esta é a forma escolhida depois da reunião para que o cliente possa migrar em qualquer momento da janela, e está marcada como proposta a validar na revisão de segurança ([09:46] Sofia). Codificação e formato do header ficam no [FDD](../FDD.md).
- Relacionados, mas fora deste ADR: a URL tem de ser `https` ("nem é decisão arquitetural, é só uma validação no schema Zod", [09:23] Sofia) e o payload tem limite de 64KB ("não vejo como decisão arquitetural separada, é só requisito não funcional", [09:24] Larissa). Os dois entram como requisitos não funcionais no [PRD](../PRD.md) e como regras no [FDD](../FDD.md).

## Alternativas Consideradas

### Alternativa 1: secret global da plataforma (ALT-08, discutida)

- **Descrição:** uma única secret, a mesma para todos os endpoints de todos os clientes.
- **Prós:** uma só secret para gerar, guardar e distribuir.
- **Contras:** um vazamento num cliente compromete as entregas de todos.
- **Motivo do descarte:** "Senão se vaza uma, vaza tudo" ([09:21] Sofia).

### Alternativa 2: confiar só no TLS, sem assinatura (plausível, não discutida)

- **Descrição:** como o `https` já é obrigatório ([09:23] Sofia), dispensar a assinatura.
- **Prós:** nada a gerar, guardar ou rotacionar.
- **Contras:** no envio, o TLS autentica o endpoint do cliente para a plataforma, e não a plataforma para o cliente. Qualquer um que descubra a URL consegue enviar um POST válido.
- **Motivo do descarte:** não atende o requisito de o cliente validar que a requisição "veio realmente da gente" e não foi adulterada ([09:19] Sofia).

### Alternativa 3: uma única assinatura durante a janela de rotação (plausível, não discutida)

- **Descrição:** continuar assinando só com a secret antiga até o fim das 24h, ou passar direto para a nova no momento da rotação.
- **Prós:** um único valor no header, sem lógica de janela no worker.
- **Contras:** só com a antiga, o cliente é obrigado a trocar exatamente no corte; só com a nova, a antiga perde o efeito na hora e quem ainda não migrou passa a rejeitar as entregas.
- **Motivo do descarte:** contraria o objetivo de dar tempo ao cliente "pra ele ter tempo de migrar os sistemas dele" ([09:21] Sofia).

### Alternativa 4: assinar o timestamp junto com o corpo (plausível, não discutida)

- **Descrição:** incluir o `X-Timestamp` na base do HMAC, para que a assinatura também proteja contra reenvio.
- **Prós:** o cliente poderia rejeitar requisições antigas com garantia criptográfica.
- **Contras:** muda a base do cálculo que foi decidida e que o cliente vai implementar.
- **Motivo do descarte:** a decisão é "HMAC-SHA256 sobre o corpo do request" ([09:22] Sofia), e o `X-Timestamp` foi definido como apoio opcional do cliente ("se quiser", [09:44] Diego).

## Consequências

### Positivas

- Um vazamento fica isolado num endpoint ([09:21] Sofia), e o risco é real ([09:22] Diego).
- Padrão de mercado: "todo cliente sério tem biblioteca pra isso" ([09:20] Sofia).
- O cliente consegue validar a origem e a integridade de cada entrega ([09:19] Sofia).
- A rotação permite reagir a um vazamento sem interromper as entregas: o cliente migra dentro das 24h ([09:21] Sofia).

### Negativas

- O HMAC exige a secret em claro na hora de assinar. Ao contrário da senha de usuário, guardada só como hash (`prisma/schema.prisma:28`), a secret precisa ser armazenada de forma recuperável. Como guardá-la é item da revisão de segurança ([09:46] Sofia).
- Durante as 24h de rotação, o worker calcula duas assinaturas e o cliente precisa aceitar qualquer uma delas. A forma ainda depende da revisão de segurança.
- A secret só é mostrada na criação e na rotação. Se o cliente a perder, o único caminho é rotacionar.
- A lista de redact do logger (`src/shared/logger/index.ts:4-11`) não cobre secret hoje. Incluí-la é proposta de implementação a validar na revisão de segurança, não decisão da reunião.
- A assinatura não impede o reenvio de uma requisição capturada: o `X-Timestamp` ajuda o cliente a detectar replay "se quiser" ([09:44] Diego), mas não é assinado.
- A revisão de segurança de pelo menos dois dias úteis antes do deploy entra no prazo da entrega ([09:46] Sofia, [09:47] Larissa).

### Trade-off explícito

> **CONS-1:** Aceitamos guardar uma secret recuperável por endpoint, assinar duas vezes durante as 24h de rotação e passar por revisão de segurança antes do deploy, em troca de vazamentos isolados por endpoint e de uma verificação de origem e integridade que o cliente faz com biblioteca padrão.

## Referências

- Transcrição: [09:19] Sofia, [09:20] Sofia, [09:21] Sofia, [09:21] Bruno, [09:22] Diego, [09:22] Sofia, [09:23] Sofia, [09:24] Larissa, [09:31] Marcos, [09:44] Diego, [09:46] Sofia, [09:47] Larissa, [09:48] Larissa.
- Código: `src/shared/logger/index.ts` (L4-11, `redactPaths`), `prisma/schema.prisma` (L28, `passwordHash`).
