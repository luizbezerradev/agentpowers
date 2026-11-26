# Sprint {{NN}} — {{Título da Sprint}}

## Objetivo da Sprint
{{Explicação narrativa do que esta sprint entrega e por que ela vem nesta posição da ordem lógica.}}

## Dependências
- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md)
- Sprint anterior: {{SPRINT_NN-1_*.md ou "nenhuma"}}

---

## Histórias de Usuário

### História 1: **{{COD}}-{{NNN}}** - {{Título da história}}

**Como** {{persona/ator}},
**Quero** {{capacidade}},
**Para** {{benefício/valor}}.

#### Critérios de Aceite:
- [ ] {{Critério observável 1 — comportamento visível/mensurável}}
- [ ] {{Critério observável 2}}
- [ ] {{Critério de qualidade: logs/erros/performance quando aplicável}}
- [ ] Testes cobrem o fluxo principal da história (unit/integration — quando aplicável)
- [ ] Object Calisthenics respeitados nos arquivos modificados (sem `else`, sem primitivos expostos, entidades ≤ 50 linhas)

**Task Breakdown:**

#### [{{módulo-ou-área}}]
<!-- Use o módulo/pacote/domínio do projeto (ex.: [auth], [billing], [frontend]) ou área
     ([Backend], [Frontend], [QA]). Cada task aponta o caminho do arquivo a criar/modificar e descreve
     o que muda — o mais descritivo possível.
     PATH: use caminho RELATIVO à raiz do projeto/repo. Evite caminhos absolutos de máquina
     (`/home/.../`), que apodrecem quando o repo muda de pasta. -->
- [ ] **{{COD}}-{{NNN}}.1** - {{Ação}} em `{{caminho/relativo/do/arquivo}}` — {{detalhe técnico do que muda}}
- [ ] **{{COD}}-{{NNN}}.2** - {{Ação}} em `{{caminho/relativo/à/raiz}}`

#### [QA]
- [ ] **{{COD}}-{{NNN}}.N** - {{Validação/teste a executar}}

---

### História 2: **{{COD}}-{{NNN+1}}** - {{Título}}
<!-- Repita o bloco de história quantas vezes necessário. Numere as tasks continuando a sequência. -->

---

## Definition of Done da Sprint

- {{Condição de negócio 1 que prova a sprint concluída}};
- {{Condição de negócio 2}};
- lint/typecheck do projeto passa **sem workarounds**;
- todas as tasks e critérios de aceite estão marcados `[x]` com evidência colada;
- **TDD (quando aplicável):** testes escritos antes ou junto ao código; todos passando;
- **Review (reviewer do /codeform:execute):** Object Calisthenics validados; YAGNI/KISS/DRY respeitados; nenhum `else` desnecessário, primitivo exposto ou getter/setter onde "Tell, Don't Ask" cabe;
- dá para seguir para a próxima sprint sem rediscutir esta base.

---

## Flags de risco e paralelização
<!-- Escrito pelo /codeform:plan. O /codeform:execute decide o security LENDO estas flags: omitir uma flag
     que o diff depois toca é gap. Flags possíveis: lógica de negócio crítica · superfície de segurança
     (entrada externa, auth, persistência, dado sensível, LLM) · API pública ou contrato · migração de
     dado em produção · frontend novo · nenhuma (scaffolding). -->
- **Flags de risco:** {{flags}}
- **Paralelização:** {{bloco e com quem roda em paralelo, ou "nenhuma" + o motivo}}
- **Tier:** {{leve | médio | pesado | crítico}} (tabela do §6.2 da methodology)

## Registro dos gates
<!-- Preenchido pelo /codeform:execute ao final. O /codeform:release bloqueia a branch se estiver vazio. -->
- **Review:** {{rodadas, veredicto — ou "dispensado: motivo"}}
- **Security:** {{veredicto e cobertura — ou "não se aplica: motivo"}}
- **Achados e correções aplicadas:** {{lista, ou "nenhum"}}
- **Riscos aceitos pelo usuário:** {{severidade + justificativa, ou "nenhum"}}
