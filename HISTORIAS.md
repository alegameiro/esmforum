# Especificação de Histórias de Usuário e Critérios de Aceitação — ESM Forum

Este documento apresenta a especificação detalhada de **3 Histórias de Usuário** selecionadas a partir do backlog do **ESM Forum**, utilizando a estrutura clássica dos **3 C's** (*Cartão*, *Conversa* e *Confirmação/Critérios de Aceitação*) do Extreme Programming (XP), a métrica de estimativa por **Story Points** (Planning Poker), a validação pelo acrônimo **INVEST** e a **Matriz de Priorização** orientada a valor de negócio.

---

## 1. Visão Geral das Histórias Selecionadas

As três funcionalidades escolhidas para o detalhamento representam o núcleo funcional que transforma o fórum de um repositório estático em uma plataforma colaborativa e pesquisável:

| ID | Título da História | Épico | Prioridade | Story Points |
| :--- | :--- | :--- | :---: | :---: |
| **US01** | Sistema de Votação em Perguntas e Respostas | Engajamento & Qualidade | **Alta (P1)** | **5 SP** |
| **US02** | Busca de Perguntas por Palavras-chave | Descoberta & Recuperação | **Alta (P2)** | **3 SP** |
| **US03** | Categorização de Perguntas por Tags | Organização de Conteúdo | **Média (P3)** | **3 SP** |

---

## 2. Detalhamento das Histórias de Usuário (Modelo 3 C's)

---

### História 1: US01 — Sistema de Votação em Perguntas e Respostas

#### Cartão (Card)
> **Como** membro ativo da comunidade do fórum,  
> **Eu quero** votar positivamente (*upvote*) ou negativamente (*downvote*) em perguntas e respostas,  
> **Para** destacar os conteúdos mais úteis/relevantes e desencorajar postagens incorretas ou fora de tópico.

* **ID:** `US01`
* **Épico:** Engajamento e Avaliação de Conteúdo
* **Estimativa:** `5 Story Points` (Pequena/Média complexidade com regra de consistência de votos)

---

#### Conversa (Conversation)
* **Product Owner (PO):** *"Precisamos de um mecanismo para que a própria comunidade qualifique as respostas. Respostas certas e explicadas devem subir no topo, enquanto informações desatualizadas devem perder visibilidade."*
* **Desenvolvedor Backend:** *"Como garantiremos que um usuário não vote várias vezes na mesma pergunta para inflar o placar?"*
* **PO:** *"Cada usuário só pode registrar um voto por item. Se ele clicar em 'upvote' novamente, o voto é cancelado. Se mudar de 'upvote' para 'downvote', o saldo deve recalcular a diferença (-2)."*
* **Desenvolvedor Frontend:** *"Visitantes não autenticados podem votar?"*
* **PO:** *"Não. Visitantes anônimos podem visualizar a pontuação final, mas ao tentar votar, devem ser redirecionados para a tela de login."*
* **Analista de Testes (QA):** *"E se o autor da pergunta votar na própria publicação?"*
* **PO:** *"Regra de negócio: o autor não pode votar no próprio conteúdo para evitar fraude de engajamento."*

---

#### Confirmação (Critérios de Aceitação / BDD)

```gherkin
Cenário 1: Voto positivo registrado com sucesso
  Dado que o usuário está autenticado na plataforma
  E visualiza uma pergunta ou resposta enviada por outro membro
  Quando clica no botão "Upvote" (Seta para cima)
  Então o sistema registra o voto positivo
  E o contador de pontos da publicação é incrementado em +1
  E o botão de "Upvote" fica visualmente destacado (ativo).

Cenário 2: Cancelamento de voto prévio (Toggle)
  Dado que o usuário já havia votado "Upvote" em uma pergunta
  Quando ele clica novamente no botão "Upvote"
  Então o sistema remove o voto do usuário
  E o contador de pontos retorna ao valor anterior (-1)
  E o botão de "Upvote" volta ao estado neutro.

Cenário 3: Alteração direta de voto (Upvote para Downvote)
  Dado que o usuário havia dado "Upvote" em uma resposta (saldo +1)
  Quando ele clica diretamente no botão "Downvote"
  Então o sistema altera o voto de positivo para negativo
  E o saldo de pontuação da resposta é decrementado em 2 pontos
  E o botão de "Downvote" fica destacado.

Cenário 4: Impedimento de voto no próprio conteúdo
  Dado que o usuário é o autor da pergunta exibida
  Quando ele tenta clicar nos botões de "Upvote" ou "Downvote"
  Então o sistema desabilita os botões de voto
  E exibe a mensagem: "Você não pode votar na sua própria publicação."

Cenário 5: Tentativa de voto por usuário anônimo
  Dado que o visitante não está autenticado
  Quando ele clica em votar em qualquer publicação
  Então o sistema exibe um modal/alerta solicitando login
  E redireciona o visitante para a página de autenticação.
```

---

#### Validação INVEST e Racional de Estimativa (5 SP)
* **Independent:** Pode ser implementada sobre o modelo existente de perguntas/respostas adicionando a tabela associativa de votos.
* **Negotiable:** O leiaute dos botões e o aviso de bloqueio para o autor foram negociados na conversa com o PO.
* **Valuable:** Prioridade máxima para ordenar o conteúdo do fórum por relevância.
* **Estimable:** Estimada em **5 Story Points** no Planning Poker devido às regras de transição de estado (-1, 0, +1) e validações de integridade no backend.
* **Small:** Cabe dentro da Sprint de 2 semanas.
* **Testable:** 5 cenários BDD claros e automatizáveis em testes de integração da API e UI.

---

### História 2: US02 — Busca por Palavras-chave

#### Cartão (Card)
> **Como** usuário ou visitante do fórum,  
> **Eu quero** pesquisar perguntas digitando termos ou palavras-chave no campo de busca,  
> **Para** encontrar respostas para minhas dúvidas rapidamente sem criar tópicos duplicados.

* **ID:** `US02`
* **Épico:** Descoberta e Recuperação de Informação
* **Estimativa:** `3 Story Points` (Complexidade simples/média utilizando filtros SQL/SQLite)

---

#### Conversa (Conversation)
* **PO:** *"A barra de busca deve ser proeminente no cabeçalho. Ao digitar termos como 'React' ou 'SQLite', o fórum deve listar perguntas que contenham essas palavras no título ou no corpo."*
* **Desenvolvedor Backend:** *"A busca deve considerar letras maiúsculas/minúsculas ou acentuação?"*
* **PO:** *"A busca deve ser insensível a maiúsculas e minúsculas (case-insensitive). Se buscar 'node', deve encontrar 'Node.js' ou 'NODE'."*
* **Desenvolvedor Frontend:** *"Exibiremos paginação se houver muitos resultados?"*
* **PO:** *"Sim, e se nenhum resultado for encontrado, a tela deve exibir uma mensagem amigável convidando o usuário a criar a pergunta."*

---

#### Confirmação (Critérios de Aceitação / BDD)

```gherkin
Cenário 1: Busca realizada com sucesso por palavra-chave
  Dado que o usuário está em qualquer página do fórum
  Quando ele digita "middleware" na barra de busca e pressiona Enter
  Então o sistema retorna todas as perguntas que contêm o termo "middleware" no título ou na descrição
  E destaca o termo buscado nos resultados exibidos.

Cenário 2: Busca insensível a caixa (Case Insensitive)
  Dado que existem perguntas cadastradas com os títulos "Erro no SQL" e "sqlite bug"
  Quando o usuário busca por "sql"
  Então ambas as perguntas devem ser listadas nos resultados.

Cenário 3: Busca sem resultados correspondentes
  Dado que o usuário digita um termo inexistente ("xyz123")
  Quando ele confirma a busca
  Então o sistema exibe a mensagem: "Nenhuma pergunta encontrada para 'xyz123'."
  E apresenta o botão: "Faça essa pergunta à comunidade".

Cenário 4: Busca com campo em branco
  Dado que o usuário está com a barra de busca vazia
  Quando clica no ícone de pesquisa
  Então o sistema mantém a listagem geral de perguntas sem realizar requisição desnecessária.
```

---

#### Validação INVEST e Racional de Estimativa (3 SP)
* **Independent:** Não depende da votação nem de tags, operando diretamente sobre a tabela de perguntas.
* **Negotiable:** A experiência de feedback para busca vazia foi definida dinamicamente.
* **Valuable:** Evita perguntas duplicadas e acelera a resolução de dúvidas de estudantes.
* **Estimable:** Estimada em **3 Story Points** devido à simplicidade da query SQL (`LIKE %termo%` ou operador de busca textual no SQLite).
* **Small & Testable:** Cenários objetivos e de rápida verificação automatizada.

---

### História 3: US03 — Categorização de Perguntas por Tags

#### Cartão (Card)
> **Como** autor de uma pergunta,  
> **Eu quero** associar uma ou mais tags (ex: `#javascript`, `#banco-de-dados`) à minha publicação,  
> **Para** organizar o tópico por assunto e facilitar a navegação de especialistas na área.

* **ID:** `US03`
* **Épico:** Organização de Conteúdo e Taxonomia
* **Estimativa:** `3 Story Points` (Modelagem de relacionamento N:N entre Pergunta e Tag)

---

#### Conversa (Conversation)
* **PO:** *"Queremos que cada pergunta tenha de 1 a no máximo 5 tags para categorizar o assunto."*
* **Desenvolvedor Backend:** *"As tags serão livres ou pré-definidas?"*
* **PO:** *"No MVP, o usuário pode digitar novas tags ou selecionar tags já existentes sugeridas pelo autocomplete."*
* **Desenvolvedor Frontend:** *"Clicar em uma tag na tela inicial deve filtrar as perguntas daquela tag?"*
* **PO:** *"Exatamente! Clicar na tag '#node' deve filtrar o feed exibindo apenas perguntas marcadas com essa tag."*

---

#### Confirmação (Critérios de Aceitação / BDD)

```gherkin
Cenário 1: Inclusão de tags na criação de pergunta
  Dado que o usuário está preenchendo o formulário de nova pergunta
  Quando ele digita e seleciona as tags "react" e "javascript"
  E envia a pergunta
  Então a pergunta é salva e exibe os marcadores visualmente como "#react" e "#javascript".

Cenário 2: Validação de limite de tags por pergunta
  Dado que o usuário já adicionou 5 tags à sua pergunta
  Quando ele tenta adicionar a 6ª tag
  Então o sistema bloqueia a inserção
  E exibe a mensagem: "Você pode adicionar no máximo 5 tags por pergunta."

Cenário 3: Filtragem de perguntas por clique na tag
  Dado que o usuário está visualizando a listagem de perguntas
  Quando ele clica sobre a etiqueta "#express" em qualquer card
  Então o fórum recarrega o feed exibindo apenas perguntas categorizadas com a tag "#express".
```

---

#### Validação INVEST e Racional de Estimativa (3 SP)
* **Independent:** Pode ser desenvolvida de forma independente da votação e da busca textual.
* **Negotiable:** O limite de 5 tags foi acordado em conversa para evitar poluição visual.
* **Valuable:** Melhora a navegação do fórum por domínio de conhecimento.
* **Estimable:** Estimada em **3 Story Points** devido à tabela intermediária de relacionamento N:N (`Pergunta_Tag`).
* **Small & Testable:** Critérios objetivos com testes de limites (máximo de 5 tags) e filtragem de feed.

---

## 3. Matriz de Priorização e Justificativa de Execução (Simulação de PO)

Na engenharia de software ágil, a priorização do backlog é conduzida pelo **Product Owner (PO)** avaliando a relação entre **Valor de Negócio**, **Incerteza/Risco Técnico** e **Dependência de Arquitetura**:

```
        ▲ ALTO
        │   [US01] Sistema de Votação
  VALOR │   (Diferencial colaborativo core)
   DE   │                                   [US02] Busca por Palavra-chave
NEGÓCIO │                                   (Urgente para usabilidade)
        │
        │                                   [US03] Categorização por Tags
        │                                   (Organização secundária)
        └───►
            BAIXO ─────────────────────────► ALTO
                     ESFORÇO / COMPLEXIDADE TÉCNICA
```

### Racional da Ordem de Execução no Sprint Backlog:

1. **1º Lugar: `US01` — Sistema de Votação em Perguntas e Respostas (P1 - 5 SP)**
   * **Justificativa:** É a funcionalidade central de engajamento do fórum. Ela resolve o problema do ruído de informações, permitindo que as melhores respostas ganhem destaque. Possui o maior valor de negócio transformacional para a plataforma.
2. **2º Lugar: `US02` — Busca por Palavras-chave (P2 - 3 SP)**
   * **Justificativa:** À medida que a base de perguntas cresce com o uso do sistema, a busca torna-se essencial para evitar duplicações de postagens e retrabalho dos membros da comunidade.
3. **3º Lugar: `US03` — Categorização por Tags (P3 - 3 SP)**
   * **Justificativa:** Complementa a funcionalidade de busca, organizando a taxonomia do fórum por assuntos específicos. Pode ser implementada logo após o mecanismo básico de busca textual estar estável.

---

## 4. Conclusão

Com a especificação das 3 Histórias de Usuário validadas pelo padrão 3 C's e alinhadas aos critérios de aceitação BDD, garantimos uma ponte transparente entre as necessidades do usuário e a equipe de desenvolvimento.

A **História US01 (Sistema de Votação)** foi selecionada para o detalhamento formal em formato de **Caso de Uso** e modelagem de **Diagramas UML** na sequência do projeto.
