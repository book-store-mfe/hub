# Book Store MFE - Hub

Este repositório centraliza os microfrontends da Book Store via submodules Git.

## 📦 Módulos

* [store](https://github.com/book-store-mfe/store)
* [catalog](https://github.com/book-store-mfe/catalog)
* [account](https://github.com/book-store-mfe/account)
* [review](https://github.com/book-store-mfe/review)

---

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

Por exemplo, para rodar o **store**:

```sh
cd store
npm start
```

Para o **catalog**:

```sh
cd catalog
npm start
```

---

## Iniciando TODOS os módulos juntos

```sh
npm run start:all
```

Isso inicia todos os apps em paralelo.

---
