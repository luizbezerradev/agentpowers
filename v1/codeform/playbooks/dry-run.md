# Playbook — Dry-run pré-execução

> Lido pelo subagent **novo, de contexto limpo** que o `/codeform:plan` spawna depois que o usuário aprova os
> artefatos. O contexto é limpo de propósito: quem escreveu as sprints tem viés para marcar ✅.
> Este playbook é **read-only**: o subagent produz o relatório (Blocos A–C e D1); quem aplica as
> atualizações aprovadas (D2–D4) é o pai.

<execution_context>
Antes de prosseguir, leia integralmente `../methodology.md` — em especial o §0 (Contrato de Pair
Programming), o §2 (Caminhos canônicos) e o §6.1 (plano de paralelização).
</execution_context>

<source_of_truth>
⚠️ **LEIA ANTES DE TUDO — este bloco se sobrepõe a qualquer outra instrução deste arquivo.**

Não é porque está no `CLAUDE.md`, nem porque está escrito na sprint ou no `PROJECT_CONTEXT.md`, nem
sequer porque já está implementado no código, que aquilo está correto e deve ser tratado como **fonte
da verdade**. O que é correto é o que é **conceitual e arquiteturalmente correto** — e é isso que deve
ser levado em consideração ao validar cada task.

Não fique tentando modelar a nova implementação no que já existe só porque já existe e já está lá,
como se tudo tivesse que se modificar para funcionar do jeito que está. O que se quer é o **inverso**:
se o dry-run identificar que o que existe está conceitual ou arquiteturalmente incorreto, ou é um
workaround, e a sprint e o `CLAUDE.md` seguem esse erro, então **temos que debater e corrigir tudo** —
para que a nova implementação fique correta e, se necessário, corrigindo também o que já existe, tendo
em mente que utilizamos **Clean Architecture e SOLID**.

Sendo assim, ao percorrer cada task e cada decisão do `PROJECT_CONTEXT.md`:

> Qual o caminho mais enterprise e conceitualmente e arquiteturalmente correto, que resolve o problema
> **na fonte**, sem workarounds, sem seguirmos com dívida técnica e que é **prova de futuro**?

Isso vale **independente do esforço necessário para implementação** e independente do esforço que a
sprint já levou para ser escrita, desde que atenda a estes requisitos. Ao apontar um desvio, **seja
didático**: explique qual formato você recomenda e por quê. Lembre-se: a sprint é um guia, **não uma
verdade absoluta**.

**Não implemente e não reescreva a sprint sozinho — traga para debate (Bloco C → D1).**
</source_of_truth>

<protocolo_subagent>
🤖 **Você roda como subagent, spawnado pelo agente do comando (o "pai"). Você não tem canal com o usuário** (§0.7 da methodology).

- Onde este playbook diz **PARE**, **pergunte**, **aguarde a escolha** ou **debata**: não decida e não siga
  em frente. **Encerre o turno** devolvendo um bloco `PARADA` com a decisão, as opções (vantagens e
  desvantagens), a sua recomendação e o motivo. O pai leva ao usuário e **retoma você via `SendMessage`**
  com a resposta. O §0.2 e o §0.3 continuam valendo; só mudou quem fala com o usuário.
- **Leia na íntegra** todo arquivo em que basear uma afirmação e cite `caminho:linha` (§0.1). O pai abre
  cada linha citada em achado bloqueador/crítico antes de apresentá-lo; achado sem linha verificável é
  descartado.
- **Declare a cobertura:** termine dizendo o que leu e o que **não** leu. "Nada encontrado" sem a lista do
  que foi lido é incompleto.
- **Na dúvida entre ✅ e ⚠️, marque ⚠️.** Você foi spawnado limpo justamente para não herdar o otimismo de quem escreveu as sprints.
- **Formato de saída:** o relatório D1 — resumo executivo, tabela de gaps que exigem decisão, decisões pendentes do Bloco A3 agrupadas por sprint, e a validação do plano de paralelização (Bloco B3). Nada é escrito em disco.
</protocolo_subagent>

<objective>
Realizar uma **revisão geral completa de todas as sprints** da iniciativa antes de qualquer linha de
código ser escrita. Garantir que, no momento da execução, não seja preciso tomar nenhuma decisão nova,
isolada ou ambígua — tudo já deve estar decidido, nomeado e justificado na sprint.

Esta fase opera em dois modos simultâneos:

1. **Dry-run de implementação:** simule mentalmente cada sprint como se estivesse implementando de
   verdade — percorra cada história, cada critério de aceite, cada task do task breakdown. Se algo não
   estiver claro o suficiente para codar imediatamente, é uma lacuna.
2. **Gap analysis PROJECT_CONTEXT ↔ Sprints:** compare o que o `PROJECT_CONTEXT.md` prometeu com o
   que as sprints de fato cobrem. Tudo que foi planejado deve aparecer em alguma sprint; tudo que
   aparece nas sprints deve ter base no contexto.

Ao final, entregue um relatório de gaps para decisão. **Não implemente e não reescreva nada — traga tudo para debate.**
</objective>

<guiding_question>
Ao percorrer cada task no Bloco A, não valide só "está claro o suficiente para codar" — valide se o
caminho descrito é o **certo**. Para cada task, pergunte-se:

> A task, do jeito que está escrita, descreve o caminho do `<source_of_truth>`?

Reescrever uma task ou reestruturar uma sprint inteira agora é sempre mais barato que descobrir o
problema em execução.

Aplicado a esta fase: se uma task ou uma decisão de `PROJECT_CONTEXT.md` estiver clara o suficiente
para codar, mas o caminho **em si** for
conceitualmente ou arquiteturalmente errado, isso também é um achado do dry-run — não é só sobre
detectar ambiguidade, é sobre validar se o que está decidido é a decisão certa. Explique
didaticamente por que o caminho atual está errado, qual formato você recomenda e por quê, registre
como recomendação (Bloco C) e **traga para debate** — não reescreva a sprint sozinho antes do D2.
</guiding_question>

<process>
Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo.

### Setup

1. **Resolver a iniciativa** a partir do brief. Localize `docs/Sprints/<Iniciativa>/`.
2. **Carregar tudo:**
   - `PROJECT_CONTEXT.md` — leia na íntegra.
   - `SPRINTS_OVERVIEW.md` — ordem oficial, descrição de cada sprint.
   - Todos os `SPRINT_NN_*.md` — leia cada um na íntegra, na ordem do overview.
   - **Wireframe** (se a iniciativa toca frontend): a seção "Wireframe" do `PROJECT_CONTEXT.md` tem a
     URL do artifact e o `.html`? Leia o `.html` na íntegra.
3. **context7 (obrigatório):** para cada lib/framework/SDK citado nas sprints, rode
   `resolve-library-id` → `query-docs`, para validar que nomes de API, formatos e integrações batem
   com a realidade da biblioteca — não com a memória.

---

### Bloco A — Dry-run por sprint

Para **cada sprint**, na ordem do overview, percorra:

#### A1. Simulação de implementação
Percorra cada `### 🎫 História` e cada task do Task Breakdown como se fosse executar agora:
- O caminho do arquivo existe no projeto / faz sentido dentro da estrutura esperada?
- O que está escrito em "detalhe técnico do que muda" é específico o suficiente para codar sem
  perguntas?
- As dependências de outras sprints (ordem, contratos, exports) estão explicitadas na seção
  "Dependências"?
- Se uma task diz "criar X", está claro: nome exato, local exato, assinatura, tipos, comportamento?
- Se uma task diz "modificar Y", está claro: qual trecho, qual mudança, qual o antes e o depois?
- Se uma task diz "remover Z", o legado está completamente mapeado (imports, re-exports, usages)?

Marque cada ponto como: ✅ claro | ⚠️ ambíguo (precisa de complemento) | ❌ lacuna (blocking)

#### A2. Checklist MANDATORY por sprint
- [ ] Nenhuma tipagem fraca implícita nas tasks (tipos dos novos contratos explícitos)?
- [ ] Legado de **código** mapeado para remoção, se a sprint substitui algo existente?
- [ ] Se a sprint muda schema/formato/local de um dado **já em produção**, a **migração** desse dado
  está mapeada como task (não só a remoção do código antigo)?
- [ ] Nomes de arquivos, pastas, funções, variáveis e tipos seguem a convenção do projeto?
- [ ] Cada task aponta o caminho **relativo à raiz do projeto** (não path parcial ou ambíguo)?
- [ ] Lint/typecheck do projeto passará (nada na sprint exige um padrão que viola as regras)?
- [ ] YAGNI/KISS: a sprint pede só o que os critérios de aceite exigem? Nenhuma abstração especulativa?
- [ ] Critérios de aceite são observáveis e verificáveis (não "deve validar" mas "rejeita X quando Y")?

#### A2.1. Frontend × wireframe (se a iniciativa toca frontend)
- Toda task de frontend cita a região/estado do wireframe que implementa?
- Todo estado do wireframe (vazio, erro, carregando, modal…) tem task que o cobre?
- Todo componente `NOVO` do wireframe virou task própria?
- Alguma task contradiz o wireframe? Nunca resolva em silêncio: ✏️ atualizar sprint ou ❓ decisão do
  usuário (e, se o desenho é que muda, o wireframe é atualizado pelo modo de atualização do playbook de wireframe).

#### A3. Decisões pendentes
Liste toda decisão que seria necessária na hora da execução por falta de instrução na sprint. Para
cada uma: descreva a decisão, apresente opções com prós/contras, e **PARE** (§0.2) — não decida
sozinho, não omita, não prossiga sem resposta. Agrupe no relatório final (D1).

---

### Bloco B — Gap analysis PROJECT_CONTEXT ↔ Sprints

#### B1. Cobertura do plano
- Pegue cada fase/etapa da seção 11 do `PROJECT_CONTEXT.md`.
- Mapeie: qual `SPRINT_NN_*.md` cobre essa fase? (cite a história e a task)
- O que ficou sem sprint correspondente? → **gap de cobertura** (❌)
- O que aparece em alguma sprint mas não tem base no `PROJECT_CONTEXT`? → **scope creep** (⚠️)

#### B2. Consistência entre sprints
- Nomes de campos/entidades compartilhados entre histórias de sprints diferentes aparecem
  consistentes?
- Decisões de arquitetura do `PROJECT_CONTEXT` (seção 3 — princípios obrigatórios de engenharia — e
  seção 13 — riscos e decisões de implementação) estão refletidas nas tasks das sprints?

#### B3. Ordem de dependências e plano de paralelização
- A ordem do `SPRINTS_OVERVIEW.md` respeita dependências reais entre sprints?
- Há sprints que poderiam ser paralelas mas estão em série?
- **Há histórias na mesma sprint que poderiam virar sprints paralelas?** Para cada par de histórias de uma
  mesma sprint, calcule a interseção dos paths das tasks. Se for vazia e a única dependência for um contrato
  fixável antes (interface, tipo de resultado, DTO, assinatura de use case), reporte ⚠️ "separável" citando o par
  e o contrato. Se o §11.1 registra o motivo de mantê-las juntas (arquivo indivisível ou dependência de
  comportamento), confira o motivo contra as tasks: motivo que as tasks não sustentam é ❌.
- **Valide o `PROJECT_CONTEXT.md` §11.1 contra as tasks reais** (§6.1 da methodology): para cada bloco
  declarado paralelo, extraia os paths de todas as tasks de cada sprint e **calcule a interseção**.
  Interseção não vazia, ou arquivo de contenção sem dono único, é ❌: o bloco não é paralelo.
- O §11.1 está no **formato de diagrama em traços (ASCII)**, com a lista "Paths que as tasks tocam" abaixo? Tabela
  ou Mermaid no lugar do diagrama é ⚠️ de formato (o usuário pediu traços).
- O **veredito** (`serial`/`parcial`/`total`), os **blocos**, a **ordem de merge** e o **tier** de cada
  sprint estão presentes e coerentes com as flags de risco da sprint?

---

### Bloco C — Recomendações

Para cada ponto dos blocos A e B, formato por recomendação:
```
**[SPRINT_NN | PROJECT_CONTEXT]** | **Tipo:** Ambiguidade / Gap / Legado / Nomenclatura / Decisão pendente / Scope creep
**Problema:** <descrição objetiva com referência ao trecho>
**Recomendação:** <o que fazer — resolve na fonte, sem legado>
**Fundamentação:** <por que isso importa>
**Ação:** ✏️ Atualizar sprint / ✏️ Atualizar PROJECT_CONTEXT / ❓ Decisão do usuário
```

---

### Bloco D — Relatório (D1) — o resto é do pai

**D1. Entregue ao pai:**
- Resumo executivo: N sprints revisadas, X OK, Y com ajustes sugeridos, Z com gaps para decisão.
- Tabela de gaps que requerem decisão.
- Lista de decisões pendentes do Bloco A3 (agrupadas por sprint).
- Resultado da validação do plano de paralelização (B3).

**D2–D4 não são suas.** O pai apresenta o relatório ao usuário, espera a aprovação e aplica as
atualizações em `SPRINT_NN_*.md`, `PROJECT_CONTEXT.md` e `SPRINTS_OVERVIEW.md` (nunca apagando decisões
registradas — reescrevendo ou complementando), e confirma a releitura. Se o pai retomar você para
re-validar as seções alteradas, releia só elas.

---

</process>

<guardrails>
- **Read-only.** Você não altera nenhum arquivo.
- **Citar sempre o artefato + seção + trecho** ao apontar qualquer problema.
- **Decisão é do usuário** (§0.2): em toda bifurcação, **PARE**, apresente opções e **espere a
  escolha** — gaps vão para o relatório (D1), nunca resolvidos unilateralmente.
- **Não simule código** — o dry-run valida *a sprint*, não escreve implementação.
- **Não expanda escopo:** recomendações resolvem lacunas reais, não adicionam funcionalidade não
  planejada.
- **Suba a confusão, não a esconda** (§0.3): qualquer ambiguidade encontrada em qualquer bloco (A, B
  ou C) vai para o relatório D1 como item pendente — nunca é resolvida silenciosamente marcando ✅ por
  suposição. Na dúvida entre ✅ e ⚠️, marque ⚠️.
</guardrails>

<gate>
Devolva ao pai o veredito: ✅ **Pronto para execução** ou ⚠️ **Itens pendentes de decisão do usuário**,
sempre acompanhado da cobertura (o que foi lido).
</gate>
