# Playbook — Wireframe de alta fidelidade

> Lido pelo subagent (`fork`) que o `/codeform:brainstorm` spawna depois do estudo aprovado. Também é o
> **modo de atualização** usado pelo `/codeform:execute` quando o frontend muda durante a implementação.

<execution_context>
Antes de prosseguir, leia integralmente:
- `../methodology.md` — em especial o §0 (Contrato de Pair Programming).
- O `CLAUDE.md` do app de frontend (e o da raiz) e os docs de design system que ele indica (ex.: tipografia,
  primitivas de texto/ícone/campo, inputs, controls, tabs, páginas de listagem). **Descubra onde estão** pelo
  `CLAUDE.md` do projeto ou procurando o app de frontend e uma pasta de design system (`docs/design-system/`,
  Storybook, tokens); sem design system documentado, o código real é a única fonte — diga isso. **Releia a cada execução**: eles mudam sem que este playbook seja atualizado.
- O estudo aprovado (a conversa, pois você é um `fork`). O
  `PROJECT_CONTEXT.md` ainda não existe na primeira rodada; se existir (re-rodada ou modo de
  atualização), leia-o.
- Se `docs/Sprints/<Iniciativa>/` já tiver um `WIREFRAME_*.html`, leia-o: é o estado vigente a evoluir,
  não a refazer do zero.
</execution_context>

<source_of_truth>
⚠️ **LEIA ANTES DE TUDO — este bloco se sobrepõe a qualquer outra instrução deste arquivo.**

O wireframe existe para **avaliar uma decisão**, e uma avaliação só vale se o desenho for **fiel ao
produto**: o que o usuário vê aqui tem de ser o que o frontend vai renderizar. Duas fontes, em ordem:

1. **O código real do frontend** (os componentes de UI, o código compartilhado, os módulos, o CSS global e os
   arquivos de tokens que ele importa, no app de frontend do projeto) — o que de fato existe.
2. **O design system**, se o projeto tiver um — o que *deveria* existir.

Se as duas divergirem (tela legada fora do padrão, componente que viola as regras cardinais), **não
escolha em silêncio**: mostre o desvio e traga para debate (§0.2). Desenhe pelo design system quando a
tela for nova; pelo componente existente apenas quando o objetivo for justamente evoluí-lo.

> Qual o caminho mais enterprise e conceitualmente correto para esta tela, usando os blocos que o
> frontend **já tem** — e, quando falta um bloco, dizendo explicitamente que ele **não existe**?

**Não invente componente.** Se o desenho precisa de algo que o frontend não tem, marque no wireframe
como `NOVO` (visualmente distinto), liste-o no relatório e deixe a decisão ao usuário. Nunca o desenhe
como se já existisse — isso cristaliza uma promessa falsa no `/codeform:plan`.

**Não implemente nada no frontend — traga para debate.**
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
- **Você não implementa nada no frontend.** Os únicos arquivos escritos são o `WIREFRAME_*.html` e o artifact.
- **Formato de saída:** duas fases.
  - **Fase 1 (passos 1–4):** devolva o inventário de fidelidade e **todas** as decisões pendentes (`🆕 NOVO`, desvios entre código e design system, recorte de estados) num bloco `PARADA` — mesmo que não haja `NOVO`, o pai precisa aprovar o recorte antes de você desenhar.
  - **Fase 2 (passos 5–9), depois que o pai retomar você com as decisões:** devolva a URL do artifact, o caminho do `.html`, o inventário final e o que não foi reproduzido fiel.
  Ajustes pedidos pelo usuário chegam por `SendMessage` e voltam pelos passos 2–8 só para o que mudou.
</protocolo_subagent>

<objective>
Criar um **Canvas Design Artboard Artifact** — wireframe de **alta fidelidade** — da tela/fluxo indicado no brief,
construído com os componentes reais do frontend da codebase, para que o usuário avalie **como a feature
ficaria** antes de as sprints serem escritas.

"Componentes reais" num artifact HTML significa: **reproduzir exatamente** o que o componente real
renderiza — mesma estrutura de DOM, mesmas classes utilitárias, mesmos tokens (`--control-*`,
`bg-accent`, `bg-active`, `border-border`, `text-content-*`…), mesmas variantes e tamanhos, mesmos
ícones (o `lucide-react` que o componente usa) — e **nunca** uma aproximação "no estilo de". O wireframe
é descartável; a **fidelidade** não é.

Esta fase **não cria código de produto**. O entregável é o artifact (e seu `.html` fonte).
</objective>

<guiding_question>
Para cada elemento desenhado, pergunte-se:

> Eu abri o componente que renderiza isto e copiei o que ele realmente emite — ou desenhei de cabeça?

E, no nível da tela:

> Este desenho serve ao objetivo da implementação, ou é só uma tela bonita? Cada região responde a um
> requisito do objetivo; região sem requisito é scope creep visual — corte.

Estamos em **dev, mas já existe produção**: o wireframe mostra o estado **alvo**, sem preservar tela
legada "por compatibilidade". Mas se a mudança altera uma tela com **dados já em produção**, desenhe
também como esses dados existentes aparecem (estado migrado), não só o caso vazio.
</guiding_question>

<process>
Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo.

1. **Travar o que desenhar.** Se o brief estiver vazio ou vago (não dá para saber qual tela, qual
   fluxo, qual persona), **PARE e pergunte**. Declare suposições; confirme as arriscadas. Liste as
   **telas/estados** que vai desenhar (ex.: vazio, preenchido, carregando, erro, permissão negada,
   modal de confirmação) e confirme com o usuário se o recorte tiver mais de uma forma razoável.
2. **Localizar o que já existe.** Ache a(s) tela(s) vizinha(s) e os componentes candidatos Abra e leia **na íntegra** cada um. Registre
   `caminho:linha` de cada componente que vai usar.
3. **Ler o design system** relevante (ver `<execution_context>`) e a doc oficial via **context7** para
   Radix/shadcn/lib de UI tocada — variantes, estados e acessibilidade vêm da doc, não da memória.
4. **Montar o inventário de fidelidade** — antes de escrever HTML, apresente no chat uma tabela:

   | Região do wireframe | Componente real | `caminho:linha` | Variante/tamanho | Status |
   |---|---|---|---|---|
   | Cabeçalho da página | `<ComponenteExistente>` | `src/…:12` | — | ✅ existe |
   | Seletor de nível | — | — | — | 🆕 NOVO (não existe) |

   Todo `🆕 NOVO` e todo desvio entre código e design system **para aqui** (§0.2): apresente as opções
   com vantagens e desvantagens, recomende uma e **espere a escolha** antes de desenhar a região.
5. **Carregar a skill `artifact-design`** (via `Skill`) e seguir o contrato de página dela (título,
   tokens em `:root`, tema claro/escuro, layout em largura de celular). Os **tokens do wireframe vêm
   dos tokens reais do frontend** (valores lidos de `globals.css`/arquivos de tokens), mapeados para
   as variáveis do `:root` do artifact — não valores inventados. O wireframe deve funcionar nos dois
   temas se o frontend os tem.
6. **Escrever o HTML** na pasta temp (scratchpad) da sessão, como `WIREFRAME_<TEMA_EM_MAIUSCULAS>.html`
   — é dali que o artifact é publicado (passo 7). Requisitos:
   - **Um bloco por estado** da lista do passo 1, navegáveis (abas/âncoras) — avaliar só o caminho feliz
     esconde justamente o que falta decidir.
   - **Interatividade mínima** que ajude a avaliar (abrir modal, alternar aba, expandir seção), em JS
     inline simples — sem framework, sem dependência que o componente real não use.
   - **Anotações discretas** (numeradas) ligando cada região ao requisito do objetivo que ela atende,
     com legenda que mostra o componente real e o `caminho` — é o elo que o `/codeform:plan` aproveita.
   - Regiões `🆕 NOVO` com marca visual própria (borda tracejada + selo) e legenda do que falta.
   - Texto real do produto (pt-BR, com acentos), dados de exemplo **plausíveis do domínio** — nunca
     "Lorem ipsum" nem `Item 1/Item 2`.
   - Acessibilidade básica preservada (papéis, foco visível, contraste), porque o componente real a
     tem e o desenho deve refleti-la.
7. **Publicar o artifact** com a ferramenta `Artifact` apontando para o HTML do scratchpad (privado por
   padrão), com `icon` e `description` de uma frase. **Em seguida, crie a pasta da iniciativa**
   `docs/Sprints/<Iniciativa>/` se ainda não existir (**declare o nome da iniciativa** — é o mesmo que o
   `/codeform:plan` usará) e **copie para lá o mesmo HTML** como `WIREFRAME_<TEMA_EM_MAIUSCULAS>.html`
   (convenção das outras iniciativas): é a cópia versionada que permite editar o wireframe no futuro.
   Em ajustes, **edite a cópia de `docs/Sprints/` e republique-a passando a `url` do artifact** (mesma
   URL) — nunca crie um novo artifact a cada rodada. **A URL do artifact é entregável**: declare-a no
   relatório, junto do caminho do `.html` — o `/codeform:plan` a grava na seção "Wireframe" do
   `PROJECT_CONTEXT.md`, e o `/codeform:execute` (com o reviewer dele) partem dela. Se o
   `PROJECT_CONTEXT.md` já existir, confirme que a seção "Wireframe" tem essa URL.
8. **Conferir antes de entregar:** releia o HTML produzido contra o inventário do passo 4 — cada região
   tem o componente, as classes e os tokens declarados? Se algo foi desenhado "de cabeça", corrija
   agora ou marque honestamente na legenda. Diga no relatório o que **não** conseguiu reproduzir fiel
   (ex.: componente que depende de dado ao vivo, editor rico, canvas) e por quê.
9. **Relatório no chat** (§0.8): (1) link do artifact e caminho do `.html`; (2) o que o desenho mostra,
   por estado; (3) inventário final de componentes (reais × `NOVO`); (4) desvios entre código e design
   system encontrados; (5) decisões que o usuário precisa tomar; (6) próximos passos. O pai conduz a avaliação.
10. **Iterar:** o pai retoma você com os ajustes do usuário; cada um volta pelos passos 2–8 apenas para o que mudou.
</process>

<update_mode>
Este playbook também é o **mecanismo de atualização** do wireframe durante o `/codeform:execute`: quando o
frontend implementado diverge do desenho (ajuste de componente, estado novo, decisão tomada na
implementação), o agente do execute segue este modo — no próprio fio ou via subagent. Nesse modo, **não
recomece**: leia a URL do artifact no `PROJECT_CONTEXT.md` (seção "Wireframe"), edite o `WIREFRAME_*.html`
existente só no que mudou, releia os componentes reais alterados (passos 2 e 8), republique **na mesma
URL** e diga no relatório o que mudou e por quê. A URL nunca muda.
</update_mode>

<guardrails>
- **Sem código de produto.** Só leitura do frontend; os únicos arquivos escritos são o
  `WIREFRAME_*.html` (e o artifact publicado).
- **Fidelidade acima de beleza.** Um componente que parece certo mas não bate com o DOM/classes reais é
  falha, mesmo que o resultado agrade o olho.
- **Nunca desenhe como existente o que não existe** (`🆕 NOVO` sempre explícito).
- **Contrato de Pair Programming** (§0 da methodology): cite `caminho:linha`; suba confusões antes de
  seguir; a decisão de design é do usuário (§0.2).
- **Sem dado real.** Não coloque PII, nome de cliente ou dado de produção nos exemplos do wireframe.
- **Não é a spec.** O wireframe é insumo de decisão; os critérios de aceite continuam nascendo no
  `/codeform:plan`. Divergência futura entre wireframe e sprint se resolve pelo usuário, não em silêncio.
- **Suba a confusão, não a esconda** (§0.3): qualquer ambiguidade entre objetivo, código e design
  system — em qualquer passo — é motivo para parar e perguntar **antes** de desenhar a região.
</guardrails>

<gate>
Entregável ao pai (Fase 2): URL do artifact, caminho `docs/Sprints/<Iniciativa>/WIREFRAME_<TEMA>.html`,
inventário final (reais × `NOVO`), desvios e o que não foi reproduzido fiel. **Quem conduz a aprovação do
desenho com o usuário é o pai**; o `PROJECT_CONTEXT.md` deve trazer o link do artifact e o caminho do
`.html`, e as sprints devem listar como tasks cada componente marcado 🆕 NOVO.
</gate>
