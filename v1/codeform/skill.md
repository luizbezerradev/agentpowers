---
name: codeform
description: Metodologia pessoal de specs em 5 comandos (brainstorm → plan → execute | orchestrate → release). Use quando o usuário quer estudar uma feature/iniciativa e desenhar o wireframe (brainstorm), escrever PROJECT_CONTEXT + sprints e validá-los num dry-run independente (plan), implementar uma sprint com review e auditoria de segurança por subagents (execute), paralelizar sprints independentes em raias (orchestrate), ou validar a branch e abrir o PR (release). Roteia para /codeform:brainstorm, /codeform:plan, /codeform:execute, /codeform:orchestrate e /codeform:release e aplica as regras MANDATORY (sem tipagem fraca, lint na fonte, sem legado, Object Calisthenics, YAGNI/KISS/DRY, SDD/TDD, comentário só por exceção justificada). /codeform:issue é independente: só cria issues.
---

# Personal Specs — Metodologia de Planejamento e Execução via Sprints (fluxo v2)

Esta skill faz o Claude Code **saber sozinho** qual comando do fluxo usar quando o usuário fala em estudar uma
feature, montar o `PROJECT_CONTEXT.md`, escrever e validar sprints, implementá-las, paralelizá-las ou abrir o PR.
A fonte de verdade detalhada está em `skills/codeform/methodology.md`; as instruções dos subagents estão
em `skills/codeform/playbooks/`.

A unidade de trabalho é a **sprint** (histórias + critérios de aceite + task breakdown + Definition of Done),
no formato de `templates/sprint.template.md`.

## O ciclo → qual comando usar

| O usuário quer… | Use | Dentro dele (gates humanos entre as seções) |
|---|---|---|
| Entender a codebase/issue e, se há tela, desenhar o wireframe | **`/codeform:brainstorm <objetivo \| issue>`** | study → **aprova** → cria a branch → wireframe (`fork`, só se toca frontend) → **aprova** |
| Escrever `PROJECT_CONTEXT` + sprints + overview e validá-los | **`/codeform:plan`** | write (com plano de paralelização, flags e tier) → **aprova** → dry-run (subagent limpo) → **resolve gaps** |
| Implementar UMA sprint (serial, ou como raia com `--lane`) | **`/codeform:execute <sprint>`** | implementa → spawna reviewer (e security, pela flag) → corrige o simples, escala o sensível |
| Paralelizar sprints independentes | **`/codeform:orchestrate`** (`--status`, `--merge`) | gate do plano → N sessões `execute --lane` → fila de decisões → merge |
| Validar a branch e abrir o PR | **`/codeform:release [--base <branch>]`** | convenções do projeto + issues sincronizadas (pai) → build/lint/test por unidade (subagents) → PR se tudo ✅ |
| Criar uma issue a partir de um problema | **`/codeform:issue`** | independente do ciclo |

**Rota depois do `plan`:** o veredito do plano de paralelização (PROJECT_CONTEXT §11.1) decide. `serial` →
`/codeform:execute` sprint a sprint. `parcial`/`total` → `/codeform:orchestrate` para os blocos paralelos. Sem duas
ou mais sprints independentes num bloco, o orchestrator **não orquestra**: manda rodar o `execute`.

**Respeite os gates humanos:** cada seção termina com revisão do usuário; não pule para a próxima sem
aprovação explícita. Se o usuário descrever a tarefa sem citar o comando, identifique em qual ponto do ciclo
ele está e ofereça/rode o comando correspondente.

## Como explicar ao usuário (§0.8–0.9)

**Sempre em duas camadas:** primeiro a versão **simples** (didática, humana, objetiva, literal — o usuário é
AUTISTA), depois a versão **técnica**. Vale para toda explicação em qualquer command; detalhe na methodology §0.9.

## Quem faz o quê (resumo do §0.7)

O agente do comando (o **pai**) conversa com o usuário e responde pelo resultado. Subagents (os **filhos**)
recebem um brief fechado, **não falam com o usuário** — onde precisariam decidir, devolvem `PARADA` — e o pai
**verifica** o que eles trazem antes de apresentar. A implementação e a escrita das sprints são sempre do pai;
o dry-run, o review e o security são de subagents **de contexto limpo**; o wireframe é de um `fork`; o
build/lint/test do release é de subagents mecânicos. **Raias do orchestrator são sessões, não subagents.**

## Regras MANDATORY (resumo — detalhe na methodology §3)

1. Proibido tipagem fraca (`any` etc.). 2. Rodar lint/typecheck ao final e corrigir **na fonte, sem
workarounds**. 3. Caminho correto do repositório ao editar. 4. Sem legado no **código** — **substituir**, nunca
acumular; **dados já em produção são preservados** — migrar, nunca descartar. 5. Sempre na fonte, sem
complexidade desnecessária. 6. Marcar `[x]` nas tasks/critérios do `SPRINT_NN_*.md` ao concluir. 7. Antes de
executar: ler `PROJECT_CONTEXT` + a sprint alvo, ver o existente e buscar docs via **context7**. 8. Aplicar
**Object Calisthenics** (§4.1) + **YAGNI/KISS/DRY/Less Code** (§4.2) em todo código novo. 9. Seguir **SDD** e
**TDD** quando a lógica for isolável (§5). 10. **Comentário é exceção, não padrão** (§4.3): só as seis
categorias da lista fechada; **proibido comentário-changelog**. Arquivo novo nasce no padrão; arquivo que a
task edita tem os comentários do **arquivo inteiro** normalizados; arquivo não tocado é só **reportado**.

## Caminhos canônicos

- Iniciativas em **`docs/Sprints/<Iniciativa>/`** na raiz do projeto atual (`<Iniciativa>` = `<n>-<slug>`, com `<n>` = número da issue, salvo padrão de nome que o projeto declare); ao fim do
  `plan`, cada História vira sub-issue da issue pai (só acompanhamento; os arquivos seguem como fonte de verdade). Ao criar a pasta, o caminho relativo é
  comentado na issue de origem (se houver).
- Artefatos: `WIREFRAME_<TEMA>.html` (se há frontend),
  `PROJECT_CONTEXT.md`, `SPRINTS_OVERVIEW.md`, `SPRINT_NN_TITULO.md` (um por sprint).
- Estado das raias: `$(git rev-parse --git-common-dir)/sprint-lanes/<Iniciativa>/SPRINT_NN.md` (não versionado).

## Formato dos artefatos

`templates/sprint.template.md` (agora com "Flags de risco e paralelização" e "Registro dos gates"),
`templates/project-context.template.md` (agora com a §11.1, plano de paralelização) e
`templates/sprints-overview.template.md` (agora com "Blocos de execução" e "Estado do plano"). Esta skill cobre
o **ciclo**, não redefine esses formatos.

Detalhes completos do ciclo, regras MANDATORY, tiers de model/effort e princípios de engenharia: ver
`methodology.md`.
