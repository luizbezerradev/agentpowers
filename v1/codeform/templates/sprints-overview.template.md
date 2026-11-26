# SPRINTS OVERVIEW — {{NOME_DA_INICIATIVA}}

Este arquivo organiza a sequência oficial de sprints da iniciativa descrita em
[PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md).

As sprints foram ordenadas para permitir:
1. implementação incremental;
2. validação técnica por camadas (cada sprint é testável antes da próxima);
3. {{razão específica da iniciativa}}.

## Ordem oficial de execução
1. [SPRINT_01_{{TITULO}}.md](./SPRINT_01_{{TITULO}}.md) — {{resumo de uma linha}}
2. [SPRINT_02_{{TITULO}}.md](./SPRINT_02_{{TITULO}}.md) — {{resumo}}
<!-- ... todas as sprints, na ordem lógica de implementação -->

## Blocos de execução
<!-- Reflete o PROJECT_CONTEXT §11.1. Sprints do mesmo bloco são paralelas (/codeform:orchestrate);
     blocos rodam em sequência. Plano `serial`: um bloco por sprint. -->
| Bloco | Sprints | Paralelas? | Ordem de merge |
|---|---|---|---|
| A | SPRINT_01 | não | — |
| B | SPRINT_02 ∥ SPRINT_03 | sim | SPRINT_02 → SPRINT_03 |

## Estado do plano
- **Dry-run:** {{⚠️ pendente | ✅ aprovado em AAAA-MM-DD}} <!-- o /codeform:orchestrate exige ✅ -->

## Regra de execução
- Nenhuma sprint começa antes de a anterior ter os critérios de aceite **validados** e marcados `[x]`.
- Cada sprint roda via `/codeform:execute <sprint>` (ou, em bloco paralelo, via `/codeform:orchestrate`) seguindo as regras MANDATORY; review e security acontecem dentro do execute.
- {{regra adicional específica da iniciativa, se houver}}.

## Resultado esperado ao final da sequência
Ao final da Sprint {{N}}, o projeto deverá ter:
- {{entrega 1}};
- {{entrega 2}};
- {{entrega final que materializa o objetivo do PROJECT_CONTEXT}}.
