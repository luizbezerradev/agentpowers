# PROJECT_CONTEXT — {{NOME_DA_INICIATIVA}}

> Preencha cada seção a partir do estudo aprovado no `/codeform:brainstorm`. Linguagem **clara para qualquer dev**.
> Remova as instruções entre `<!-- -->` ao final. Seções marcadas **(obrigatória)** não podem faltar.

## 1. Objetivo deste documento
- Consolida o desenho funcional e operacional da iniciativa.
- Deixa claro **onde queremos chegar**, **por que** este é o desenho correto e **como** chegaremos lá.
- Lista as regras obrigatórias da iniciativa.

## 2. Objetivo final da implementação **(obrigatória)**
<!-- Descrição narrativa do estado final desejado. Fluxo final passo a passo. Regras de negócio
     obrigatórias. Resultado esperado. -->

## 2.1 Wireframe **(obrigatória se a iniciativa toca frontend)**
<!-- Link do artifact (canvas de alta fidelidade do /codeform:brainstorm) e caminho do
     docs/Sprints/{{iniciativa}}/WIREFRAME_{{TEMA}}.html. É a fonte de verdade visual: o
     /codeform:execute implementa o frontend com base nele e o atualiza quando o frontend mudar;
     o reviewer do /codeform:execute confere a UI real contra ele. Sem frontend: escreva "Não se aplica — sem
     frontend". -->
- Artifact: {{URL_DO_ARTIFACT}}
- Fonte: `docs/Sprints/{{iniciativa}}/WIREFRAME_{{TEMA}}.html`

## 3. Princípios obrigatórios de engenharia
<!-- Ex.: sem fluxo paralelo suportado; implementar direto na fonte; responsabilidades claras;
     idempotência e auditabilidade. Inclua aqui as regras MANDATORY relevantes desta iniciativa. -->

## 4. Diagnóstico do estado atual
<!-- O que está quebrado/incompleto hoje. Arquivos observados que evidenciam o problema
     (caminhos relativos à raiz do projeto). Conclusões diagnósticas. -->

## 5. O que a documentação oficial nos mostra
<!-- Opcional. Decisões embasadas em docs oficiais (via context7). Liste as referências. -->

## 6. Domínios e sprints candidatas **(obrigatória)**
<!-- Uma subseção por domínio. Lista as sprints que cobrirão esse domínio — cada sprint agrupa
     histórias relacionadas, sem virar um contrato formal por feature. -->
### 6.1 `{{dominio-A}}`
- {{história/capacidade-1}} — {{resumo de uma linha}}
- {{história/capacidade-2}} — {{resumo}}
### 6.2 `{{dominio-B}}`

## 7. Conceitos principais do domínio
<!-- Cada conceito/entidade com seus campos principais e como se relacionam. -->

## 8. Fluxo final, didático e completo **(obrigatória)**
### 8.1 Etapa 1 — {{NOME}}
### 8.2 Etapa 2 — {{NOME}}
<!-- Todas as etapas do fluxo, na ordem. -->

## 9. Árvore de arquivos e pastas impactadas **(obrigatória)**
<!-- Notação [criar] / [modificar] / [remover]. Caminhos relativos à raiz do projeto. -->
```text
docs/Sprints/{{iniciativa}}/
  PROJECT_CONTEXT.md           [criar]
  SPRINTS_OVERVIEW.md          [criar]
  SPRINT_01_{{TITULO}}.md      [criar]
  SPRINT_02_{{TITULO}}.md      [criar]
src/
  {{modulo}}/
    {{arquivo}}.ts              [criar]
    {{arquivo-existente}}.ts    [modificar]
```

## 10. Modelo de dados alvo
### 10.1 Tabelas/coleções novas ou remodeladas
### 10.2 Tabelas/coleções antigas a remover
### 10.3 Chaves e unicidades recomendadas

## 11. Como chegaremos lá — fases **(obrigatória)**
<!-- Fases sequenciais de alto nível. Cada fase vira uma ou mais sprints no /codeform:plan, com a
     entrega esperada. Esta seção vira a base do SPRINTS_OVERVIEW.md. -->
### Fase 1 — {{nome}} → entrega: {{...}}
### Fase 2 — {{nome}} → entrega: {{...}}

## 11.1 Plano de paralelização **(obrigatória)**
<!-- Contrato do §6.1 da methodology. O /codeform:plan escreve, o dry-run valida e o /codeform:orchestrate lê.
     Paralelismo que não está provado aqui não existe. Extraia os paths das TASKS de cada sprint e calcule
     as interseções — não presuma. Iniciativa inteira serial: veredito `serial` e as demais linhas só
     justificam (qual path/dependência impede). -->
- **Veredito:** {{serial | parcial | total}}

<!-- FORMATO OBRIGATÓRIO: diagrama em TRAÇOS (ASCII, dentro de ```text). Nada de tabela e nada de Mermaid.
     Um bloco por faixa: "BLOCO X  SNN  título  [TIER]" + o serviço/paths na linha de baixo. `|` e `v` = dependência
     (a de baixo só começa depois da de cima). Sprints paralelas ficam lado a lado, abrem com `+----+` e fecham
     com a linha "merge: SNN -> SMM". Depois do diagrama, uma linha "Como ler" e a lista "Paths que as tasks tocam"
     (uma linha por sprint) — é o insumo da interseção que o /codeform:orchestrate confere. -->
```text
 BLOCO A    S01  {{título}}                      [{{LEVE|MÉDIO|PESADO|CRÍTICO}}]
            |    {{serviço}}: {{paths}}
            v
 BLOCO B    S02  {{título}}                      [{{TIER}}]
            |    {{serviço}}: {{paths}}
            +-----------------------------+
            |                             |
            v                             v
 BLOCO C    S03  {{título}}               S04  {{título}}
 (PARALELO)      [{{TIER}}]                    [{{TIER}}]
                 {{serviço}}: {{paths}}        {{serviço}}: {{paths}}
            |                             |
            +------- merge: S03 -> S04 ---+
            |
            v
 BLOCO D    S05  {{título}}                      [{{TIER}}]
                 {{serviço}}: {{paths}}
```

**Como ler:** cada `|` e `v` é uma dependência; só as sprints lado a lado dentro do mesmo bloco rodam ao mesmo
tempo. O tier de cada sprint está entre colchetes.

**Paths que as tasks tocam** (insumo da interseção):
- S01 `{{paths}}`
- S02 `{{paths}}`

- **Arquivos de contenção** (editados por mais de uma sprint, ou lidos por uma depois de editados por outra):
  `{{arquivo}}` — dono único: SPRINT_NN. Sem dono único, as sprints não são paralelas.
- **Ordem de merge** de cada bloco paralelo: {{SPRINT_NN → SPRINT_MM}}.
- **Independência verificada por:** {{interseção de paths vazia entre as sprints do bloco X}}.

## 12. Casos de teste obrigatórios
### 12.1 {{cenário}}
<!-- Passo a passo do teste que prova que o objetivo foi atingido. -->

## 13. Riscos e decisões de implementação
### 13.1 Risco: {{...}}
- **Risco:** {{o que pode dar errado}}
- **Decisão:** {{o que faremos para evitar}}

## 14. Onde queremos chegar e como chegaremos lá **(obrigatória)**
### Onde queremos chegar
### Como chegaremos lá
### Regra final desta iniciativa
