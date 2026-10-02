# CLASSROOM GAMIFICATION — versão corrigida

A interface original do sistema foi preservada. A principal mudança foi feita no funcionamento do armazenamento e da abertura dos jogos.

## O que foi corrigido

- Jogos cadastrados ficam persistidos no Firebase Firestore.
- Arquivos HTML/ZIP e capas são enviados ao Firebase Storage.
- A lixeira e a reativação também são persistidas.
- Jogos passam a abrir **dentro da própria plataforma**, em uma janela interna com iframe, sem abrir nova aba.
- Arquivos ZIP com `index.html` (ou o primeiro HTML encontrado) podem ser executados internamente; CSS, JS e recursos locais simples são carregados a partir do próprio ZIP.
- O plano de fundo enviado pelo usuário foi incorporado diretamente ao `index.html`, então não depende mais de `image_483032.jpg`.
- O Firebase mantém os arquivos e o catálogo principal.
- Uma Cloud Function espelha o catálogo em um arquivo `data/games.json` no GitHub. Assim, o GitHub mantém uma cópia/versionamento do catálogo sem expor um token de acesso no navegador.

## 1. Configurar o Firebase

No Firebase Console:

1. Crie ou abra seu projeto.
2. Ative Authentication → Sign-in method → Anonymous.
3. Crie o Firestore Database.
4. Ative o Storage.
5. Adicione um app Web e copie o objeto `firebaseConfig`.
6. Abra `index.html` e substitua os valores dentro de `const firebaseConfig` pelos valores do seu projeto.
7. Publique as regras `firestore.rules` e `storage.rules`.

## 2. Configurar o espelho GitHub

A Function precisa ser instalada/deployada pelo Firebase CLI.

Na pasta `functions`, execute:

```bash
npm install
```

Na raiz do projeto, configure os parâmetros:

```bash
firebase functions:secrets:set GITHUB_TOKEN
firebase functions:config:set # não é necessário para os parâmetros abaixo
```

Para os parâmetros `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_BRANCH` e `GITHUB_CATALOG_PATH`, o Firebase solicitará os valores durante o primeiro deploy da Function. Use, por exemplo:

- `GITHUB_OWNER`: seu usuário ou organização
- `GITHUB_REPO`: repositório da plataforma
- `GITHUB_BRANCH`: `main`
- `GITHUB_CATALOG_PATH`: `data/games.json`

O token GitHub deve ter permissão de conteúdo do repositório escolhido. Ele fica somente no backend da Function; **não** coloque o token dentro do `index.html`.

Depois:

```bash
firebase deploy --only functions,firestore:rules,storage
```

Para hospedar pela própria plataforma Firebase:

```bash
firebase deploy --only hosting
```

## 3. Observação importante sobre o GitHub

O navegador não deve receber um token pessoal do GitHub para gravar diretamente no repositório. Nesta versão, os arquivos dos jogos ficam no Firebase Storage e o catálogo/versionamento dos jogos é sincronizado pelo backend para o GitHub. Isso evita expor credenciais no código da página.

## Arquivos principais

- `index.html` — plataforma e interface original.
- `functions/index.js` — sincronização segura do catálogo com GitHub.
- `firebase.json` — configuração do Hosting/Functions/Firestore/Storage.
- `firestore.rules` — regras do catálogo.
- `storage.rules` — regras dos arquivos dos jogos.
