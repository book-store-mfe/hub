# Book Store MFE - Hub

---

## 📖 Índice

* [Visão Geral](#visão-geral)
* [Módulos (MFEs)](#módulos-mfes)
* [Arquitetura e Fluxos](#arquitetura-e-fluxos)

  * [Composição via Submodules](#composição-via-submodules)
  * [Interação entre MFEs e Shared Lib](#interação-entre-mfes-e-shared-lib)
  * [Fluxo Completo: Deploy + Reports](#fluxo-completo-deploy--reports)
  * [Fluxo automático do Reports](#fluxo-automático-do-reports)
* [Deploy dos Microfrontends](#deploy-dos-microfrontends)
* [Automação de Reports](#automação-de-reports)

  * [Relatórios Gerados](#relatórios-gerados)
  * [Fluxo automático de atualização de reports](#fluxo-automático-de-atualização-de-reports)
* [Guia de Desenvolvimento Local](#guia-de-desenvolvimento-local)
* [npm link da Shared Lib](#npm-link-da-shared-lib)

---

## Visão Geral

Este repositório é o **ponto centralizador** dos microfrontends da Book Store, utilizando [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) para orquestrar todos os MFEs e a shared-lib.
* **Esse repositório não expõe nenhuma aplicação diretamente**, serve apenas como hub de orquestração, referência e documentação.
* O fluxo automatizado garante que sempre que um MFE mudar, os relatórios globais e testes de integração fiquem atualizados.
* O uso de submodules mantém cada app independente, mas permite integração e testes globais reais.

---

## preview

![preview](./.assets/preview.png)

---

## Módulos (MFEs)

| Projeto                  | Função                           |
| ------------------------ | -------------------------------- |
| [store][store]           | Host Shell (aplicação principal) |
| [catalog][catalog]       | Catálogo de livros               |
| [account][account]       | Área do usuário                  |
| [review][review]         | Reviews de livros                |
| [shared-lib][shared-lib] | Biblioteca compartilhada         |
| [reports][reports]       | Reports automáticos (Smoke + Dependencies)   |

[store]: https://github.com/book-store-mfe/store
[catalog]: https://github.com/book-store-mfe/catalog
[account]: https://github.com/book-store-mfe/account
[review]: https://github.com/book-store-mfe/review
[shared-lib]: https://github.com/book-store-mfe/shared-lib
[reports]: https://github.com/book-store-mfe/reports

---

### Interação entre MFEs e Shared Lib

```mermaid
flowchart TD
  Store((store))
  Catalog((catalog))
  Account((account))
  Review((review))
  SharedLib[(shared-lib)]

  Store --"import"--> SharedLib
  Account --"import"--> SharedLib
  Review --"import"--> SharedLib
  Store --"lazy load remote"--> Catalog
  Store --"lazy load remote"--> Account
  Catalog --"lazy load remote"--> Review
```

---

### Fluxo Completo: Deploy + Reports

```mermaid
sequenceDiagram
    participant Dev as Dev
    participant MFERepo as MFE Repo (store/account/...)
    participant GitHub as GitHub Actions (no MFE)
    participant Reports as reports Repo (CI)
    participant GHPages as GitHub Pages

    Dev->>MFERepo: Faz push na branch main
    MFERepo->>GitHub: Trigger workflow (build+deploy)
    GitHub->>GHPages: Publica build no gh-pages
    GitHub->>Reports: Dispara workflow via repository_dispatch
    Reports->>Reports: Atualiza reports (Smoke + Dependencies)
    Reports->>GHPages: Publica relatório HTML do smoke test
    Reports->>Reports: Atualiza DEPENDENCIES-REPORT.md
```

---

### Fluxo automático do Reports

#### O que acontece ao atualizar qualquer MFE:

1. **Commit/push** em qualquer repositório de MFE (ex: `catalog`).
2. O **GitHub Actions** do MFE:

   * Builda e faz deploy no GitHub Pages do respectivo MFE.
   * **Dispara um `repository_dispatch`** para o repositório `reports`.
3. O repositório `reports`:

   * Roda **Smoke Tests E2E** (testa host+remotes juntos).
   * Atualiza e publica:

     * **Smoke Test** (HTML report)
     * **Dependencies Report** (validação das dependências)
   * Publica o relatório HTML no GitHub Pages de `reports`.

#### 🔍 Fluxo do Reports

```mermaid
flowchart TD
    DevPush([Push em qualquer MFE])
    ActionMFE([GitHub Action<br>no MFE])
    RepoReports([Repo de reports])
    SmokeTest([Rodar Smoke Test])
    Dependencies([Gerar Dependencies Report])
    Pages([Publicar em GH Pages])

    DevPush --> ActionMFE
    ActionMFE -->|repository_dispatch| RepoReports
    RepoReports --> SmokeTest
    RepoReports --> Dependencies
    SmokeTest --> Pages
    Dependencies --> RepoReports
```

---

## Automação de Reports

### Relatórios Gerados

| Relatório                   | Local / Link                                                                                           | Descrição                  |
| --------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------- |
| **Smoke Test (HTML)**       | [GH Pages - Reports](https://book-store-mfe.github.io/reports/)                                        | testes integrando todos MFEs |
| **Dependencies (Markdown)** | [`DEPENDENCIES-REPORT.md`](https://github.com/book-store-mfe/reports/blob/main/DEPENDENCIES-REPORT.md) | Árvore e links entre MFEs  |

### Fluxo automático de atualização de reports

* **Disparo automático**: todo deploy de MFE dispara update dos reports.
* **Relatórios publicados**: HTML navegável (GH Pages), Markdown de dependências.

---

## Guia de Desenvolvimento Local

## 🚀 Como clonar o repositório com todos os submodules

```sh
git clone --recurse-submodules https://github.com/book-store-mfe/hub.git
```

Ou, se já clonou sem submodules:

```sh
git submodule update --init --recursive
```

---

## 🔄 Atualizando todos os submodules

```sh
git submodule update --remote --merge
```

---

## Iniciando um módulo específico

Por exemplo, para rodar o **catalog**:

```sh
npm install:catalog
npm start:catalog
```

---

## Iniciando TODOS os módulos juntos

```sh
npm run install:all
npm run start:all
```

Isso inicia todos os apps em paralelo.

---

## 🔗 Usando `shared-lib` localmente com `npm link`

Durante o desenvolvimento, é possível consumir o pacote `shared-lib` **localmente**, sem precisar publicar no NPM, usando o comando `npm link`.

### Passo a passo para integração local

#### 1. Build do shared-lib

Entre na pasta do submodule `shared-lib`:

```sh
cd shared-lib
npm install
npm run build
```

#### 2. Link global do shared-lib

```sh
cd dist/shared-lib
npm link
```

Isso vai registrar o pacote localmente na sua máquina.

#### 3. Link nos MFEs (store, catalog, account, review)

Para cada microfrontend que consome o shared-lib, rode:

```sh
cd ../store      # (ou catalog, account, review)
npm link @bookstore-app/shared-lib
```

Repita para todos os MFEs que vão consumir a shared-lib.

#### 4. Rodando em dev

* Ao rodar `npm start` em cada MFE, o código da `shared-lib` será importado diretamente do seu diretório local.
* Sempre que fizer alterações no código da lib, execute `npm run build` novamente dentro de `shared-lib` para propagar as mudanças.
* Os MFEs verão a versão mais recente do código buildado automaticamente (pode ser necessário reiniciar a aplicação em alguns casos).

#### 5. Removendo o link (opcional)

Se quiser voltar a usar a versão publicada no NPM:

```sh
cd store
npm unlink @bookstore-app/shared-lib
```

Repita para cada microfrontend que usava o link.

---

### 📑 Dicas

* O `npm link` cria symlinks. Pode ter conflitos se dependências peer (ex: Angular) não forem iguais. Se tiver erros estranhos, limpe o `node_modules` e reinstale tudo.
* Lembre-se de sempre rodar o build da lib após alterar o código-fonte.
* Esse fluxo é excelente para acelerar o desenvolvimento multi-repo sem precisar publicar releases no npm a cada alteração.

---
