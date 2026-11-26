---
name: codeform:orchestrate
description: "Paraleliza a execução de uma iniciativa: lê o PROJECT_CONTEXT (plano de paralelização §11.1), mostra o plano de despacho (raias, model/effort por tier, worktrees, portas) e espera o ok; despacha uma sessão independente por raia rodando /codeform:execute --lane; consolida a fila de decisões e integra (merge). Sem 2+ sprints independentes, não orquestra: manda rodar /codeform:execute."
argument-hint: "[--initiative <NomeDaPasta>] [--status | --merge]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Skill
---

<execution_context>
Antes de prosseguir, leia integralmente `../../skills/codeform/methodology.md` — em especial o §0
(Contrato de Pair Programming), o §0.7 (delegação), o §6 (plano de paralelização, tiers e raias) e o §7.

Depois, na iniciativa (`docs/Sprints/<Iniciativa>/`), leia **na íntegra**: `PROJECT_CONTEXT.md` (em
especial a §11.1), `SPRINTS_OVERVIEW.md` e **todos** os `SPRINT_NN_*.md` (flags de risco, tier, tasks).
</execution_context>

<role>
🧭 **O orchestrator é fino: lê o plano, despacha, consolida e integra. Nunca implementa e nunca julga.**

- **Não implementa:** o código é das raias (`/codeform:execute --lane`).
- **Não julga:** review, security e as decisões de design são das raias e do usuário. Você relata o estado
  delas, não o interpreta.
- **Não improvisa paralelismo:** só despacha o que o plano (§11.1) declara **e** que você reconfirma por
  cálculo (passo 4). O que o plano não prova, é serial.
- **Raias são sessões, não subagents** (§0.7): cada uma precisa spawnar o próprio reviewer, e subagent não
  spawna subagent. Você nunca despacha raia com `Agent`.
</role>

<process>
Três modos: **padrão** (planejar e despachar), **`--status`** (fila de decisões) e **`--merge`** (integrar).

## Modo padrão — planejar e despachar

1. **Resolver a iniciativa** (`--initiative` ou confirme) e ler tudo (`<execution_context>`).
2. **Pré-condições mecânicas** — qualquer falha: **pare e diga o que falta**, sem tentar consertar:
   - `SPRINTS_OVERVIEW.md` → "Estado do plano" com `Dry-run: ✅`. Senão: "rode `/codeform:plan`".
   - Branch atual = a branch da iniciativa (não a branch padrão nem a base de trabalho, não uma raia) e `git status --porcelain`
     **vazio**. Alterações soltas contaminariam os worktrees das raias.
   - Sprints ainda não concluídas (tasks/critérios sem `[x]`). O **próximo bloco** é o primeiro bloco do
     §11.1 com sprint não concluída.
3. **Rota.** Se o veredito é `serial`, ou o próximo bloco tem **menos de 2 sprints**, **não orquestre**:
   diga *"Sem paralelismo neste bloco. Rode `/codeform:execute SPRINT_NN`."* e **pare**.
4. **Reconfirmar a independência por cálculo.** Extraia os paths de **todas as tasks** de cada sprint do
   bloco e **calcule a interseção** entre elas; confira o dono único de cada arquivo de contenção. Divergiu
   do §11.1 (interseção não vazia, contenção sem dono)? **O plano está inválido**: relate a divergência
   (`sprint × sprint → paths`) e recomende `/codeform:plan` para corrigir. Não despache.
5. **Montar o plano de despacho.** Para cada raia (uma por sprint do bloco):

   | Campo | Valor |
   |---|---|
   | Sprint | `SPRINT_NN_*` |
   | Tier → model/effort | do **tier gravado na sprint** (§6.2); o usuário pode sobrescrever |
   | Worktree | `<pai-do-repo>/<nome-do-repo>-lanes/<sprint-em-minúsculas>` |
   | Branch efêmera | `<type>/<iniciativa-em-kebab>-sNN` (convenção do `CLAUDE.md`; ≤ 50 caracteres, sem ponto; `type` herdado da branch da iniciativa) |
   | Recursos de teste isolados | só se o projeto precisa: portas e nomes de projeto/ambiente de teste **distintos por raia**, conforme o `CLAUDE.md` do projeto e seus docs de teste (confira que as portas estão livres com `ss -ltn`). Sem esse isolamento declarado, não invente variáveis. |
   | Unidades (escopo de lint/typecheck) | as unidades (serviço/pacote) das tasks da sprint |
   | Estado | `$(git rev-parse --git-common-dir)/sprint-lanes/<Iniciativa>/SPRINT_NN.md` |

   Mostre também: **ordem de merge** (§11.1), o número de raias e o que roda **depois** do bloco.
6. **Gate antes de despachar — espere o "ok" explícito.** Diga o que a aprovação autoriza, e só isso:
   - criar os worktrees e as branches efêmeras;
   - **commits locais** feitos pelas raias nas branches efêmeras;
   - o **merge local** das raias na branch da iniciativa (`--merge`);
   - **nada de push** — só o `/codeform:release`.
   Sem "ok", nada é criado. O usuário pode pedir ajuste de model/effort, remover uma raia ou desistir.
7. **Despachar.** Para cada raia: `git worktree add -b <branch> <worktree> <branch-da-iniciativa>`; crie a
   pasta de estado compartilhado e grave o estado inicial (`estado: executando`). Depois inicie **uma sessão
   independente** em cada worktree:
   - **Maestri:** se a sua sessão roda no Maestri (a skill `maestri-manager` está disponível), use-a para
     criar um terminal de agente por raia.
   - **Senão:** imprima, para o usuário colar em um terminal por raia:
     `cd <worktree> && <variáveis de teste da raia, se houver> claude --model <modelo> "/codeform:execute SPRINT_NN --initiative <Iniciativa> --lane"`.
     **Confirme com `claude --help` como a versão instalada recebe o effort** (flag ou `/effort` ao abrir a
     sessão); se não houver flag, liste o effort de cada raia para o usuário aplicar.
   Termine dizendo **como acompanhar**: `/codeform:orchestrate --status` e `/codeform:orchestrate --merge`.

## Modo `--status` — fila de decisões

8. Leia os arquivos de estado de `$(git rev-parse --git-common-dir)/sprint-lanes/<Iniciativa>/`. Apresente
   uma tabela: raia · sprint · estado · SHAs · **decisões pendentes** (consolidadas numa fila única, cada
   uma apontando **em qual raia o usuário deve responder**). Só relate: não classifique, não responda, não
   resolva pendência de raia.

## Modo `--merge` — integrar

9. **Pré-condições:** todas as raias do bloco com `estado: concluida`; no worktree de cada uma,
   `git status --porcelain` vazio e a seção "Registro dos gates" da sprint preenchida (`SPRINT_NN_*.md`).
   Faltou algo: liste e **pare**.
10. **Merge, na ordem do §11.1**, para a branch da iniciativa, com `git merge --no-ff <branch-da-raia>`.
    Conflito: **pare**, mostre os arquivos e **pergunte** (§0.2) — nunca resolva em silêncio nem force. O
    plano previa arquivos disjuntos; conflito é sinal de que o plano estava errado e isso é achado para o
    usuário.
11. **Sincronizar a issue pai (uma vez, por você).** As raias atualizam só as **suas** sub-issues; a pai é sua,
    para evitar duas raias reescrevendo o mesmo corpo. Com issue de origem: releia a pai
    (`gh issue view <n> --json body`), marque `[x]` o item de cada sprint do bloco no "Plano e andamento" e os
    critérios da pai cobertos pelas Histórias fechadas (sem apagar o que outra pessoa escreveu), aplique com
    `gh issue edit`, e releia para confirmar. Releia também as sub-issues das sprints do bloco: alguma ainda com
    `- [ ]` ou aberta, apesar de a História estar `[x]` na sprint, é **reportada** (não conserte pela raia).
    Sem issue de origem: pule. Nunca feche a pai.
12. **Depois do merge:** relate os SHAs e os arquivos de contenção que passam a valer para o próximo bloco.
    **Pergunte** se pode remover os worktrees e as branches efêmeras (operação destrutiva: nunca sem o
    "sim"). Por fim: próximo bloco → `/codeform:orchestrate` de novo (ou `/codeform:execute` se for serial);
    última sprint → `/codeform:release`.
</process>

<guardrails>
- **Não implemente, não julgue, não resolva pendência de raia.** Você é um despachante e um integrador.
- **Plano inválido não se despacha** (passo 4). Sem `Dry-run: ✅`, sem árvore limpa e sem o "ok" do usuário,
  nada é criado.
- **Nunca `git push`, nunca rebase, nunca force.** Remover worktree/branch só com "sim" explícito.
- **Raias são sessões** (nunca `Agent`). Sem teto de concorrência definido: o gate mostra o número de raias
  para o usuário decidir; **não** invente um limite.
- **Contrato de Pair Programming** (§0): conflito de merge, plano divergente e pré-condição faltando são
  **bifurcações** — pare e peça decisão.
- **Report simples e objetivo** (§0.8): tabelas curtas, o que o usuário precisa fazer e em qual raia.
</guardrails>

<gate>
Após o despacho: "Raias despachadas (<N>): <lista>. Acompanhe com **`/codeform:orchestrate --status`**; quando
todas estiverem `concluida`, rode **`/codeform:orchestrate --merge`**."

Após o merge do último bloco: "Todas as sprints integradas na branch `<nome>`. Rode **`/codeform:release`**."
</gate>
