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

**Andamento**

| Fase | Status |
| --- | --- |
| 0. Setup (branch `docs/design-docs-webhooks`, ambiente, registro do processo) | ✅ |
| 1. Extração e catálogo da transcrição | ⏳ |
| 2. ADRs | ⏳ |
| 3. RFC | ⏳ |
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

<!-- Próximos: prompts dirigidos de extração da transcrição (fase 1) e de geração dos documentos (fases 2-6). -->

## Iterações e ajustes

_Em andamento: cada fase acrescenta aqui os momentos em que a IA gerou algo errado ou superficial e o que foi corrigido._

| # | Fase / documento | O que a IA gerou | Problema | Ajuste |
| --- | --- | --- | --- | --- |

## Como navegar a entrega

| Ordem | Arquivo | Pergunta que responde | Status |
| --- | --- | --- | --- |
| 1 | [`docs/PRD.md`](./docs/PRD.md) | Por que e o quê? | ⏳ |
| 2 | [`docs/RFC.md`](./docs/RFC.md) | Como pretendemos resolver e o que ainda está em aberto? | ⏳ |
| 3 | [`docs/adrs/`](./docs/adrs/README.md) | Por que decidimos exatamente assim? | ⏳ |
| 4 | [`docs/FDD.md`](./docs/FDD.md) | Como construir, em detalhe? | ⏳ |
| 5 | [`docs/TRACKER.md`](./docs/TRACKER.md) | De onde veio cada coisa? | ⏳ |
