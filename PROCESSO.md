# Metodologia e Processo de Desenvolvimento — ESM Forum

> **Link do Quadro Visual:** [GitHub Projects — ESM Forum]

(https://github.com/users/alegameiro/projects/3/views/1?visibleFields=%5B%22Title%22%2C%22Assignees%22%2C%22Status%22%2C%22Linked+pull+requests%22%2C%22Sub-issues+progress%22%2C412433196%2C412434010%5D)

---

## 1. Abordagem Selecionada: Híbrida (Scrum + Kanban + XP)

Para o desenvolvimento e evolução do **ESM Forum**, adotou-se uma abordagem ágil integrada que combina a estrutura de gestão do **Scrum**, a visibilidade e controle de fluxo do **Kanban** e a excelência técnica do **Extreme Programming (XP)**.

### Justificativa da Escolha:
- **Scrum (Gestão de Iterações e Cerimônias):** Fornece uma cadência regular por meio de Sprints fixas de 2 semanas, delimitando objetivos claros (*Sprint Goals*) e promovendo ritos essenciais de alinhamento, inspeção e adaptação.
- **Kanban (Gestão de Fluxo e Limite de WIP):** Garante a gestão visual de todas as etapas de desenvolvimento, estabelecendo um sistema puxado (*pull system*) com limite rigoroso de Trabalho em Andamento (*Work in Progress — WIP = 2*) na coluna de desenvolvimento. Isso previne a sobrecarga da equipe, reduz a alternância de contexto (*multitasking*) e torna visíveis os gargalos da esteira.
- **Extreme Programming (XP - Práticas de Engenharia):** Garante a qualidade interna do código do backend e frontend através de disciplinas técnicas como *Pair Programming*, *Design Simples (YAGNI)*, *Refatoração Contínua*, *Testes Automatizados (TDD)* e *Integração Contínua (CI)*.

---

## 2. Papéis e Responsabilidades no Time

A equipe organiza-se de forma auto-gerenciada e multidisciplinar, distribuindo os seguintes papéis do Scrum/XP:

* **Product Owner (PO):** Responsável por maximizar o valor de negócio do produto, criar, ordenar e priorizar os itens do *Product Backlog* com base no valor para o usuário (usando níveis de prioridade `Alta`, `Média` e `Baixa`), além de validar os critérios de aceitação durante a *Sprint Review*.
* **Developers (Equipe de Desenvolvimento):** Responsáveis por estimar, decompor histórias em tarefas, auto-organizar a execução do trabalho e construir os incrementos de software com alta qualidade técnica conforme a *Definition of Done*.
* **Scrum Master / Facilitador:** Responsável por promover a eficácia do processo ágil, ajudar o time a remover impedimentos e assegurar que o limite de WIP do Kanban e as cerimônias do Scrum sejam respeitados.

---

## 3. Estrutura do Quadro Kanban e Fluxo de Trabalho

O quadro visual hospedado no **GitHub Projects** reflete o fluxo de valor do projeto, composto pelas seguintes colunas e regras de transição:

1. **`Product Backlog`**: Contém a lista priorizada de todas as Histórias de Usuário (*User Stories*) mapeadas para o projeto.
2. **`Sprint Backlog`**: Contém as histórias selecionadas durante o *Sprint Planning* para execução no ciclo atual de 2 semanas.
3. **`Em Progresso` — LIMITE WIP = 2**: Coluna onde ocorrem a implementação do código e os testes de unidade (sob a prática de *Pair Programming*).
   * **Regra do WIP:** A coluna suporta no máximo 2 cards simultâneos. Se a coluna atingir o limite de 2 itens, nenhum desenvolvedor pode puxar uma nova história do *Sprint Backlog*. A equipe deve focar em destravar e concluir o trabalho em andamento (*"Pare de começar e comece a terminar"*).
4. **`Em Revisão`**: Etapa onde o código desenvolvido passa por *Code Review* e validação dos critérios de aceitação.
5. **`Pronto`**: Onde ficam os incrementos finalizados que cumprem rigorosamente a *Definition of Done*.

---

## 4. Cadência do Ciclo de Desenvolvimento (Sprints e Cerimônias)

O desenvolvimento é estruturado em **Sprints de 2 semanas (10 dias úteis)**, seguindo a seguinte agenda de cerimônias:

| Cerimônia / Evento | Frequência / Momento | Participantes | Objetivo Principal |
| :--- | :--- | :--- | :--- |
| **Sprint Planning** | Dia 1 da Sprint (2h) | PO, SM e Devs | Definir o *Sprint Goal*, selecionar as Histórias de Usuário prioritárias do *Product Backlog* para o *Sprint Backlog* e quebrá-las em tarefas técnicas. |
| **Daily Scrum** | Diariamente (15 min) | Devs e SM | Sincronizar o trabalho diário em direção ao *Sprint Goal*, responder o que foi feito ontem, o que será feito hoje e identificar impedimentos. |
| **Refinamento do Backlog** | Meio da Sprint (1h) | PO e Devs | Detalhar, estimar e esclarecer dúvidas sobre histórias futuras no *Product Backlog*. |
| **Sprint Review** | Dia 10 da Sprint (1h) | PO, SM, Devs e Stakeholders | Demonstrar o incremento de software funcionando ao PO para inspeção e aceite com base nos critérios de aceitação. |
| **Sprint Retrospective**| Dia 10 da Sprint (1h) | SM e Devs | Inspecionar o processo, as interações e as ferramentas, identificando pontos fortes e ações de melhoria para a próxima Sprint. |

---

## 5. Práticas Técnicas de Extreme Programming (XP) Integradas

As práticas de engenharia do XP são aplicadas ao longo de todo o desenvolvimento para garantir sustentabilidade do código:

* **Pair Programming (Programação em Pares):** Desenvolvimento colaborativo onde dois desenvolvedores trabalham juntos sobre o mesmo código (alternando os papéis de *Driver* e *Navigator*), reduzindo defeitos e compartilhando conhecimento.
* **Design Simples & YAGNI (*You Aren't Gonna Need It*):** Foco em implementar a solução mais simples que atenda aos requisitos atuais, evitando abstrações ou funcionalidades prematuras para necessidades hipotéticas futuras.
* **Refatoração Contínua (Refactoring):** Melhoria constante da estrutura interna do código (legibilidade, acoplamento e coesão) sem alterar seu comportamento externo.
* **Desenvolvimento Dirigido por Testes (TDD / Test-First):** Escrita de testes automatizados para validar a lógica de negócio e garantir regressão zero.
* **Integração Contínua (CI):** Integração frequente do código ao repositório mestre, validando builds e evitando divergências de código.

---

## 6. Métricas de Acompanhamento do Processo

Para medir a eficiência do fluxo e o desempenho do time, são observadas duas métricas complementares:

* **Velocidade do Time (Métrica Scrum):** Quantidade de pontos/histórias concluídas por Sprint, auxiliando no planejamento de capacidade das Sprints subsequentes.
* **Lead Time e Throughput (Métricas Kanban):**
  * *Lead Time:* Tempo total transcorrido desde que a história entra em *In Progress* até ser movida para *Done*.
  * *Throughput:* Quantidade de entregas concluídas por unidade de tempo, garantindo vazão contínua.
