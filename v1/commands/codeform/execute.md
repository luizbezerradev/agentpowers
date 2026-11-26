---
name: codeform:execute
description: "Implementa UMA sprint na fonte, seguindo as histórias/critérios ao pé da letra (regras MANDATORY, lint, tasks [x]) e, ao concluir, spawna o reviewer (e o security, se a flag pedir): corrige o simples, escala o sensível. Com --lane, roda como raia do /codeform:orchestrate. Substitui /sprint:execute + /sprint:review + /sprint:security."
argument-hint: "<sprint (número, arquivo ou título)> [--initiative <NomeDaPasta>] [--lane]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Skill
  - Artifact
  - Task
  - Agent
  - SendMessage
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---

<execution_context>
Antes de prosseguir, leia integralmente o arquivo de metodologia abaixo — princípios e regras desta fase,
em especial o §4 (Clean Code, 9 Object Calisthenics, YAGNI/KISS/DRY/Less Code Best Code), o §5 (SDD/TDD), o
§0.7 (delegação), o §6.2 (tiers) e o §7 (achados simples × sensíveis):
`../../skills/codeform/methodology.md`

Se a sprint toca frontend, leia também o **wireframe**: a URL do artifact está na seção "Wireframe" do
`PROJECT_CONTEXT.md`; leia o `WIREFRAME_*.html` correspondente na íntegra (`Artifact` com `action:
"read"` na URL, se o `.html` local não estiver disponível).
</execution_context>

<source_of_truth>
⚠️ **LEIA ANTES DE TUDO — este bloco se sobrepõe a qualquer outra instrução deste arquivo, inclusive
ao "seguir a sprint ao pé da letra".**

Não é porque está no `CLAUDE.md`, nem porque está escrito na sprint, nem sequer porque já está
implementado no código, que aquilo está correto e deve ser tratado como **fonte da verdade**. O que é
correto é o que é **conceitual e arquiteturalmente correto** — e é isso que deve ser levado em
consideração em cada decisão de implementação.

Não fique tentando modelar a nova implementação no que já existe só porque já existe e já está lá,
como se tudo tivesse que se modificar para funcionar do jeito que está. O que se quer é o **inverso**:
se ao implementar você identificar que o que existe está conceitual ou arquiteturalmente incorreto, ou
é um workaround, e a sprint e o `CLAUDE.md` seguem esse erro, então **temos que debater e corrigir
tudo** — para que a nova implementação fique correta e, se necessário, corrigindo também o que já
existe, tendo em mente que utilizamos **Clean Architecture e SOLID**.

Sendo assim, em **toda** decisão de implementação — grande ou pequena, prevista na sprint ou não:

> Qual o caminho mais enterprise e conceitualmente e arquiteturalmente correto, que resolve o problema
> **na fonte**, sem workarounds, sem seguirmos com dívida técnica e que é **prova de futuro**?

Isso vale **independente do esforço necessário para implementação**, desde que atenda a estes
requisitos. Ao apontar um desvio, **seja didático**: explique qual formato você recomenda e por quê.
Lembre-se: a sprint é um guia, **não uma verdade absoluta**.

**Não implemente ainda aquele trecho — pare e traga para debate antes de escrever o código.**
</source_of_truth>

<delegation>
⚠️ **A implementação é sua, no fio principal; a revisão é de subagents novos** (§0.7 da methodology).

- **Implementação — nunca delegada.** Escrita de código e decisão de design a cada linha são julgamento, e
  o `<source_of_truth>` acima manda parar e debater antes de codar a parte duvidosa. Se o volume parecer
  grande demais para uma sessão, **pare e diga isso ao usuário**.
- **Review e security — subagents novos, de contexto limpo**, spawnados por você **ao concluir a sprint**
  (passos 12–19), seguindo os playbooks `~/.claude/skills/codeform/playbooks/review.md` e `security.md`.
  Eles são read-only e não falam com o usuário; **você** verifica o que eles trazem, corrige o que é simples
  e leva o resto ao usuário (§7 da methodology).
- **Por que você pode spawnar:** subagent não spawna subagent. Se você está rodando como raia do
  orchestrator, é uma **sessão** independente, não um subagent — por isso o `/codeform:orchestrate` nunca
  despacha raias com `Agent`.
</delegation>

<lane_mode>
🛤️ **`--lane`: você é uma raia do `/codeform:orchestrate`.** O worktree, a branch efêmera e os recursos de teste
isolados da raia (portas, nome de ambiente… as variáveis de ambiente que o orchestrator exportou, se o projeto
as exige) já foram preparados — **não os altere**. O que muda:

- **Estado compartilhado:** grave/atualize `$(git rev-parse --git-common-dir)/sprint-lanes/<Iniciativa>/SPRINT_NN.md`
  (fora da árvore versionada) com `estado` (`executando` · `aguardando-decisao` · `concluida` · `bloqueada`),
  os SHAs dos commits e as **decisões pendentes**. Atualize ao iniciar, a cada checkpoint, ao parar para o
  usuário e ao concluir.
- **Parada humana:** quando precisar do usuário (bifurcação de design, achado sensível/CRÍTICO), marque
  `aguardando-decisao`, apresente a decisão **nesta sessão** e espere. O usuário consolida as pendências de
  todas as raias com `/codeform:orchestrate --status`.
- **Commits:** autorizados pelo gate do orchestrator **apenas nesta branch efêmera**. Commits no formato e
  na granularidade que o `CLAUDE.md` do projeto declara (ex.: um por unidade/serviço, mensagem convencional),
  com o trailer de atribuição exigido ali. **Nunca `git push`.**
- **Lint/typecheck só nas unidades da raia** — um lint com auto-fix no repositório inteiro reformata arquivos
  de outras raias.
- **Escopo:** só os arquivos da sua sprint. Precisar de um arquivo de contenção que o plano (§11.1) atribui a
  outra raia é uma **bifurcação**: pare e peça decisão.

Sem `--lane`: **não commite** a menos que o usuário peça.
</lane_mode>

<objective>
Implementar a sprint indicada (`$ARGUMENTS`) seguindo o `SPRINT_NN_*.md` ao pé da letra, na **ordem
lógica** definida no `SPRINTS_OVERVIEW.md`, e então submetê-la a **review** (e, se aplicável,
**security**) por subagents independentes. É um **projeto real** — nada pode ser simulado ou hardcoded.
</objective>

<guiding_question>
Ao aplicar o `<source_of_truth>` em cada decisão, tenha em mente: "enterprise" **não** é sinônimo de
"mais complexo". A solução correta ainda deve ser a mais simples que resolve o problema por completo
(YAGNI/KISS) — sem abstração especulativa, sem "flexibilidade" que ninguém pediu. O que o esforço
maior nunca justifica é o contrário: aceitar workaround, débito técnico ou solução conceitualmente
errada.

Aplicado a esta fase: se, ao implementar, você identificar que
o `SPRINT_NN_*.md` pede um caminho conceitualmente ou arquiteturalmente errado — mesmo que já
tenha passado pelo dry-run do `/codeform:plan` —, **não implemente ainda**: pare, explique didaticamente por
que o caminho da sprint está errado, qual caminho você recomenda e por quê, e **traga para debate**
antes de escrever qualquer código.
</guiding_question>

<frontend_wireframe>
🖼️ **Todo frontend desta sprint é implementado com base no wireframe** (artifact linkado no
`PROJECT_CONTEXT.md`).

- Estrutura, componentes, estados e textos saem do wireframe — que já foi construído com os
  componentes reais do frontend. Reuse os mesmos componentes e tokens que ele declara na legenda.
- Região marcada `NOVO` no wireframe: o componente é criado nesta sprint, seguindo o desenho.
- **Divergência entre o wireframe e o que a implementação exige** (componente não se comporta como
  desenhado, estado que faltou, decisão nova): **PARE** (§0.2), traga para debate e, aprovada a
  mudança, **atualize o artifact** antes de seguir.
- **Se o frontend mudar durante o execute por qualquer motivo, o artifact tem de ser atualizado**:
  edite o `WIREFRAME_*.html` só no que mudou e republique **na mesma URL** (siga o `<update_mode>` do
  playbook `~/.claude/skills/codeform/playbooks/wireframe.md`, no próprio fio ou via subagent). A história de frontend **não
  fecha** (`[x]`) com wireframe desatualizado.
- Nunca implemente de cabeça uma tela que o wireframe cobre, nem deixe o wireframe atrás do código.
</frontend_wireframe>

<mandatory>
Estas regras são **inegociáveis** (detalhe completo na §3 da methodology):

1. **PROIBIDO tipagem fraca** (`any` etc.). Veja a tipagem correta em `node_modules`/stubs quando
   necessário.
2. **Rode lint/typecheck ao final** e **corrija sempre na fonte — sem workarounds**.
3. **Use o caminho correto do repositório/pacote** ao modificar arquivos.
4. **Sem legado (código), dados preservados (produção):** **substitua** a implementação anterior
   quando a sprint mudar — sem manter código morto ou compatibilidade não pedida. Essa regra é sobre
   **código**. Já existe **produção**: se a task mudar schema, formato ou local de um dado existente,
   **migre** — nunca descarte ou perca dado real. Se a sprint não deixou a migração explícita,
   **PARE** (§0.2) antes de rodar qualquer coisa destrutiva contra dados de produção.
5. **Sempre na fonte:** nunca workaround nem complexidade desnecessária que não resolve o que a sprint
   pede.
6. **Ao concluir uma história, marque `[x]`** em todos os critérios de aceite e tasks dela no
   `SPRINT_NN_*.md`.
7. **OBRIGATÓRIO antes de iniciar:** leia por completo o `PROJECT_CONTEXT.md` e o `SPRINT_NN_*.md`
   alvo, veja o que já está implementado e faça busca via context7 para libs tocadas. Somente após
   isso dê início à execução.
8. **Clean Code + 9 Object Calisthenics + YAGNI/KISS/DRY/Less Code Best Code** (§4 da methodology) em
   **todo** código novo — não é opcional nem "se der tempo":
   - **Clean Code:** nomes que revelam intenção, funções pequenas e com um só nível de abstração,
     sem efeitos colaterais escondidos, e **comentário só pelas seis categorias da §4.3** (regra 11
     abaixo).
   - **9 Object Calisthenics** (§4.1): um nível de indentação por método; sem `else` (early return);
     envolver primitivos/strings com semântica de domínio em Value Objects; coleções em classes
     próprias; uma chamada de ponto por linha (Law of Demeter); sem abreviações; entidades ≤ 50
     linhas / métodos ≤ 10 linhas; ≤ 2 variáveis de instância por classe; sem getters/setters
     (Tell, Don't Ask).
   - **YAGNI:** só o que a história pede — nada especulativo "para o futuro".
   - **KISS:** a solução mais simples que satisfaz os critérios de aceite vence sempre.
   - **DRY:** lógica em um único lugar canônico; extraia só a partir de 3+ ocorrências concretas.
   - **Less Code Best Code:** menos código é menos bug — remova o que a sprint permitir remover.
   - Em código legado tocado pela task: **melhore o que for alterado, nunca piore.**
9. **SDD/TDD** (§5 da methodology): parta dos critérios de aceite da história (SDD); quando a lógica
   for isolável, escreva o teste antes do código (TDD: Red → Green → Refactor).
10. **Tipos de teste por criticidade** (§5.3 da methodology — escalonado, não uniforme): a cada
    história, aplique unitário + integração + e2e + regressão + coverage + mutação conforme a matriz
    do §5.3. Mínimo inegociável: lógica isolável → unitário + **coverage 100%** (linha/branch cobertos
    por asserção real, não só execução); toda história → suíte de regressão completa passando ao
    final; lógica de negócio crítica (cálculo, regra de domínio, auth, fluxo financeiro) → mutation
    testing sem mutante sobrevivente; fluxo que atravessa camadas reais → teste de integração sem
    mock no ponto validado; fluxo crítico/ponta a ponta → e2e.
11. **Comentário é exceção, não padrão** (§4.3 da methodology). O padrão é **não escrever
    comentário**; cada um que sobrevive se justifica por uma das **seis categorias** da lista fechada
    (regra de negócio implícita · decisão não-óbvia com link para ADR · aviso de consequência
    não-local · contrato público TSDoc/GoDoc · fonte externa/justificativa de `eslint-disable` ·
    `TODO(ID)` rastreável). Três consequências operacionais nesta fase:
    - **PROIBIDO comentário-changelog.** Nada de *"havia um `status` aqui e ele saiu na LDD-452"*,
      *"antes era X"*, *"morreu na sprint Y"*, *"este campo deixou de existir"*. O arquivo carrega o
      **estado vigente**, não a história da mudança. Essa narrativa é valiosa e tem **destino
      próprio**: corpo do commit, ADR, ou o `SPRINT_NN_*.md`/`PROJECT_CONTEXT.md` da iniciativa —
      **escreva-a lá**, não no código. Também proibido: paráfrase do código, banner decorativo
      (`═══ … ═══`, `// ---- helpers ----`), código comentado, comentário que salva nome ruim.
    - **Orçamento:** bloco ≤ 5 linhas; densidade do arquivo ≤ 10% (`grep -cE '^\s*(//|\*|/\*)'`
      contra `wc -l`). Acima de 10%, justifique bloco a bloco ou corte.
    - **Oportunidade:** arquivo novo **nasce** no padrão. **Todo arquivo que a task editar tem os
      comentários do arquivo inteiro normalizados** — não só as linhas do diff (a §0.1 já obriga ler
      o arquivo inteiro; o que se lê, se corrige). Arquivo que a task **não** tocou: **não edite** —
      registre caminho + densidade para o fechamento (passo 20).
</mandatory>

<process>
Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo. Nada de assumir, nada de
"feito" sem prova, a decisão de design é do usuário.

Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo. Nada de assumir, nada de
"feito" sem prova, a decisão de design é do usuário.

**Setup (uma vez):**
1. **Resolver a sprint** (`--initiative` ou confirme) e abrir o `SPRINT_NN_*.md` alvo.
2. **Pré-leitura obrigatória** (MANDATORY 7 + §0.1): leia `PROJECT_CONTEXT.md`, a sprint, o
   **wireframe** (se a sprint toca frontend — `<frontend_wireframe>`) e os arquivos existentes que ela
   toca. Reporte o estado atual citando `caminho:linha`. **Não assuma o que existe.** Registre também:
   - `BASE_SHA` = `git rev-parse HEAD` (de onde a sprint parte);
   - o **snapshot do que já estava sujo** (`git status --porcelain`): esses arquivos **não** são da sprint
     e ficam fora do diff que os revisores recebem;
   - o **tier** e as **flags de risco** da sprint (seção "Flags de risco e paralelização").
3. **context7** para cada lib/framework/SDK que a sprint toca.

**Loop por história (repita para CADA história, na ordem):**
4. **Plano da história:** liste as tasks do Task Breakdown e como vai verificar cada critério de
   aceite. Para cada task, **classifique se a lógica é isolável** pelo critério do §5.2 da
   methodology (lógica de negócio, função pura, serviço com contrato claro, ou critério
   determinístico mapeável 1:1 para caso de teste) ou se cai na exceção (integração com
   infraestrutura externa, código de UI/visual, scaffolding sem lógica). Essa classificação decide
   se o passo 7 (TDD) se aplica a esta task. Classifique também, pela matriz do §5.3, **quais tipos
   de teste** (unitário/integração/e2e/regressão/coverage/mutação) esta task exige — não é uniforme:
   uma task de DTO trivial não pede mutation testing, uma regra de cálculo financeiro pede.
5. **Decisão do usuário** (§0.2): qualquer decisão durante a implementação que a sprint não cobriu —
   ou que a sprint cobre de um jeito que não é o mais enterprise/correto — deve ser **imediatamente
   comunicada**, aplicando o `<guiding_question>` acima: apresente o problema, as opções com
   prós/contras, sua recomendação didática e o porquê, e **espere a escolha antes de codar** essa
   parte.
6. **Simplicidade** (§0.4 + YAGNI/KISS): escolha o menor caminho que resolve o critério de aceite **por
   completo e na fonte** — simplicidade não é desculpa para workaround ou solução conceitualmente
   errada, é a ausência de complexidade acidental/especulativa além do que o problema exige.
7. **TDD quando aplicável** (MANDATORY 9 + §5.2): se o passo 4 classificou a lógica como isolável,
   **🔴 Red** — escreva agora o teste que descreve o critério de aceite desta task e rode-o,
   **colando a saída que confirma a falha**. Se a task caiu na exceção do §5.2 (integração externa,
   UI, scaffolding), registre por que e siga direto ao passo 8 sem pular esta decisão em silêncio.
8. **Implementar cirurgicamente** (§0.5) aplicando Clean Code + Object Calisthenics + DRY (MANDATORY
   8): **🟢 Green** — o mínimo de código para satisfazer o teste do passo 7 (ou, se não houve TDD
   nesta task, a implementação direta do critério); só o que a task pede; estilo existente; sem
   legado (MANDATORY 4); sem tipagem fraca (MANDATORY 1); código morto que notar → mencione, não
   apague. **Antes de fechar o arquivo, normalize os comentários dele por inteiro** (MANDATORY 11):
   apague o que não cabe nas seis categorias — em especial todo comentário-changelog — e, se a razão
   dele ainda não estiver no commit/ADR/sprint, **mova-a para lá** antes de apagar.
9. **Provar** (§0.6 + MANDATORY 9/10 + §5.3): rode a **suíte de testes** do projeto (regressão
   completa) e o lint/typecheck, **colando as duas saídas** — lint limpo não é evidência de critério
   satisfeito, teste passando é. Para a lógica isolável desta task, rode também **coverage** e
   **colone o relatório**: 100% de linha/branch do código novo, sem exceção não justificada. Se a task
   envolveu lógica de negócio crítica, rode **mutation testing** e colone o resultado — corrija
   qualquer mutante sobrevivente na fonte (teste que executa a linha mas não valida o comportamento),
   nunca suprima o mutante. Se a task cruzou camadas reais, confirme que o teste de integração roda
   sem mock no ponto validado; se é fluxo crítico/ponta a ponta, confirme o e2e.
   **🔵 Refactor** — limpe o código (Object Calisthenics, DRY, KISS) sem quebrar o verde e rode de
   novo. Corrija na fonte até passar tudo limpo. **Só então** marque `[x]` nas tasks e critérios de
   aceite desta história.
10. **Checkpoint:** pause e mostre o incremento ao usuário antes da próxima história (em `--lane`, atualize também o estado da raia — `<lane_mode>`). Em história de
    frontend, diga se o wireframe foi atualizado (e o que mudou) ou que a implementação bate com ele.

**Fechamento da implementação:**
11. Confirme que a Definition of Done da sprint está satisfeita; reporte o que mudou e as evidências —
    inclusive, no frontend, a lista de atualizações do wireframe (URL inalterada).

**Gates de qualidade (review e security):**
12. **Decida quais gates rodam.** Leia as **flags de risco** da sprint e o **diff real**
    (`git diff --name-only $BASE_SHA` + `git ls-files --others --exclude-standard`, menos o snapshot do
    passo 2). Regras do §1 da methodology:
    - **Review:** obrigatório se a sprint toca lógica de negócio, API pública, fluxo crítico ou frontend;
      opcional em scaffolding puro — **dispensar exige motivo escrito**.
    - **Security:** obrigatório se o **diff** toca entrada externa, autenticação/autorização, persistência
      ou query, dado sensível ou LLM/prompt. A flag da sprint pode **promover** (rodar security) mas
      **nunca dispensar** uma superfície que o diff de fato toca. Só roda review se nenhuma superfície for
      tocada — e diga isso.
    Anuncie a decisão e o porquê ao usuário (uma linha cada). Não pergunte, **decida** — a decisão é sua.
13. **Review — spawne o subagent** (`Agent`, `subagent_type: "general-purpose"`; model/effort do **tier**
    da sprint, coluna "Reviewer" do §6.2). Brief fechado: *"Siga
    `~/.claude/skills/codeform/playbooks/review.md`. Sprint: `<caminho do SPRINT_NN_*.md>`;
    `PROJECT_CONTEXT`: `<caminho>`. Arquivos da sprint: `<lista>`. Wireframe (se frontend): `<URL e
    .html>`. Read-only. Devolva o relatório no formato do playbook."* Não passe suas conclusões nem o
    que você acha que está certo.
14. **Triagem do review.** `PARADA` → ao usuário, resposta por `SendMessage`. Para cada achado
    **bloqueador**: abra o `caminho:linha` e confirme; descarte o que não procede, **dizendo o que
    descartou**. Classifique **simples × sensível** pelos critérios do §7 (a proposta do reviewer é
    sugestão, a classificação é sua).
    - **Simples:** corrija agora, cirurgicamente, mostre o diff, rode lint/typecheck **e** a suíte
      relevante **colando as saídas** (§0.6).
    - **Sensível:** apresente ao usuário (§0.2): o problema, as opções com prós/contras, a recomendação e
      o porquê, e **espere a escolha antes de codar**. Em `--lane`, marque `aguardando-decisao`.
15. **Re-review.** Havendo correções, **retome o mesmo reviewer** via `SendMessage` restrito aos arquivos
    alterados. **No máximo 2 rodadas** de review no total; persistindo bloqueador, escale ao usuário.
16. **Security** (se o passo 12 mandou) — **só depois** do review fechado (revisar antes evita auditar
    código que ainda vai mudar). Spawne um subagent novo (model/effort do **tier**, coluna "Security" —
    nunca rebaixado). Brief: *"Siga `~/.claude/skills/codeform/playbooks/security.md`. Sprint …
    Arquivos: … Read-only. Devolva o relatório D1 e a cobertura."*
17. **Triagem do security.** Verifique **todo** achado 🔴/🟠 abrindo fonte e sink. Corrija **no sink**, sem
    perguntar, só o que o §7 chama de simples (🔵/🟡 locais) — diff + testes colados. **Todo 🔴/🟠, e tudo
    o que toque auth, persistência, migração ou contrato**, vai ao usuário, **que decide corrigir, aceitar
    ou debater**. **Aceitar um risco é decisão dele**, de qualquer severidade; nunca sua.
18. **Re-verificação.** Para cada correção, refaça o caminho fonte → sink e mostre por que o ataque agora
    falha e que o comportamento legítimo não mudou. Houve correção? **Uma** nova passada do security
    (`SendMessage`) restrita aos arquivos alterados.
19. **Registro dos gates.** Preencha a seção "Registro dos gates" do `SPRINT_NN_*.md`: review (rodadas,
    veredicto) ou dispensa + motivo; security (veredicto, cobertura) ou "não se aplica" + motivo; achados e
    correções aplicadas; **riscos aceitos** (severidade + justificativa do usuário). É o que o
    `/codeform:release` confere.
19a. **Sincronizar sub-issues** (regras de sincronia do §2 da methodology; sem issue de origem, pule).
    - **Durante a sprint:** se alguma decisão aprovada alterou uma História ou a issue, atualize o corpo da
      pai e das sub-issues afetadas **antes** de seguir (§2).
    - **Ao fechar a sprint:** só se o Registro dos gates (passo 19) estiver preenchido com review ✅ (ou
      dispensa justificada), security ✅ ou "não se aplica", **sem bloqueador aberto** e DoD satisfeita.
      Para cada História **desta sprint** com todos os critérios `[x]` no `SPRINT_NN_*.md`: localize a
      sub-issue, marque todos os itens do checklist do corpo (`gh issue edit <n> -F <corpo com [x]>`) e feche
      como correta: `gh issue close <n> --reason completed -c "Implementada e revisada (review ✅, security
      <✅|n/a>) — ver <caminho do SPRINT_NN_*.md>"`. História com critério pendente ou risco aceito não
      resolvido: **não feche**, comente o que falta e avise o usuário. Nunca feche a issue pai (quem fecha é
      o PR). Em `--lane`, faça no fim da raia, antes de `concluida`.
    - **Issue pai:** (em `--lane`, **não** edite a pai — o `/codeform:orchestrate --merge` faz isso uma vez; atualize só as suas sub-issues.) Releia o corpo (`gh issue view <n> --json body`), marque `[x]` o item desta sprint no
      "Plano e andamento" e os **critérios de aceite** da pai cobertos pelas Histórias fechadas (sem apagar
      o que outra pessoa escreveu), e aplique com `gh issue edit`. Critério da pai que depende de sprint
      futura continua `[ ]`.
    - **Casamento:** sub-issue casa pelo título sem o prefixo `[..]` ou pelo caminho do `SPRINT_NN_*.md`
      no corpo (methodology §2). Sub-issue sem casamento: reporte, não pule.
    - Falhou (permissão, sub-issue não encontrada)? Reporte qual e siga; não declare sincronizado sem reler
      o estado (`gh issue view <n> --json state,body`) e conferir que não restou `- [ ]` nas Histórias
      fechadas.

**Fechamento:**
20. **Relatório de comentários** (MANDATORY 11): liste (a) os arquivos cujos comentários você normalizou, com
    a densidade antes → depois; e (b) os arquivos que você **leu mas não tocou** e estão acima de 10% de
    densidade — só `caminho` + `%`, sem editar. O item (b) é insumo para o usuário decidir se vira task;
    **não** o transforme em faxina por conta própria.
21. **Relatório final** (§0.8): resultado; histórias e critérios `[x]`; sub-issues fechadas/atualizadas; **agentes, modelos e effort
    usados** (você, reviewer, security, com as rodadas); evidências; decisões submetidas ao usuário;
    riscos aceitos; bloqueios e próximo passo. Em `--lane`: estado `concluida` e SHAs no arquivo da raia.
</process>

<guardrails>
- Surgical changes: toque só o necessário; case o estilo do projeto; remova só órfãos que você criou.
- Uma sprint por vez, na ordem do overview. Não comece a próxima sem a atual validada.
- **Suba a confusão, não a esconda** (§0.3): se a sprint, o código existente ou a documentação entrarem em
  contradição — ou se tiver qualquer dúvida sobre qual é o caminho mais enterprise e correto — **PARE antes
  de escrever a linha de código daquele trecho** e traga para debate.
- Clean Code + Object Calisthenics valem desde a primeira linha escrita da história, não só no review.
- **Os revisores não são você.** Não passe a eles suas conclusões, não os use para "validar" o que você
  já decidiu, e **nunca declare "review aprovado" sem o relatório do subagent e a sua verificação dele**.
- **Sensível nunca é corrigido sozinho.** Na dúvida entre simples e sensível, é sensível (§7).
- **Sem `git push`.** Commits só em `--lane` (branch efêmera) ou se o usuário pedir.
</guardrails>

<gate>
Sprint implementada, DoD satisfeita, gates de qualidade fechados, **Registro dos gates** preenchido e
**issues sincronizadas** (passo 19a, relidas com `gh issue view` — sem `- [ ]` nas Histórias fechadas nem na
pai para o que a sprint cobre; sem issue de origem, "não se aplica"):

- Há próxima sprint no overview: "Sprint concluída. Rode **`/codeform:execute <próxima>`**" — ou
  **`/codeform:orchestrate`** se o plano (§11.1) indicar bloco paralelo.
- Era a última sprint: "Última sprint concluída. Rode **`/codeform:release`** — valida a branch inteira e abre
  o PR."

Se houver achado sensível ou CRÍTICO em aberto: "Sprint bloqueada. Decida os itens apresentados acima antes
de avançar."
</gate>
