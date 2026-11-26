# Fluxo de trabalho

## Como trabalhar
- Responda em Português Brasil. Identificadores de código ficam no original.
- Entenda antes de mudar: leia o código e rastreie o fluxo real. Reuse o que já existe antes de escrever.
- TDD sempre: teste falhando (rode e veja falhar) → mínimo para passar → refatorar. Skill `tdd`.
- Nunca diga "pronto" sem a saída de um comando rodado depois da última edição. Skill `verify`.
- Código: KISS, YAGNI, DRY, menos código é melhor. Skill `principles`. Apagar vale mais que adicionar.
- Comandos de build/teste/lint do projeto ficam no CLAUDE.md do projeto e em `.claude/test-cmd`.

- Explique SEMPRE em duas camadas, nesta ordem: (1) versão simples — direta, literal, humana, didática e objetiva, sem metáfora ambígua nem jargão sem explicação (o usuário é AUTISTA); (2) versão técnica, rotulada. As duas não podem se contradizer.

## Pista leve × fluxo completo
- Mudança pequena (≤ ~3 arquivos, critério de aceite em uma frase, sem risco): faça direto, com `tdd` + `verify`.
- Maior, ambígua ou arriscada: seguimos utilizando `/sprint:brainstorm` -> `/sprint:plan` -> `/sprint:orchestrate` ou `/sprint:execute` -> `/sprint:release` (entender → TDD com princípios → reviewer isolado).

## SEMPRE pare e pergunte antes de
- Migration, mudança de schema ou de dados existentes
- Auth, permissões, segredos, pagamento (skill `security`)
- Dependência nova
- Mudança de contrato de API pública
- Operação destrutiva, deploy, infra, push em main
- Requisito ambíguo com impacto real
- Alterar teste que já existia (nunca altere teste para fazê-lo passar)
- Escopo crescendo além do que foi pedido

## Subagents
- Paralelize só trabalho independente (áreas distintas, arquivos disjuntos). Na dúvida, sequencial.
- Passe a cada subagent: objetivo, fronteiras e formato de saída.
