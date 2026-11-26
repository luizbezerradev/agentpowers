# Playbook — Code review pós-execução

> Lido pelo subagent **novo, de contexto limpo** que o `/codeform:execute` spawna ao concluir uma sprint.
> É **read-only**: o subagent aponta e classifica; quem corrige o que é simples e escala o que é
> sensível é o pai (§7 da methodology).

<execution_context>
Antes de prosseguir, leia integralmente `../methodology.md` — em especial o §4 (Clean Code, 9 Object
Calisthenics, YAGNI/KISS/DRY/Less Code Best Code), o §5 (SDD/TDD) e o §7 (achados simples × sensíveis).
</execution_context>

<source_of_truth>
⚠️ **LEIA ANTES DE TUDO — este bloco se sobrepõe a qualquer outra instrução deste arquivo, inclusive
ao checklist de fidelidade à sprint.**

Não é porque está no `CLAUDE.md`, nem porque está escrito na sprint, nem sequer porque já está
implementado no código, que aquilo está correto e deve ser tratado como **fonte da verdade**. O que é
correto é o que é **conceitual e arquiteturalmente correto** — e é isso que deve ser levado em
consideração ao revisar cada trecho.

Não valide a nova implementação só porque ela se encaixou no que já existe. O que se quer é o
**inverso**: se o review identificar que o que existe está conceitual ou arquiteturalmente incorreto,
ou é um workaround, e o código recém-escrito, a sprint e o `CLAUDE.md` seguem esse erro, então
**temos que debater e corrigir tudo** — para que a nova implementação fique correta e, se necessário,
corrigindo também o que já existe, tendo em mente que utilizamos **Clean Architecture e SOLID**.

Sendo assim, para cada trecho revisado:

> Qual o caminho mais enterprise e conceitualmente e arquiteturalmente correto, que resolve o problema
> **na fonte**, sem workarounds, sem seguirmos com dívida técnica e que é **prova de futuro**? O que
> foi implementado é esse caminho?

Isso vale **independente do esforço que a correção exigiria**, desde que atenda a estes requisitos. Ao
apontar um desvio, **seja didático**: explique qual formato você recomenda e por quê. Lembre-se: a
sprint é um guia, **não uma verdade absoluta**.

**Não aplique a correção sozinho — traga para debate.**
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
- **Você não corrige nada.** Nem o que parece óbvio: a correção dos achados simples é do pai, a dos sensíveis é do usuário.
- **Formato de saída:** relatório com (1) resumo por arquivo (✅/⚠️/❌); (2) a tabela do frontend contra o wireframe, se aplicável; (3) lista de achados — cada um com `id`, `caminho:linha`, violação, correção sugerida, **classe** (`bloqueador` | `melhoria`) e **tratamento proposto** (`simples` | `sensível`, pelos critérios do §7 da methodology); (4) veredicto (**Sprint aprovada** | **Sprint bloqueada**); (5) cobertura.
</protocolo_subagent>

<objective>
Realizar o **code review da sprint indicada no brief** após sua execução, verificando se o
código implementado (1) cobre fielmente as histórias e critérios de aceite do `SPRINT_NN_*.md` e (2)
respeita os princípios de qualidade da metodologia (§4 e §5 da methodology). **Este playbook é
read-only** — aponta e classifica; não corrige.
</objective>

<guiding_question>
Este review não é só "o código bate com a sprint linha por linha" — é validar se o que foi
implementado é, de fato, o caminho do `<source_of_truth>`.

Consequência direta para a classificação dos achados: um código que "funciona" mas resolve com
workaround, dívida técnica ou fora do lugar conceitualmente correto é **bloqueador**, mesmo que
tecnicamente satisfaça o critério de aceite.

Aplicado a esta fase: se o código seguiu o `SPRINT_NN_*.md` ao
pé da letra mas o *desenho pedido pela sprint* é que está conceitualmente errado, **isso também é um
achado do review** — não valide "porque a sprint mandou". Registre o achado, explique didaticamente
por que o caminho da sprint está errado e qual você recomenda, e **traga para debate** (§0.2): não proponha como "simples" (§7) o que muda o desenho — mesmo que pareça óbvio.
</guiding_question>

<when_to_apply>
**Obrigatório** quando a sprint implementa:
- Lógica de negócio (cálculos, regras de domínio, fluxos de estado)
- API pública (endpoints, contratos, DTOs)
- Fluxo crítico (autenticação, pagamento, autorização, dados sensíveis)
- **Qualquer tela ou componente do frontend** — o review inclui o item 7 do checklist (frontend)

**Opcional (mas recomendado)** em scaffolding/infra sem lógica.
</when_to_apply>

<checklist>
Avalie cada item abaixo para **cada arquivo modificado pela sprint**. Para cada violação, cite
`caminho:linha`, descreva o problema e sugira a correção — sem aplicar sem aprovação.

### 1. Fidelidade às histórias

- Todo **Critério de Aceite** de cada história está implementado e verificável no código?
- Toda task do **Task Breakdown** foi de fato executada como descrito?
- Existe algo no código que a sprint **não** pede (scope creep)?
- A **Definition of Done** da sprint está satisfeita por completo?

### 2. Clean Code

- Nomes (variáveis, funções, classes) revelam intenção sem precisar de comentário explicativo?
- Funções pequenas, com um só nível de abstração e sem efeitos colaterais escondidos?
- Nenhum código morto, `console.log`/print de debug ou trecho comentado esquecido?

#### 2.1 Comentários (§4.3 da methodology) — verificação obrigatória

O padrão do repositório é **ausência de comentário**; cada um que sobrevive precisa se justificar.
Este é hoje o desvio mais frequente, então revise-o explicitamente, arquivo por arquivo.

Para **cada comentário** do diff, decida em qual das seis categorias permitidas ele cai — regra de
negócio implícita · decisão não-óbvia com link para ADR · aviso de consequência não-local · contrato
público (TSDoc/GoDoc, descrevendo o contrato) · fonte externa/justificativa de `eslint-disable` ·
`TODO(ID)` rastreável. **Não caiu em nenhuma? É achado.** Classifique pelo anti-padrão:

| Achado | Como aparece | Correção a propor |
|---|---|---|
| **Comentário-changelog** | *"havia X aqui"*, *"saiu na LDD-452"*, *"antes era Y"*, *"morreu na sprint Z"*, *"deixou de existir"* | Apagar do código. Se a razão ainda não está no corpo do commit / ADR / `SPRINT_NN_*.md`, **mover para lá** antes de apagar — a narrativa não se perde, muda de arquivo. |
| **Paráfrase do código** | `// incrementa o contador` | Apagar. |
| **Banner decorativo** | `═══ SEÇÃO ═══`, `// ---- helpers ----` | Apagar; se o arquivo precisa de índice visual, ele está grande demais (§4.1 nº 7). |
| **Código comentado** | trecho inteiro comentado | Apagar — o `git` guarda. |
| **Comentário que salva nome ruim** | `// aqui "d" é o documento` | Renomear o símbolo (§4.1 nº 6). |
| **Comentário desatualizado** | descreve comportamento que o diff mudou | Atualizar ou apagar — mentira com autoridade é pior que ausência. |

Verifique também o **orçamento** de cada arquivo tocado pela sprint:

```
grep -cE '^\s*(//|\*|/\*)' <arquivo>   # linhas de comentário
wc -l < <arquivo>                       # linhas totais
```

- **Bloco > 5 linhas** → achado; corte para uma frase + link para o ADR/sprint.
- **Densidade > 10%** → cobre justificativa **bloco a bloco**. Arquivo que é só declaração de
  contrato público pode legitimamente passar de 10%; arquivo de implementação, não.
- **Arquivo tocado pela sprint com comentário fora do padrão que sobreviveu** → achado: a
  MANDATORY 11 do `/codeform:execute` manda normalizar o **arquivo inteiro**, não só as linhas do diff.
- **Arquivo que a sprint não tocou** → **fora de escopo deste review.** Não é achado; se você notar
  densidade alta, mencione como observação separada, sem propor edição.

### 3. 9 Object Calisthenics (§4.1 da methodology)

| # | Princípio | O que verificar |
|---|---|---|
| 1 | Um nível de indentação por método | `if/for/while` aninhados sem extração? |
| 2 | Sem `else` | `else` onde um early return resolveria? |
| 3 | Sem primitivos expostos | `string`/`int` soltos onde há semântica de domínio? |
| 4 | Coleções encapsuladas | `List`/`Map`/`Set` manipulados fora de classe dedicada? |
| 5 | Uma chamada de ponto por linha | Encadeamentos `a.getB().getC()`? |
| 6 | Sem abreviações | Nomes abreviados onde o nome completo cabe? |
| 7 | Entidades pequenas | Classe > 50 linhas ou método > 10 linhas? |
| 8 | ≤ 2 variáveis de instância | Classe nova com 3+ variáveis de estado? |
| 9 | Sem getters/setters | Getter/setter onde "Tell, Don't Ask" resolve melhor? |

### 4. Princípios Clássicos (§4.2 da methodology)

- **YAGNI:** existe código que nenhuma história da sprint pediu?
- **KISS:** existe uma versão mais simples que satisfaz os critérios de aceite?
- **DRY:** lógica duplicada (≥ 3 ocorrências) que deveria ser extraída?
- **Less Code Best Code:** existe código que poderia ser removido sem perder nenhum critério de
  aceite (helper não usado, branch morta, parâmetro sem uso)?

### 5. SDD/TDD e tipos de teste (§5 da methodology)

- Todo critério de aceite e task da sprint está `[x]` com evidência colada?
- Para lógica isolável: existem testes **unitários** cobrindo os critérios de aceite?
- **Coverage:** o código novo/tocado por lógica isolável tem **100%** de linha/branch cobertos por
  teste com asserção real — não apenas execução sem validação de comportamento?
- **Mutação:** para lógica de negócio crítica (cálculo, regra de domínio, auth, fluxo financeiro),
  rodou-se mutation testing? Há mutante sobrevivente sem correção do teste?
- **Integração:** onde a história atravessa 2+ camadas reais (serviço+repositório, API+banco), existe
  teste de integração sem mock no ponto que deveria validar o contrato real?
- **E2E:** fluxo crítico ou jornada ponta a ponta exposta pela história tem teste e2e que simula o
  caminho real do usuário?
- **Regressão:** a suíte completa do projeto passa após a mudança? Se a história é bugfix ou protege
  comportamento existente, há caso de regressão dedicado que falharia sem a correção?
- Todos os testes passam? (cole a saída do runner e do relatório de coverage/mutação quando
  aplicável)
- O escalonamento por criticidade (§5.3) foi respeitado — nenhum tipo aplicável foi pulado, e nenhum
  tipo foi aplicado onde não fazia sentido (ex.: mutation testing em scaffolding trivial)?

### 6. Regras MANDATORY remanescentes (§3 da methodology)

- Nenhuma tipagem fraca sem justificativa?
- Lint/typecheck do projeto passa limpo?
- Nenhum código legado mantido em paralelo?

### 7. Frontend — conferência na tela real (só se a sprint toca frontend)

Ler o código **não basta**: o review do frontend acontece **na UI rodando**, contra o wireframe.

- **Subir o app** e abrir as telas da sprint (skill `run`; para navegar/interagir, skill
  `maestri-portal`; para testes de browser, o harness que o app de frontend já tiver, ex.: Playwright). Se não for possível subir,
  **diga isso** e não aprove o frontend — nunca aprove tela que não viu.
- **Wireframe:** a URL do artifact está na seção "Wireframe" do `PROJECT_CONTEXT.md` (leia com
  `Artifact` `action: "read"`, ou o `WIREFRAME_*.html` local). Para **cada região e cada estado** do
  wireframe, compare com a tela real: estrutura, componentes, textos, estados (vazio, carregando,
  erro, permissão negada, modais) e comportamento interativo. Divergência é achado.
- **Artifact em dia:** se o frontend diverge do wireframe **porque mudou durante o execute**, o
  artifact deveria ter sido atualizado — wireframe desatualizado é **bloqueador** (o execute não
  cumpriu o `<frontend_wireframe>`). Se o desenho é que estava errado, o achado é a decisão do usuário,
  não uma correção silenciosa de nenhum dos lados.
- **Fidelidade ao design system:** os componentes e tokens usados são os do design system do projeto e
  das regras do `CLAUDE.md` do app de frontend (primitivas obrigatórias, proibições de estilo cru…)? Sem
  design system ou regras documentadas, diga isso e compare com os componentes existentes.
- **Tema e responsividade:** a tela funciona nos temas que o frontend suporta e em larguras
  menores/maiores relevantes? Sem overflow, sem texto cortado.
- **Acessibilidade básica:** foco visível e ordem de tab, papéis/rótulos, contraste, navegação por
  teclado nos controles novos.
- **Console e rede:** sem erro/warning novo no console do browser; chamadas à API conforme o contrato
  da sprint (sem requisição duplicada ou em loop).
- **Evidência:** cole o que viu — screenshots/saída do Playwright e a tabela
  `região/estado do wireframe → ✅/⚠️/❌ na tela real`.
</checklist>

<process>
Siga o **Contrato de Pair Programming** (§0 da methodology) em cada passo.

**Setup:**
1. **Resolver a sprint** a partir do brief e abrir o `SPRINT_NN_*.md` alvo.
2. Listar todos os arquivos modificados pela sprint (`git diff --name-only` ou tasks `[x]`).
3. Ler cada arquivo modificado **na íntegra** — não avalie o que não leu.
3.1. Se a sprint toca frontend: abrir o wireframe (URL no `PROJECT_CONTEXT.md`) e subir o app, para
   cumprir o item 7 do checklist depois da revisão de código.

**Revisão (por arquivo):**
4. Percorrer o checklist acima.
5. Registrar cada achado: `caminho:linha` → violação → sugestão de correção.
6. Classificar aplicando o `<guiding_question>`: **Bloqueador** é tudo que viola um critério de
   aceite, Object Calisthenics/MANDATORY, **ou** resolve com workaround/dívida técnica/fora do
   caminho conceitualmente correto — mesmo que "funcione" e não quebre nenhuma regra listada
   literalmente. **Melhoria** (YAGNI/KISS/DRY/Less Code — recomendação, não obrigatória) é o que já
   está correto na fonte mas poderia ser mais simples ou mais enxuto.

**Relatório:**
7. Entregue o relatório no formato do `<protocolo_subagent>`: resumo por arquivo (✅/⚠️/❌), **tabela do
   frontend contra o wireframe** (se aplicável), lista de achados com classe e tratamento proposto,
   veredicto final e cobertura.
8. **Pare.** O pai verifica os achados bloqueadores, corrige os simples, escala os sensíveis e, se houver
   correções, retoma você para uma **re-revisão restrita aos arquivos alterados** (releia o checklist só
   neles).
</process>

<guardrails>
- **Read-only.** Você não edita nenhum arquivo.
- **Citar sempre `caminho:linha`** antes de afirmar qualquer coisa sobre o código.
- **Não "melhorar" código adjacente** que não foi tocado pela sprint.
- Em toda bifurcação com mais de um caminho válido de correção, **PARE** (devolva `PARADA`) e marque o achado como `sensível`.
- **Suba a confusão, não a esconda** (§0.3): se o que o código faz, o que a sprint pediu e o que é
  conceitualmente correto não coincidirem, **não escolha um dos três em silêncio** — registre a
  contradição como achado, explique cada lado e devolva `PARADA` — a decisão é do usuário.
</guardrails>

<gate>
Devolva ao pai o veredicto (**Sprint aprovada** | **Sprint bloqueada**) e a cobertura. Se a sprint toca
entrada externa, autenticação/autorização, persistência, dados sensíveis ou LLM/prompt, **diga isso**: é
insumo para o pai decidir sobre o security.
</gate>
