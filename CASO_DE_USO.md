# Especificação Detalhada de Caso de Uso — ESM Forum

**Projeto:** ESM Forum  
**Documento:** Detalhamento do Caso de Uso (Parte 2 — Tarefa 2)  
**Funcionalidade:** Sistema de Votação em Perguntas e Respostas  

---

## 1. Identificação do Caso de Uso

* **Código:** `UC01`
* **Nome:** Votar em Pergunta ou Resposta
* **Ator Principal:** Usuário Autenticado (Membro registrado do fórum)
* **Atores Secundários:** Sistema ESM Forum (Backend e Banco de Dados)
* **Objetivo:** Permitir que usuários autenticados expressem sua avaliação (voto positivo ou negativo) sobre uma pergunta ou resposta publicada, garantindo a qualificação comunitária do conteúdo.
* **Nível:** Objetivo do Usuário (*User-Goal Level*)

---

## 2. Pré-condições

1. O sistema deve estar operacional com a API Backend (`http://localhost:5000`) e o Frontend React (`http://localhost:3000`) ativos.
2. O conteúdo (pergunta ou resposta) alvo da votação deve existir e estar visível na plataforma.
3. O usuário deve estar previamente autenticado na sessão ativa do navegador.

---

## 3. Pós-condições

1. O voto (positivo `+1` ou negativo `-1`) é registrado ou atualizado na tabela de votos do banco de dados.
2. O saldo líquido de votos (*score* = votos positivos - votos negativos) do conteúdo é recalculado e persistido.
3. A interface do usuário é atualizada em tempo real, destacando o botão correspondente ao voto computado e exibindo a nova pontuação.

---

## 4. Fluxo Principal (Caminho Feliz) — Registrar Voto Positivo ou Negativo

| Passo | Ação do Ator (Usuário) | Resposta do Sistema (ESM Forum) |
| :---: | :--- | :--- |
| **1** | O Usuário visualiza a lista de perguntas ou os detalhes de uma pergunta com suas respostas. | O Sistema exibe o conteúdo com os botões de votação (Upvote `▲` / Downvote `▼`) e o saldo atual de pontos. |
| **2** | O Usuário clica no botão de voto positivo (`Upvote`) ou voto negativo (`Downvote`) em uma pergunta ou resposta. | O Frontend intercepta o clique e verifica o estado de autenticação do usuário. |
| **3** | — | O Frontend envia uma requisição HTTP `POST /api/perguntas/:id/voto` (ou `/api/respostas/:id/voto`) contendo o ID do conteúdo, o ID do usuário e o tipo de voto (`tipo: +1` ou `tipo: -1`). |
| **4** | — | O Backend valida os dados, verifica as Regras de Negócio (`RN01` e `RN02`) e registra a transação no banco de dados. |
| **5** | — | O Backend atualiza a pontuação total do conteúdo e retorna a resposta HTTP `200 OK` com o novo saldo e o estado do voto. |
| **6** | O Usuário observa a atualização na tela. | O Frontend atualiza o contador numérico de votos e destaca a cor do botão selecionado (ex: verde para upvote, vermelho para downvote). |

---

## 5. Fluxos Alternativos

### FA01: Alteração de Voto (Mudar de Upvote para Downvote ou vice-versa)
* **Início:** No Passo 2 do Fluxo Principal, o Usuário clica no botão oposto ao seu voto atual (ex: clica em `Downvote` tendo anteriormente votado `Upvote`).
1. O Frontend identifica que o usuário já possui um voto ativo do tipo oposto para aquele conteúdo.
2. O Frontend envia a requisição HTTP `PUT /api/votos/:id` com o novo tipo de voto (`tipo: -1`).
3. O Backend recalcula o saldo descontando o voto anterior e somando o novo voto (variação líquida de 2 pontos).
4. O Backend persiste a alteração e retorna HTTP `200 OK`.
5. O Frontend remove o destaque do botão anterior, aplica o destaque no novo botão e atualiza o saldo exibido na tela.
* **Fim:** O caso de uso encerra no Passo 6 do Fluxo Principal.

### FA02: Remoção/Cancelamento de Voto
* **Início:** No Passo 2 do Fluxo Principal, o Usuário clica novamente no botão correspondente ao seu voto atual (ex: clica em `Upvote` quando seu voto atual já é `Upvote`).
1. O Frontend identifica que a ação representa a anulação do voto existente.
2. O Frontend envia a requisição HTTP `DELETE /api/votos/:id`.
3. O Backend remove o registro do voto do banco de dados e ajusta o saldo do conteúdo (removendo a pontuação anterior).
4. O Backend retorna HTTP `200 OK`.
5. O Frontend remove o destaque visual de ambos os botões de voto e atualiza o saldo de pontos na tela.
* **Fim:** O caso de uso encerra no Passo 6 do Fluxo Principal.

---

## 6. Fluxos de Exceção

### FE01: Usuário Não Autenticado tenta Votar
* **Início:** No Passo 2 do Fluxo Principal, um visitante anônimo clica em qualquer botão de votação.
1. O Frontend verifica que não há token/sessão de usuário ativa.
2. O Sistema bloqueia o envio da requisição HTTP.
3. O Sistema exibe um componente *Modal* ou mensagem de alerta: *"Você precisa estar conectado para votar. Deseja fazer login?"*.
4. O Usuário é redirecionado para a tela de Login ou opta por permanecer como leitor.
* **Fim:** O caso de uso é interrompido sem alterar os dados.

### FE02: Usuário tenta Votar no Próprio Conteúdo (Auto-votação)
* **Início:** No Passo 4 do Fluxo Principal, o Backend identifica que o `usuario_id` da requisição é idêntico ao `autor_id` da pergunta ou resposta.
1. O Backend rejeita a transação por violação da Regra de Negócio `RN02`.
2. O Backend retorna o código de erro HTTP `403 Forbidden` com a mensagem: `{"erro": "Não é permitido votar no próprio conteúdo."}`.
3. O Frontend exibe um aviso *Toast* no canto da tela com a mensagem de bloqueio.
* **Fim:** O caso de uso é interrompido mantendo a pontuação inalterada.

### FE03: Falha de Comunicação ou Erro Interno do Servidor
* **Início:** No Passo 3 ou 4 do Fluxo Principal, ocorre uma queda na conexão ou erro inesperado no banco de dados.
1. O Backend retorna o código HTTP `500 Internal Server Error` ou a requisição sofre *timeout*.
2. O Frontend captura o erro e reverte temporariamente qualquer mudança otimista feita na interface.
3. O Frontend exibe uma mensagem de erro: *"Não foi possível computar seu voto no momento. Tente novamente em instantes."*.
* **Fim:** O caso de uso é encerrado sem inconsistências no banco de dados.

---

## 7. Regras de Negócio (RN)

* **`RN01` — Unicidade de Voto por Conteúdo:** Um usuário registrado pode computar no máximo 1 voto por pergunta ou resposta (`+1` ou `-1`). A tabela de votos no banco deve ter uma restrição de chave única composta `(usuario_id, conteudo_id, tipo_conteudo)`.
* **`RN02` — Proibição de Auto-votação:** Um usuário é estritamente proibido de votar em perguntas ou respostas de sua própria autoria, evitando a inflação artificial de reputação.
* **`RN03` — Exigência de Autenticação:** Apenas usuários com conta ativa e autenticada podem interagir com a funcionalidade de votação. Visitantes anônimos possuem permissão apenas de leitura.
* **`RN04` — Cálculo Transacional do Score:** O saldo de votos de um conteúdo deve ser atualizado em uma transação atômica no banco de dados para evitar condições de corrida (*race conditions*) durante acessos simultâneos.

---

## 8. Requisitos Não Funcionais Associados

* **`RNF01` — Tempo de Resposta:** A requisição de votação deve ser processada pelo backend e refletida no frontend em menos de **200 milissegundos**.
* **`RNF02` — Usabilidade e Feedback Visual:** A interface deve fornecer feedback imediato indicando o estado do voto do usuário por meio de variações de cor e ícones legíveis.
