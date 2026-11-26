---
name: codeform:plan
description: "Planejar — write (PROJECT_CONTEXT + sprints + overview, com plano de paralelização, flags de risco e tier) → [aprova] → dry-run em subagent limpo → [resolve gaps] → plano aprovado. Substitui /sprint:write + /sprint:dry-run."
argument-hint: "[--initiative <NomeDaPasta>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - Agent
  - SendMessage
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---

<execution_context>
Antes de prosseguir, leia integralmente:
- `../../skills/codeform/methodology.md` — em especial o §0 (Contrato de Pair Programming), o
  §0.7 (delegação), o §2 (Caminhos canônicos) e o §6 (plano de paralelização e tiers).
- Os três templates que definem o formato exato de cada artefato:
  - `../../skills/codeform/templates/project-context.template.md`
  - `../../skills/codeform/templates/sprints-overview.template.md`
  - `../../skills/codeform/templates/sprint.template.md`
- O estudo aprovado: a conversa do `/codeform:brainstorm` desta sessão. Sem estudo
  aprovado na conversa, **PARE** e mande rodar `/codeform:brainstorm`.
- O wireframe aprovado (`WIREFRAME_*.html` na pasta da iniciativa e a **URL do artifact**, informada na
  conversa do brainstorm). Leia o HTML na íntegra. Se a iniciativa toca frontend e não há wireframe aprovado, **PARE** e
  mande rodar `/codeform:brainstorm`.
</execution_context>

<source_of_truth>
⚠️ **LEIA ANTES DE TUDO — este bloco se sobrepõe a qualquer outra instrução deste arquivo.**

Não é porque está no `CLAUDE.md`, nem porque saiu do estudo do `/codeform:brainstorm`, nem sequer porque já está
implementado no código, que aquilo está correto e deve ser tratado como **fonte da verdade**. O que é
correto é o que é **conceitual e arquiteturalmente correto** — e é isso que deve ser levado em
consideração ao escrever o `PROJECT_CONTEXT.md` e as sprints.

Não fique tentando modelar a nova implementação no que já existe só porque já existe e já está lá,
como se tudo tivesse que se modificar para funcionar do jeito que está. O que se quer é o **inverso**:
se ao materializar o estudo em artefatos você identificar que o que existe está conceitual ou
arquiteturalmente incorreto, ou é um workaround, e o estudo ou o `CLAUDE.md` seguem esse erro, então
**temos que debater e corrigir tudo** — para que a nova implementação fique correta e, se necessário,
corrigindo também o que já existe, tendo em mente que utilizamos **Clean Architecture e SOLID**.

Sendo assim, em toda decisão registrada em qualquer artefato:

> Qual o caminho mais enterprise e conceitualmente e arquiteturalmente correto, que resolve o problema
> **na fonte**, sem workarounds, sem seguirmos com dívida técnica e que é **prova de futuro**?

Isso vale **independente do esforço necessário para implementação**, desde que atenda a estes
requisitos. Ao apontar um desvio, **seja didático**: explique qual formato você recomenda e por quê.
Lembre-se: o estudo e a sprint são um guia, **não uma verdade absoluta**.

**Não escreva o artefato com o desenho errado — pare e traga para debate.**
</source_of_truth>

<delegation>
⚠️ **O write é do pai; o dry-run é de um subagent novo** (§0.7 da methodology).

O dry-run roda num subagent **de contexto limpo**, seguindo o playbook
`~/.claude/skills/codeform/playbooks/dry-run.md`. Não use `fork`: ele herdaria o viés de quem
escreveu as sprints, e o dry-run existe justamente para discordar. O subagent é **read-only** e não fala
com o usuário: devolve o relatório, e **você** o verifica, o apresenta, colhe as decisões e aplica as
atualizações. Se ele devolver `PARADA`, leve ao usuário e retome o mesmo subagent via `SendMessage`.
</delegation>

<objective>
Materializar o estudo em artefatos prontos para execução — **`PROJECT_CONTEXT.md`** +
**`SPRINTS_OVERVIEW.md`** + um **`SPRINT_NN_TITULO.md`** por sprint — **e validá-los** com um dry-run
independente antes de qualquer linha de código. Além do que o antigo `/sprint:write` entregava, este comando
grava o **plano de paralelização** (PROJECT_CONTEXT §11.1), as **flags de risco** e o **tier** de cada
sprint, que o `/codeform:orchestrate` e o `/codeform:execute` consomem.

Estes artefatos são o contrato que o `/codeform:execute` vai seguir **ao pé da letra**. Por isso
**nenhuma dúvida pode virar gap na sprint escrita**: se durante a escrita surgir qualquer ambiguidade,
informação faltante ou decisão com mais de um caminho válido, **PARE imediatamente, debata com o usuário
e só continue depois de resolvido** — nunca preencha uma seção com suposição, placeholder disfarçado ou
"decisão a definir depois".
</objective>

<guiding_question>
Toda decisão registrada em `PROJECT_CONTEXT.md` ou numa sprint — princípios de engenharia (§3),
árvore de arquivos, task breakdown — deve passar pelo crivo do `<source_of_truth>` antes de ser
escrita no papel.

Estamos em **dev, mas já existe produção**: a regra de sem legado/compatibilidade vale para o
**código** — nada aqui deve preservar implementação antiga ou compatibilidade "para não quebrar
nada", o que a iniciativa substitui deve estar mapeado para remoção, não para convivência. Isso
**não** vale para os **dados**: o que já está em produção deve ser **preservado** — toda sprint que
mudar schema, formato ou local de um dado existente precisa mapear a **migração** desses dados, nunca
sua perda ou descarte.

Aplicado a esta fase: se, ao materializar o estudo em artefatos, você perceber que a análise anterior
levou a um desenho conceitualmente errado, **não escreva esse desenho só porque foi o que saiu do
estudo do `/codeform:brainstorm`** — pare, explique didaticamente por que está errado, qual caminho você recomenda e
por quê, e **traga para debate** antes de comprometer isso em `PROJECT_CONTEXT.md` ou numa sprint.
</guiding_question>

<process>
## Parte 1 — Write (pai)

### Fase A — PROJECT_CONTEXT.md

1. **Localizar/criar `docs/Sprints/`** na raiz do projeto atual (§2 da methodology). O nome da pasta da
   iniciativa segue o padrão do §2 da methodology (`<número da issue>-<slug>`, salvo padrão que o projeto declare); se a pasta já existe (criada no brainstorm), use-a. Se houver dúvida
   sobre qual é a raiz do projeto (ex.: monorepo com múltiplos `package.json`), **PARE e confirme com
   o usuário** em vez de assumir uma pasta.
   **Issue (pasta nova).** Se a sessão partiu de uma issue e a pasta da iniciativa **acabou de ser criada** (não existia), comente nela o caminho relativo: `gh issue comment <n> --body "Pasta da iniciativa: `docs/Sprints/<Iniciativa>/`"`. Pasta já existente = já comentado, não repita. Sem issue, pule.
2. **Criar `docs/Sprints/<Iniciativa>/PROJECT_CONTEXT.md`** seguindo o template importado, seção a
   seção, a partir do estudo aprovado no `/codeform:brainstorm` (conversa). **Nenhuma seção marcada `(obrigatória)` no template
   pode ficar vazia ou com placeholder** — isso inclui, entre outras, o objetivo final (§2), os
   domínios e sprints candidatas (§6), o fluxo final (§8), a árvore de arquivos impactados (§9), as
   fases de implementação (§11) e a síntese final de "onde queremos chegar / como chegaremos lá" (§14).
   Se o estudo da fase anterior não tiver informação suficiente para preencher alguma seção
   obrigatória, **pare e pergunte** em vez de inventar (ver guardrails).
3. **Wireframe (iniciativa com frontend):** preencha a seção "Wireframe" (§2.1 do template) com a **URL
   do artifact** e o caminho do `.html`. Sem a URL, **pare e peça** — o `/codeform:execute` (e o
   reviewer dele) partem dela. Toda tarefa de frontend das sprints cita a região/estado do wireframe
   que implementa, e cada componente marcado `NOVO` vira task própria. Se a iniciativa não toca
   frontend, escreva "Não se aplica — sem frontend" na seção.
4. **Remover** comentários de instrução (`<!-- -->`) e placeholders `{{...}}` do template.

### Fase B — Sprints

5. **Quebrar em sprints priorizando o paralelismo** (§6.1 da methodology): ao desenhar histórias e sprints,
   **busque sempre** a divisão em que o máximo delas seja independente — áreas, serviços e arquivos
   disjuntos, contratos combinados antes (interface/tipo/schema numa sprint-fundação curta) para que o
   resto rode lado a lado. Dependência só existe se for **real** (a sprint B consome algo que a A entrega);
   "ordem natural" ou conforto não é dependência. Cada sprint continua **testável sozinha**.
   **Paralelize também em nível de história.** Para cada par de histórias que cairia na mesma sprint,
   pergunte: (a) os paths das tasks são disjuntos? (b) a única dependência é um contrato (interface, tipo de
   resultado, DTO, assinatura de use case) que pode ser fixado antes? Se (a) e (b) valem, **separe-as em
   sprints distintas** e fixe o contrato numa sprint-fundação curta (ou na primeira sprint do bloco); elas
   passam a ser raias paralelas. A unidade da raia continua sendo a sprint: não existe raia por história.
   Agrupe histórias na mesma sprint só quando há dependência **real** de implementação (uma consome o
   comportamento da outra, não só o contrato) ou arquivo compartilhado que não dá para dividir, nunca por
   afinidade de tema. Registre no `PROJECT_CONTEXT` §11.1, para cada história mantida junta, o motivo
   (qual arquivo ou comportamento impede). Contrapeso: mais sprints paralelas significam mais RAM e mais
   decisões para o usuário; não fragmente além do que os paths disjuntos provam.
6. **Escrever um `SPRINT_NN_TITULO.md` por sprint** seguindo `templates/sprint.template.md`:
   - cabeçalho (objetivo da sprint + dependências, incluindo a sprint anterior);
   - histórias `### 🎫 História N: **{{COD}}-{{NNN}}** - Título` no formato **Como/Quero/Para**;
   - **Critérios de Aceite** com checkboxes `[ ]`, observáveis e verificáveis (não "deve validar", mas "rejeita X quando Y"). Se um critério só puder ser escrito de forma vaga porque o comportamento esperado não está claro no estudo, **PARE e pergunte** — critério vago é gap disfarçado;
   - **Task Breakdown** por módulo/área, cada task `{{COD}}-{{NNN}}.M` apontando o caminho do arquivo
     (**relativo à raiz do projeto**) e o detalhe técnico do que muda. Se não souber em qual arquivo
     uma task deveria bater, **PARE** — não aponte um caminho "provável";
   - **Definition of Done** da sprint.
7. **Validar coerência (e o paralelismo):** toda dependência declarada entre histórias/sprints precisa de
   justificativa concreta (o que uma entrega e a outra consome); sem justificativa, remova-a ou
   redesenhe para que rodem em paralelo. Além disso: nomes de campos/entidades compartilhados entre histórias de sprints
   diferentes devem ser consistentes; dependências entre sprints devem estar explícitas nas seções
   "Dependências" de cada arquivo. Qualquer inconsistência encontrada aqui (nome divergente, dependência
   implícita, sprint que assume algo que outra sprint ainda não entrega) é motivo para **PARE e
   debata** antes de seguir para a Fase C — não ajuste silenciosamente escolhendo uma das versões.

### Fase C — SPRINTS_OVERVIEW.md

8. **Escrever `docs/Sprints/<Iniciativa>/SPRINTS_OVERVIEW.md`** (ordem oficial de implementação, regra
   de execução, resultado esperado), linkando cada `SPRINT_NN_*.md` gerado na Fase B. Se a ordem lógica
   entre sprints admitir mais de uma sequência válida, **PARE** e apresente as alternativas antes de
   fixar uma.


### Fase D — Plano de paralelização, flags e tier

9. **Flags de risco e tier em cada sprint.** Preencha a seção "Flags de risco e paralelização" do
   `sprint.template.md`: as flags (lógica de negócio crítica · superfície de segurança — entrada externa,
   auth, persistência, dado sensível, LLM · API pública/contrato · migração de dado em produção ·
   frontend novo), a **paralelização** e o **tier** (`leve|médio|pesado|crítico`, tabela do §6.2 da
   methodology, a partir das flags e do tamanho). **O execute decide o security lendo estas flags** —
   flag omitida é gap: na dúvida, declare.
10. **Plano de paralelização** (PROJECT_CONTEXT §11.1, contrato do §6.1): o veredito `serial` exige
    justificativa por escrito (qual arquivo/contrato impede) — se o desenho saiu serial, tente de novo a
    quebra do passo 5 antes de aceitar. Extraia os **paths de todas as
    tasks** de cada sprint, **calcule você mesmo as interseções** entre sprints candidatas a paralelas,
    nomeie os **arquivos de contenção** com **dono único**, defina **blocos** e **ordem de merge**, e dê o
    **veredito** (`serial|parcial|total`). Interseção não vazia, ou contenção sem dono único = **serial**.
    Se houver mais de um desenho válido de blocos, **PARE** e apresente as alternativas (§0.2). Reflita
    os blocos na seção "Blocos de execução" do `SPRINTS_OVERVIEW.md`.
    **Formato do §11.1: diagrama em traços (ASCII, bloco ```text), nunca tabela e nunca Mermaid**, com os
    blocos, as setas `|`/`v`, as sprints paralelas lado a lado, a linha `merge: SNN -> SMM`, o tier entre
    colchetes e, abaixo, a lista "Paths que as tasks tocam". Siga o modelo de
    `templates/project-context.template.md` §11.1.
11. **Apresentar** a lista de sprints, a ordem, os blocos, o veredito, o tier de cada uma e o caminho dos
    artefatos.

### Gate 1 — aprovação dos artefatos
Sem aprovação explícita, o dry-run **não** roda. Correções do usuário voltam pelos passos 2–10 só no que
mudou.

## Parte 2 — Dry-run (subagent limpo)

12. **Spawne o subagent** (`Agent`, `subagent_type: "general-purpose"`, `model: "opus"`, `effort: "high"` —
    tabela do §6.2). Brief fechado: *"Siga `~/.claude/skills/codeform/playbooks/dry-run.md`.
    Iniciativa: `docs/Sprints/<Iniciativa>/`. Leia na íntegra `PROJECT_CONTEXT.md`, `SPRINTS_OVERVIEW.md` e
    todos os `SPRINT_NN_*.md`, e o wireframe se houver. Valide inclusive o plano de paralelização (§11.1).
    Read-only: não altere nada. Devolva o relatório D1 e a cobertura."* Não passe suas conclusões nem o
    estudo: o subagent deve discordar com a cabeça limpa.
13. **Verifique o relatório.** Todo item ❌/⚠️ citando trecho: abra o trecho e confirme que existe e que o
    problema procede. Descarte o que não se sustenta, **dizendo ao usuário** o que descartou e por quê.
    `PARADA` do subagent vai ao usuário e a resposta volta por `SendMessage`.
14. **Apresente o relatório D1 ao usuário** (§0.8): resumo executivo, tabela de gaps que exigem decisão,
    decisões pendentes por sprint, resultado da validação do plano de paralelização. **Aguarde a
    aprovação** — nada é escrito antes.

### Gate 2 — o usuário resolve os gaps

15. **Aplique as atualizações aprovadas** em `SPRINT_NN_*.md`, `PROJECT_CONTEXT.md` e
    `SPRINTS_OVERVIEW.md`. **Nunca apague** decisões registradas — reescreva ou complemente.
16. **Re-validação.** Releia as seções alteradas. Se a mudança alterou o plano de paralelização, os tiers
    ou mais de três sprints, **retome o subagent** (`SendMessage`) restrito às seções alteradas. Reporte:
    ✅ Pronto para execução | ⚠️ Itens pendentes de decisão do usuário.
17. **Registre o estado do plano** na seção "Estado do plano" do `SPRINTS_OVERVIEW.md` (`Dry-run: ✅
    aprovado em <data>`): é o que o `/codeform:orchestrate` confere antes de despachar.
18. **Sub-issues (só com plano ✅ aprovado e issue de origem).** Antes, liste as sub-issues que a pai já
    tem (§2 da methodology): se o plano foi refeito, **atualize** as existentes (corpo) em vez de duplicar,
    e atualize também o corpo da issue pai se o plano mudou informação dela. Para cada
    `### 🎫 História N` dos `SPRINT_NN_*.md` sem sub-issue, crie uma da issue pai, **só para acompanhamento do usuário**:
    - título e corpo vêm do template de sub-issue/História do repositório, se existir em
      `.github/ISSUE_TEMPLATE/` (mesma leitura do `/codeform:issue` §3). Regra de título que dependa da pai
      (ex.: prefixo): **copie do título da issue pai**. Sem template: título = título da história precedido do
      prefixo `[..]` da pai, se ela tiver um; corpo = critérios de aceite em checklist + o caminho do
      `SPRINT_NN_*.md`. Não copie o conteúdo da sprint: o arquivo é a fonte de verdade;
    - `url=$(gh issue create -t ... -F <arquivo.md>)`, depois vincule à pai:
      `gh api repos/{owner}/{repo}/issues/<pai>/sub_issues -F sub_issue_id=$(gh api repos/{owner}/{repo}/issues/<n> -q .id)`;
    - **Board:** se o `CLAUDE.md` do projeto diz que sub-issue não fica no board (ou como tratar um
      "auto-add"), siga-o logo após criar e vincular e releia o board para confirmar. Sem essa regra, não
      mexa no board;
    - não grave os números das sub-issues nos arquivos; liste-os ao usuário. Sem issue de origem, pule.
</process>

<guardrails>
- **Sem código de produto.** Apenas documentação de planejamento.
- **Contrato de Pair Programming** (§0): leia o real e cite `caminho:linha`; suba confusões e suposições.
- **A decisão é do usuário** (§0.2): como quebrar as histórias, nome de campos, desenho dos blocos — em
  toda bifurcação, **PARE**, apresente as opções e espere a escolha.
- **Sem legado (código), dados preservados (produção):** o que a iniciativa substitui está mapeado para
  remoção; dado já em produção tem a **migração** como task própria — nunca remoção sem preservação.
- **Não inventar campos ou regras** sem sinalizar como suposição no corpo da história ou do
  `PROJECT_CONTEXT.md`. Faltou informação para uma seção obrigatória? **Pare e pergunte.**
- **Paralelismo que o plano não prova não existe.** Interseção de paths calculada por você, não presumida;
  na dúvida, serial.
- **O dry-run não é seu.** Você não o executa no próprio fio, não passa a ele suas conclusões, e não
  reescreve a sprint antes do Gate 2.
- **Suba a confusão, não a esconda** (§0.3): terminar de escrever todas as sprints para só então levantar
  dúvidas é tarde demais — o gap já foi materializado em artefato que o execute seguirá ao pé da letra.
- **Não commite, não faça push.**
</guardrails>

<gate>
Com o plano **aprovado** (✅ Pronto para execução), indique a rota pelo veredito do §11.1:

- `serial`: "Plano aprovado, sem paralelismo. Rode **`/codeform:execute SPRINT_01`** (e as demais, na ordem do
  overview)."
- `parcial` ou `total`: "Plano aprovado, com blocos paralelos. Rode **`/codeform:orchestrate`** — ele despacha
  os blocos independentes; as sprints seriais seguem por `/codeform:execute`."

Se houver itens pendentes: "Resolva os gaps listados acima antes de iniciar a execução."
</gate>
