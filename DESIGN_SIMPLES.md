# Análise de Design Simples e Princípio YAGNI — ESM Forum

**Projeto:** ESM Forum (Backend & Frontend)  
**Disciplina:** Engenharia de Software 1 (ES1) — FGV  
**Etapa:** Parte 1 — Tarefa 3a (Extreme Programming & Design Simples)  

---

## 1. Fundamentação Teórica: Design Simples e YAGNI em Extreme Programming (XP)

No contexto de **Extreme Programming (XP)**, o projeto de software não é realizado de forma maciça no início do desenvolvimento (*Big Design Up Front — BDUF*), mas evolui de forma incremental à medida que o sistema é construído. Para orientar essa evolução sem permitir a degradação do código, Kent Beck definiu os pilares do **Design Simples** e o princípio **YAGNI (*You Aren't Gonna Need It*)**.

### 1.1. As Quatro Regras do Design Simples (Kent Beck)
Um código possui um design simples quando atende, em ordem de prioridade, aos seguintes critérios:
1. **Passa em todos os testes (*Runs all tests*):** O software deve funcionar corretamente e satisfazer os requisitos atuais.
2. **Revela a intenção (*Expresses intent*):** O código deve ser claro, legível e autoexplicativo para qualquer membro da equipe.
3. **Evita duplicação (*No duplication / DRY*):** Regras de negócio e estruturas repetidas devem ser abstraídas sem gerar complexidade excessiva.
4. **Minimiza o número de elementos (*Fewer classes/methods*):** Deve conter apenas as classes, métodos e variáveis estritamente necessários, sem abstrações ou artefatos mortos.

### 1.2. O Princípio YAGNI (*You Aren't Gonna Need It*)
O princípio YAGNI enuncia que um desenvolvedor **não deve adicionar funcionalidades ou abstrações baseadas em necessidades hipotéticas do futuro**. Em vez de antecipar requisitos incertos — o que gera complexidade especulativa e aumenta o custo de manutenção —, a equipe deve implementar a solução mais simples que funcione para os requisitos atuais (*Do the simplest thing that could possibly work*). Se e quando novos requisitos surgirem, o design será adaptado por meio de **Refatoração Contínua** sustentada por testes automatizados.

---

## 2. Análise do Código Atual do Backend (`routes/perguntas.js` e `routes/respostas.js`)

A análise das rotas de backend do *ESM Forum* (`routes/perguntas.js` e `routes/respostas.js`) revela um código focado na entrega direta dos requisitos fundamentais do fórum (listagem, consulta detalhada e criação de perguntas e respostas).

### 2.1. Aspectos que Seguam o Design Simples e YAGNI (Pontos Positivos)

1. **Mapeamento Direto aos Requisitos Atuais:**
   * **Observação:** As rotas HTTP mapeiam estritamente os casos de uso reais da aplicação: `GET /perguntas`, `GET /perguntas/:id`, `POST /perguntas`, `POST /perguntas/:id/respostas`.
   * **Conexão com YAGNI:** Não foram criadas rotas especulativas para funcionalidades ainda não solicitadas (como rotas complexas de exportação XML/PDF ou filtros avançados de busca).

2. **Ausência de Abstrações Prematuras:**
   * **Observação:** As rotas utilizam o roteador padrão do **Express.js** integrando-se diretamente ao modelo de dados do **SQLite** (`better-sqlite3`).
   * **Conexão com YAGNI:** Não foi inserida uma camada pesada de ORM (*Object-Relational Mapping*) ou padrões de projeto complexos (como *Abstract Factory* ou *Decorator*) para operações CRUD básicas. A solução atual resolve o problema de persistência local da maneira mais direta possível.

3. **Estrutura de Dados Enxuta:**
   * **Observação:** Os objetos de pergunta e resposta trafegam apenas com os atributos essenciais (`id`, `titulo`, `descricao`, `autor`, `data_criacao`).
   * **Conexão com YAGNI:** O modelo evita o "inchaço" de campos desnecessários (como atributos nulos para redes sociais, status complexos ou metadados de auditoria não solicitados).

---

### 2.2. Oportunidades de Simplificação e Refatoração (Melhorias de Design Simples)

Embora o código atual seja direto, a análise revela pontos onde as **Regras #2 (Revelar Intenção)** e **#3 (Evitar Duplicação)** de Kent Beck podem ser aprimoradas:

1. **Duplicação no Tratamento de Erros e Blocos `try/catch`:**
   * **Problema:** Cada rota repete blocos `try/catch` idênticos enviando respostas JSON de erro `500` com mensagens similares (`res.status(500).json({ erro: ... })`).
   * **Impacto:** Fere a regra de evitar duplicação (*DRY*) e aumenta a verbosidade do arquivo.
   * **Melhoria:** Centralizar o tratamento de exceções em um *middleware* global de erro do Express ou em uma função utilitária simples.

2. **Duplicação de Validação de Entradas no Corpo da Requisição:**
   * **Problema:** A checagem de campos obrigatórios (ex: verificar se `titulo` ou `descricao` estão presentes em `req.body`) é feita manualmente via múltiplos `if`s em cada endpoint.
   * **Impacto:** Polui a intenção principal da rota, que deveria focar em receber a requisição, acionar a lógica e retornar a resposta.
   * **Melhoria:** Extrair uma função auxiliar de validação simples `validarCamposObrigatorios(req.body, ['titulo', 'descricao'])`.

3. **Formatação Manual de Datas e Respostas:**
   * **Problema:** A formatação da data de criação (`new Date().toISOString()`) é repetida manualmente na criação de perguntas e de respostas.
   * **Melhoria:** Atribuir o valor default da data diretamente na tabela do SQLite (`DEFAULT CURRENT_TIMESTAMP`) ou centralizar em um utilitário.

---

## 3. Demonstração Prática de Refatoração (Antes x Depois)

### 3.1. Código Atual (Com Duplicação e Verbosidade)

```javascript
// routes/perguntas.js (Trecho Atual)
router.post('/perguntas', (req, res) => {
  try {
    const { titulo, descricao, autor } = req.body;
    
    // Validação manual repetida
    if (!titulo || !descricao || !autor) {
      return res.status(400).json({ erro: 'Todos os campos são obrigatórios.' });
    }

    const stmt = db.prepare('INSERT INTO perguntas (titulo, descricao, autor, data_criacao) VALUES (?, ?, ?, ?)');
    const info = stmt.run(titulo, descricao, autor, new Date().toISOString());

    res.status(201).json({ id: info.lastInsertRowid, titulo, descricao, autor });
  } catch (err) {
    res.status(500).json({ erro: 'Erro interno no servidor ao criar pergunta.' });
  }
});
```

### 3.2. Código Refatorado (Design Simples & YAGNI)

```javascript
// routes/perguntas.js (Versão Refatorada)
const { validarCampos } = require('../utils/validacao');

router.post('/perguntas', (req, res) => {
  validarCampos(req.body, ['titulo', 'descricao', 'autor']);

  const stmt = db.prepare('INSERT INTO perguntas (titulo, descricao, autor) VALUES (?, ?, ?)');
  const info = stmt.run(req.body.titulo, req.body.descricao, req.body.autor);

  res.status(201).json({ id: info.lastInsertRowid, ...req.body });
});
```

### Ganhos da Refatoração:
* **Maneira direta e limpa:** O endpoint reduz-se ao seu fluxo essencial (Validar -> Inserir -> Responder).
* **Intenção Clara:** Qualquer desenvolvedor lê o método e entende a intenção imediatamente.
* **Sem Complexidade Especulativa:** Não foram adicionadas abstrações genéricas; manteve-se o uso leve do SQLite.

---

## 4. Conclusão

O código atual das rotas do *ESM Forum* demonstra uma **excelente adesão inicial ao princípio YAGNI**, resolvendo os requisitos funcionais com o menor número possível de camadas. 

As oportunidades de melhoria identificadas não exigem rearquiteturar a aplicação, mas sim aplicar **refatorações pontuais** para eliminar duplicações e revelar a intenção do código. Essa abordagem pragmática reforça a visão de XP de que o código deve ser mantido limpo e simples hoje, garantindo que o sistema continue fácil de evoluir quando as 5 novas histórias de usuário da Sprint forem implementadas.
