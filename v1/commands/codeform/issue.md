---
name: codeform:issue
description: "Cria uma issue no GitHub a partir de um problema relatado: estudo breve na codebase (causas e soluções possíveis), título e corpo pelos templates de issue do repositório e preenchimento do board, se o projeto definir um."
argument-hint: "<descrição do problema ou da melhoria>"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Agent
---

Problema relatado: $ARGUMENTS

## 1. Entender o relato

Classifique como **bug**, **melhoria** ou **tarefa técnica**. Se o relato for ambíguo
a ponto de mudar a investigação, pergunte antes de seguir; caso contrário, assuma e declare a suposição.

## 2. Estudo breve (somente leitura)

Localize na codebase o ponto onde o problema nasce. Seja rápido: o objetivo é uma hipótese fundamentada, não uma
auditoria. Leia os arquivos inteiros que for citar. Não altere código.

Produza:
- **Área/serviço afetado**, entre as opções que o template escolhido no passo 3 oferecer (se houver).
- **Possíveis causas**, com `caminho/arquivo.ts:linha`. Marque o que foi **verificado no código** e o que é **hipótese**.
- **Possíveis soluções**, com o trade-off de cada uma e a recomendada.

Se não achar a causa, diga isso na issue; nunca invente.

## 3. Template da issue

**Título e corpo vêm dos templates do repositório**, nunca de um padrão desta skill. Leia
`.github/ISSUE_TEMPLATE/` (e o `config.yml`, se houver) e escolha o template pelo `name`/`description`/`labels`
que casa com o tipo do passo 1. Do template, use:

- `title` e qualquer regra de título descrita no bloco `markdown` (prefixo, formato, tamanho). Regra que
  exige uma decisão sua (ex.: qual prefixo) e que o relato não responde: pergunte antes de criar. Nunca copie
  o prefixo de um exemplo.
- `labels`.
- Corpo: para cada campo do `body` que não seja `markdown`, nesta ordem, uma seção `### <label>` seguida do
  conteúdo (a opção escolhida, no dropdown). Preencha todos os campos `required` e o que o estudo do passo 2
  trouxer; campo opcional sem conteúdo é omitido. Escreva no idioma do repositório.

O `gh issue create` não renderiza o formulário, por isso você monta o corpo no formato acima. Template em
Markdown (`.md`): use-o como está, e `gh issue create --template` serve.

**Issue em branco** só quando o caso **não se encaixa em nenhum** template, ou quando o repositório não tem
`.github/ISSUE_TEMPLATE/`. Declare ao usuário que nenhum template serve e por quê, siga o `CLAUDE.md` do
projeto para título e corpo e, na falta dele, use um título curto e as seções do passo 2. Na dúvida entre um
template e o branco, use o template.

## 4. Board do projeto (se houver)

Se o `CLAUDE.md` do projeto descreve um board/projeto do GitHub (campos como Size, Estimate, Priority, Status,
critérios para derivá-los, ids), derive os valores **do estudo, não do chute**, mostre ao usuário o resumo
(título, área, causa provável, valores do board e a justificativa em uma linha) e **aguarde a confirmação
antes de criar**. Sem board descrito: pule os passos de board, mostre o resumo (título, área, causa provável)
e aguarde a confirmação do mesmo jeito.

## 5. Criar a issue e preencher o board

Depois da confirmação, grave o corpo em um arquivo temporário e crie a issue:

```bash
url=$(gh issue create -t "<título>" -l <labels do template> -F <arquivo.md>)
```

Com board descrito no `CLAUDE.md`: obtenha o item da issue (se vier vazio, aguarde alguns segundos ou
adicione-o com `gh project item-add`), preencha os campos com `gh project item-edit` usando os ids que o
`CLAUDE.md` traz. Id inexistente: releia `gh project field-list <n> --owner <org> --format json` e use os
atuais. Em zsh, mantenha o argumento de `-q` entre aspas (colchetes viram glob).

## 6. Verificar e reportar

Releia a issue (`gh issue view`) e, havendo board, os valores do item via GraphQL (`fieldValues`). Reporte a
URL e os valores. Se algo não foi gravado, diga qual campo falhou; não declare sucesso sem a leitura.
