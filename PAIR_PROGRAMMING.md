# Política e Estratégia de Programação em Pares (*Pair Programming*) — ESM Forum

Este documento descreve a política, os papéis, a logística e a operacionalização da prática de **Programação em Pares** (*Pair Programming*), pilares do **Extreme Programming (XP)**, adaptada para o desenvolvimento colaborativo da aplicação **ESM Forum**.

---

## 1. Introdução e Fundamentação Teórica

A **Programação em Pares** é uma das práticas de engenharia fundamentais do *Extreme Programming (XP)*, na qual dois desenvolvedores trabalham juntos e simultaneamente em um mesmo computador (ou sessão remota compartilhada) para construir uma única funcionalidade.

Na teoria da Engenharia de Software (Beck, 2000; Sommerville, 2011), a prática apoia-se no princípio da **revisão contínua de código em tempo real** (*continuous code review*), proporcionando benefícios substanciais:
* **Disseminação Coletiva de Conhecimento (*Collective Code Ownership*):** Evita "silos de conhecimento", garantindo que múltiplos membros dominem o backend (Node.js/Express/SQLite) e o frontend (React).
* **Redução Significativa de Bugs:** Estudos empíricos em XP demonstram que o trabalho em par detecta falhas de lógica, erros sintáticos e problemas de segurança no momento exato em que o código é digitado.
* **Elevação da Qualidade do Design:** O diálogo constante entre os pares força a busca por soluções mais simples, legíveis e alinhadas aos princípios de *Clean Code* e *Design Simples*.

---

## 2. Definição Dinâmica de Papéis

Durante as sessões de pair programming no *ESM Forum*, a dupla assume dois papéis distintos e complementares:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SESSÃO DE PAIR PROGRAMMING                   │
├────────────────────────────────┬────────────────────────────────┤
│      PILOTO (DRIVER)           │      NAVEGADOR (NAVIGATOR)     │
├────────────────────────────────┼────────────────────────────────┤
│ • Digita o código fonte        │ • Revisa o código em tempo real│
│ • Foca na sintaxe e detalhes   │ • Pensa na arquitetura global  │
│ • Implementa o escopo imediato │ • Antecipa bugs e edge cases   │
│ • Opera o teclado e terminal   │ • Guia a lógica e os testes    │
└────────────────────────────────┴────────────────────────────────┘
```

### A) Piloto (*Driver*)
* **Responsabilidade:** Responsável pela escrita ativa do código, execução dos comandos no terminal e navegação pelos arquivos da IDE.
* **Foco Mental:** Nível tático e operacional — sintaxe correta, nomes de variáveis legíveis, manipulação de rotas do Express e componentes React.
* **Atitude:** "Pensa em voz alta" (*talk out loud*), explicando o que está digitando para o navegador acompanhar a lógica.

### B) Navegador (*Navigator*)
* **Responsabilidade:** Observa o código que está sendo produzido, fornecendo feedback contínuo, sugestões de simplificação e consultando a documentação.
* **Foco Mental:** Nível estratégico e arquitetural — verificação dos critérios de aceitação da História de Usuário, prevenção de *edge cases*, alinhamento com a arquitetura de API e criação de cenários de teste.
* **Atitude:** Atua como parceiro estratégico, sem interromper desnecessariamente o fluxo do piloto, anotando ideias de refatoração para momentos de pausa.

---

## 3. Adaptação e Logística para Ambientes Remotos

Como o projeto é desenvolvido em ambientes remotos, a infraestrutura presencial de "uma tela e dois teclados" foi adaptada por meio de ferramentas modernas de colaboração síncrona:

### A) Ferramentas Utilizadas
1. **VS Code Live Share:** Permite o compartilhamento de sessão de código em tempo real com controle simultâneo do cursor, compartilhamento de servidor local (`localhost:5000` e `localhost:3000`) e terminal integrado.
2. **Discord / Microsoft Teams:** Canal de áudio/vídeo e compartilhamento de tela de alta taxa de quadros para comunicação verbal ininterrupta.
3. **GitHub Desktop & Git:** Sincronização e controle de versão das ramificações (*branches*).

### B) Protocolo e Frequência de Rotação de Papéis
Para evitar fadiga mental e garantir engajamento equitativo de ambos os membros, adotamos a técnica de **Pomodoro Pareado** com rotação de papéis:

* **Duração do Ciclo:** **25 a 30 minutos** de programação focada.
* **Troca de Papéis:** Ao final de cada ciclo de 30 minutos (ou a cada sub-tarefa/endpoint concluído), o **Piloto vira Navegador** e vice-versa.
* **Pausa:** Intervalo de 5 minutos a cada hora de sessão para alinhamento e descanso mental.

---

## 4. Aplicação Prática no Projeto ESM Forum

Para exemplificar a aplicação real da prática no projeto, descrevemos a dinâmica adotada durante a implementação do **Card 1 — Sistema de Votação em Perguntas e Respostas**:

1. **Início da Sessão (Alinhamento - 5 min):**
   * A dupla lê junta a História de Usuário e os critérios de aceitação no quadro do **GitHub Projects**.
   * Define-se o escopo da sessão: criação da rota `POST /perguntas/:id/voto` no backend.

2. **Ciclo 1 (Piloto A / Navegador B - 30 min):**
   * *Piloto A:* Abre a sessão no *VS Code Live Share* e digita a estrutura do endpoint no arquivo `routes/perguntas.js`.
   * *Navegador B:* Identifica no modelo de dados que falta decrementar a pontuação caso o voto seja negativo e sugere tratar o caso onde o ID da pergunta não existe (`404 Not Found`).

3. **Troca de Papéis (Piloto B / Navegador A - 30 min):**
   * *Piloto B:* Passa a operar o teclado para implementar os testes unitários/de integração no backend e a integração com o componente do React no frontend.
   * *Navegador A:* Acompanha a execução do servidor, inspeciona as respostas HTTP no terminal e valida o visual no navegador (`http://localhost:3000`).

4. **Conclusão e Review (10 min):**
   * Ambas as partes validam se todos os critérios de aceitação foram atendidos e realizam o commit assinado conjuntamente (*Co-authored-by*).

---

## 5. Benefícios Observados para o ESM Forum

1. **Aumento da Coesão do Código:** O backend e o frontend mantêm um padrão uniforme de estilo e arquitetura, já que todo trecho passa pela aprovação síncrona dos dois desenvolvedores.
2. **Mitigação de Erros de Integração:** Testar síncronamente as respostas do backend (`localhost:5000`) enquanto se desenvolve a interface em React (`localhost:3000`) eliminou erros de comunicação HTTP (`Failed to fetch`).
3. **Satisfação e Aprendizado Mútuo:** A troca contínua de experiências técnicas sobre Node.js, Express, React e SQLite fortaleceu o aprendizado prático da equipe.
