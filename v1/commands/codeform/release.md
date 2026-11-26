---
name: codeform:release
description: "Liberar a branch — valida as convenções que o projeto declara (commits, título do PR, branch como aviso) no fio principal, confere a sincronia das issues, roda build/lint/test das unidades tocadas em subagents mecânicos (um por unidade, em paralelo) e, se tudo ✅ e sem avisos, abre o PR. Genérico: descobre as regras e a receita de CI no repositório atual."
argument-hint: "[--base <branch>] (branch de comparação; default: a base de trabalho que o projeto declara, ou a branch padrão do repositório)"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Task
  - Agent
---

<execution_context>
Antes de prosseguir, leia integralmente `../../skills/codeform/methodology.md` — em especial o §0 (Contrato
de Pair Programming) e o §0.7 (delegação).

**Este command não sabe nada do projeto de cabeça.** As regras e a receita de CI vêm do repositório atual,
lidas a cada execução:
- `CLAUDE.md` (raiz e dos serviços), `CONTRIBUTING.md` e os docs que eles apontam — formato de commit e de
  branch, tipos aceitos, regra de título de PR, método de merge, branch-base de trabalho, regras de issue/board.
- `.github/workflows/` (ou o CI que o projeto usar) — o workflow que **valida convenções** (se houver) e a
  receita real de build/lint/test de cada unidade, incluindo os workflows reutilizáveis que ela herda.
</execution_context>

<source_of_truth>
⚠️ **A fonte da verdade é o que o projeto declara e o CI que roda de verdade, não a sua memória de outros
repositórios.**

Aqui não se julga arquitetura: **audita-se conformidade** com um padrão que já existe. Diante disso:

- Se a prosa de convenções (`CLAUDE.md`/`CONTRIBUTING`) divergir do que o workflow de validação **de fato**
  checa, isso é **achado** (drift entre documentação e enforcement) — reporte, não resolva escolhendo um dos
  dois em silêncio.
- **Regra que o projeto não declara não é validada.** Sem regra de título de PR, sem regra de "uma unidade por
  commit" ou sem convenção de branch, o item correspondente é pulado **e o relatório diz que foi pulado e
  por quê**. Não importe regra de outro projeto.
- Não invente comando de build/lint/test. Se o CI da unidade não define um passo, use o default do workflow
  reutilizável que ela chama; se nem ele tem o passo, **não rode nada no lugar** e diga isso no relatório.
- Repositório sem CI legível: descubra os comandos nos manifestos (`package.json`, `Makefile`, `go.mod`,
  `pyproject.toml`…), **declare a suposição** e peça confirmação antes de rodar.

**Não corrija nada por conta própria — reporte e traga para debate** (§0.2 da methodology).
</source_of_truth>

<delegation>
⚠️ **Subagent só para execução mecânica em paralelo — nunca para julgar** (§0.7 da methodology).

Build, lint e test são **determinísticos**: exit code + log, sem ambiguidade a resolver.

- ✅ **Permitido:** você resolve os comandos reais de uma unidade (CI + reutilizável) e despacha **um subagent
  por unidade tocada**, em paralelo, com instrução fechada: *"rode exatamente estes comandos, nesta ordem, no
  diretório `<unit_path>`, e devolva o exit code e o log bruto (stdout+stderr) de cada um — nada além disso."*
- ❌ **Proibido:** pedir ao subagent para resumir o log, decidir se a falha é grave, sugerir correção ou
  interpretar o resultado.

**Quem lê o log bruto, decide o veredicto e escreve o relatório é você.** Se um subagent devolver algo que
não seja exit code + log (ex.: "tudo passou"), trate como incompleto e leia o log antes de reportar.

A validação de convenções (Bloco A) e a **abertura do PR** (Bloco D) **nunca** delegam: são determinísticas,
cabem no fio principal, e a segunda é uma ação visível para outras pessoas.
</delegation>

<objective>
Validar que a branch atual está pronta e, estando, **abrir o PR** no repositório atual:

1. **Convenções** (Bloco A) — as que o projeto declara (commits, título do PR, "unidade por commit"…),
   replicando a lógica do workflow de validação, se existir; o nome da branch entra só como aviso.
2. **Issues** — critérios de aceite da pai e das sub-issues todos `[x]` e sub-issues fechadas; senão, **o PR não é aberto** (passo 6).
3. **Build/Lint/Test** (Bloco B) — para cada unidade que a branch tocou, a receita real de CI dela.
4. **PR** (Bloco D) — só se tudo ✅ **e sem nenhum aviso**.

Até o Bloco D, este comando é **read-only**: reporta o que está quebrado e aguarda decisão do usuário antes
de qualquer correção.
</objective>

<when_to_apply>
**Obrigatório sempre**, para toda branch, antes de abrir ou atualizar um PR. É **por branch, não por
sprint**: roda uma vez, depois que **todas** as sprints da branch passaram pelo `/codeform:execute` (review e
security inclusos) — e, se houve paralelismo, depois do `/codeform:orchestrate --merge`.
</when_to_apply>

<scope>
O escopo é a **branch atual comparada com uma branch-base** (`--base`).

1. **Commits no escopo:** `git log <base>..HEAD` — todo commit que a branch atual tem e a base não.
2. **Merge commits são ignorados no Bloco A**: um commit com mais de um parent (ex.: sync da base na branch)
   tem diff que cruza unidades por natureza e não pode ser tratado como violação. Confirme no workflow de
   validação do projeto, se houver, que ele faz o mesmo.
3. **Unidades tocadas:** a união das unidades (serviço, pacote, app) que os arquivos dos commits no escopo
   tocam. O mapa `path → unidade` vem do projeto (workflow de validação, filtros de path dos workflows de CI
   ou `CLAUDE.md`); num repositório de uma unidade só, é a raiz. Só as tocadas entram no Bloco B.
4. **Fora do escopo:** unidades que a branch não tocou não são buildadas/lintadas/testadas. Algo quebrado de
   passagem numa não tocada é observação, não expansão de escopo.
</scope>

<process>
Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo.

### Setup

0. **Pré-condições do release** — qualquer falha: **pare e diga o que falta**.
   - Branch atual **não** é uma branch protegida (methodology, "Branch-base de trabalho") nem uma branch efêmera de raia.
   - `git status --porcelain` **vazio** (há trabalho não commitado → o PR não representaria a branch).
   - Se a branch contém uma iniciativa (`docs/Sprints/<Iniciativa>/` tocado nos commits): **toda** sprint do
     `SPRINTS_OVERVIEW.md` está concluída (`[x]`) **e** tem a seção "Registro dos gates" preenchida (review ou
     dispensa com motivo; security ou "não se aplica" com motivo; **riscos aceitos** listados). Sprint sem
     registro = a execução não passou pelos gates: **bloqueie** e mande rodar `/codeform:execute`.

1. **Resolver a base:** `--base`; senão siga "Branch-base de trabalho" da methodology. Confirme que existe:
   `git rev-parse --verify <base>`. Se não existir ou houver mais de uma candidata, **PARE e pergunte**.
2. **Ler as fontes reais** (fresco, não de memória): as regras de convenção do projeto (`CLAUDE.md`,
   `CONTRIBUTING`), o workflow de validação de convenções (tipos aceitos, mapa path→unidade, exceções como
   arquivos gerenciados) e, para cada unidade do passo 4, o workflow de CI + os reutilizáveis que herda.
3. **Levantar os commits no escopo:** `git log <base>..HEAD --format='%H %P'` (merge commits pelo nº de
   parents) e, para cada commit não-merge, `git show --name-only --format=`.
4. **Mapear unidades tocadas:** aplique o mapa `path → unidade` aos arquivos de todos os commits do passo 3,
   respeitando as exceções que o projeto declara (ex.: arquivos gerenciados por ferramenta de release).

---

### Bloco A — Convenções (nunca delegado)

Cada item abaixo só vale **se o projeto declara a regra** (passo 2). Sem regra: pule e registre.

5. **Nome da branch:** se o projeto declara um padrão, reporte como **aviso** (nunca falha) o que não o
   seguir.
6. **Issues concluídas** (só com issue de origem — `<n>` em `docs/Sprints/<Iniciativa>/`): siga o "Localizar"
   do §2 da methodology e releia a pai e **todas** as sub-issues (`gh issue view <n> --json state,body`).
   **❌ Bloqueia o PR** se qualquer um destes for verdade: (a) a pai tem critério de aceite `- [ ]`; (b) alguma
   sub-issue tem critério `- [ ]`; (c) alguma sub-issue está aberta; (d) uma sub-issue não casou com nenhuma
   História (não dá para provar que foi concluída). Liste número + o que falta. Não edite aqui (read-only):
   proponha rodar a sincronização do `/codeform:execute` §19a (ou, depois de um `--merge`, o passo 11 do
   `/codeform:orchestrate`). Critério que o usuário decidir não cumprir é decisão dele: ele atualiza a issue
   (ou autoriza explicitamente prosseguir); você nunca marca `[x]` para destravar o PR.
7. **Título do PR**, se o projeto declara regra e já existe PR (`gh pr view --json title` — se falhar porque
   não há PR ainda, **pule e diga isso**): falha se violar a regra (ex.: exigir ou proibir o formato de
   commit convencional).
8. **Commits:** aplique **só** as regras de commit que o projeto declara — formato da mensagem e, se houver,
   "no máximo uma unidade por commit". Para a regra de unidade, colete por commit não-merge o conjunto de
   unidades que seus arquivos tocam (passo 4, por commit, não a união); mais de 1 é violação — registre `sha`,
   as unidades e a mensagem. A regra é **por commit**, não pela branch: uma branch multi-unidade legítima, com
   um commit por unidade, está correta.

---

### Bloco B — Build/Lint/Test (delegação mecânica permitida)

9. Para cada unidade do passo 4: resolva a receita real (passo 2) e monte a lista exata de comandos, na ordem
   do CI daquela toolchain.
10. Despache **um subagent por unidade tocada, em paralelo** (`<delegation>`; `Agent`, `subagent_type:
    "general-purpose"`, `model: "haiku"`, `effort: "low"`): instrução fechada com os comandos exatos e o
    `unit_path`, retorno exigido = exit code + log bruto de cada comando, nada mais.
11. **Leia o log bruto de cada subagent você mesmo** antes de classificar qualquer coisa como ✅/❌. Um exit
    code 0 sem log legível ainda precisa ser conferido — não confie só no número.

---

### Bloco C — Relatório

12. **Matriz de resultado:**

    | Unidade | Toolchain | Lint | Test | Build | Evidência |
    |---|---|---|---|---|---|
    | `<unidade>` | `<toolchain>` | ✅ | ❌ | — (bloqueado) | trecho do log |

    Uma célula `—` significa "o CI desta unidade não tem este passo" (com o motivo), nunca "não rodei".
    Diferencie os dois explicitamente. Reporte exatamente a cobertura real do CI; nunca "complete" uma unidade
    com um passo que ela não tem.

13. **Violações e avisos** (Bloco A), no formato:
    ```
    ### ⚠️ <aviso>: <detalhe>
    **Esperado:** <regra declarada pelo projeto, com a fonte>
    ```
    ```
    ### ❌ Commit <sha> viola <regra>: <detalhe>
    **Mensagem:** <primeira linha do commit>
    **Correção recomendada:** <o que a regra pede>
    ```

---

### Bloco D — Veredicto, decisão e PR

14. **Veredicto:** ✅ **Pronto para PR** (pré-condições, Bloco A sem violações **e sem avisos**, Bloco B todo
    ✅ ou `—`) | ⛔ **Bloqueado** (qualquer ❌) | ⚠️ **Com avisos** (nenhum ❌, mas há avisos — ex.: branch
    fora da convenção). Issue pendente (passo 6) é ❌, não aviso.
15. **⛔ ou ⚠️ — pare.** Reporte e **aguarde a decisão** (§0.2). Se uma violação tiver mais de uma forma
    válida de correção, **PARE**, apresente as opções e espere a escolha. **Não abra PR.** Correção
    aprovada: cirúrgica, com o diff, rodando **só o check afetado** e colando a saída; depois confirme que
    o item virou ✅ sem reabrir os que já estavam ✅. Mexer no histórico (`git rebase`) é operação
    destrutiva — só com aprovação explícita.
16. **✅ — abra o PR você mesmo:**
    a. **Push da branch de trabalho:** `git push -u origin HEAD`. **Nunca** a branch padrão nem a base,
       nunca `--force`.
    b. **Já existe PR para a branch?** (`gh pr view`). Existe: o push já o atualizou — reporte a URL e
       pare; **não** crie outro.
    c. **Título:** siga a regra de título de PR que o projeto declara (formato, se deve ou não ser
       convencional) e **valide-o contra ela** antes de criar. Sem regra: um título curto que descreva o
       objetivo.
    d. **Template:** se existir `.github/pull_request_template.md` (ou equivalente), use-o como estrutura do
       corpo. Sem template, o corpo, no idioma do repositório, tem: (i) **Resumo** — o objetivo (§2 do
       `PROJECT_CONTEXT`); (ii) **Sprints** — o que cada uma entregou; (iii) **Verificação** — a matriz do
       Bloco C; (iv) **Riscos aceitos** — o que o usuário aceitou nos "Registro dos gates" (severidade +
       justificativa), ou "nenhum"; (v) o método de merge que o projeto exige, se declarado. Termine com a
       linha de atribuição de PR da sessão. Não escreva `@nome` solto no corpo (notifica a pessoa).
    e. `gh pr create --base <base> --title "<título>" --body "<corpo>"` e reporte a **URL**.
</process>

<guardrails>
- **Read-only até o Bloco D.** Nenhuma edição, nenhum `git rebase`, nenhum commit reescrito sem aprovação
  explícita.
- **PR só com issues concluídas** (pai e sub-issues com critérios `[x]`, sub-issues fechadas) **e** ✅ com zero avisos. Aviso ou falha = parar e perguntar. A autorização para abrir o PR vale
  apenas para esse caso verde.
- **Push só da branch de trabalho**, nunca da branch padrão/base, nunca com `--force`.
- **Nunca rode build/lint/test de uma unidade fora do escopo** (passo 4).
- **Nunca invente cobertura nem regra:** passo de CI inexistente → `—` com o motivo; regra não declarada pelo
  projeto → item pulado e dito no relatório.
- **A regra de unidade é por commit, não pela branch** (passo 8) — não bloqueie uma branch multi-unidade
  legítima.
- **Sprint sem "Registro dos gates" bloqueia** (passo 0): a execução não passou pelos gates de qualidade.
- **Suba a confusão, não a esconda** (§0.3): se a documentação e o workflow de validação divergirem, ou se
  uma unidade não tiver receita clara, registre como achado e pergunte.
</guardrails>

<gate>
PR aberto: "Release concluído. PR: <URL> (base `<base>`)." Acrescente o método de merge que o projeto exige,
se declarado.

Se ⛔ **Bloqueado**: "Release bloqueado. Resolva os itens do relatório — build/lint/test quebrado ou issue
com critério pendente/sub-issue aberta impede o PR."

Se ⚠️ **Com avisos**: "Release com avisos — o PR não foi aberto. Decida: corrigir os avisos ou me autorizar a
abrir o PR assim."
</gate>
