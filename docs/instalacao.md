
# Guia de Instalação e Execução — ESM Forum

Este documento descreve os pré-requisitos, instruções passo a passo e solução de problemas para a execução local da aplicação **ESM Forum** (Backend e Frontend) em ambiente Windows 11.

---

## 1. Pré-requisitos do Sistema

Para executar o projeto sem conflitos de compilação, certifique-se de ter as seguintes ferramentas instaladas:

* **Node.js:** Versão **v20 LTS** ou **v22 LTS** (Recomendado evitar versões *Current* como v24 para prevenir incompatibilidades de binários C++ do SQLite).
* **npm:** Versão **9.x** ou superior.
* **Git &amp; GitHub Desktop:** Para controle de versão e sincronização dos forks.
* **VS Code:** Editor de código recomendado.

---

## 2. Passo a Passo de Instalação e Execução

### A) Clonagem dos Repositórios
1. Realize o *fork* dos repositórios oficiais na sua conta do GitHub:
   * Backend: `esmforum`
   * Frontend: `esmforum-react`
2. No **GitHub Desktop**, clone ambos para a sua máquina local.

---

### B) Execução do Backend (`esmforum`)
1. Abra a pasta `esmforum` no VS Code.
2. Instale as dependências no terminal:
   ```bash
    npm install

1. Inicie o servidor:

   ```bash
    npm start

*(Ou* *node server.js* *). O servidor estará ativo em* *http://localhost:5000* *.*

&gt; **Nota sobre o Banco de Dados:** O projeto utiliza SQLite3 (`better-sqlite3`). Para resetar a base de dados em ambiente Linux/WSL, execute os scripts na pasta `bd/`.

---

### C) Execução do Frontend (`esmforum-react`)

1. Abra a pasta `esmforum-react` em um novo terminal do VS Code.
2. Instale as dependências e inicie a aplicação:

   ```bash
    npm install
    npm start

1. A interface gráfica abrirá automaticamente em `http://localhost:3000`.

---

## 3\. Resolução de Problemas Conhecidos (*Troubleshooting*)

* **Erro de compilação C++ /** **better-sqlite3** **(** **node-gyp** **):**
  * *Causa:* Uso do Node.js v24 (Current) sem ambiente de compilação Visual Studio instalado.
  * *Solução:* Instalar o Node.js v20/v22 LTS, reiniciar o VS Code e refazer o `npm install`.
* **Erro** **Failed to fetch** **no React:**
  * *Causa:* Executar o frontend sem o servidor backend ligado.
  * *Solução:* Manter dois terminais ativos (Backend na porta 5000 e Frontend na porta 3000).
* **Alertas do** **npm audit** **:**
  * *Recomendação:* **Não** executar `npm audit fix --force`, para evitar atualizações com quebras de compatibilidade (*breaking changes*).
