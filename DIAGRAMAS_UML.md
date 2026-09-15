# Diagramas de Modelagem UML — ESM Forum

Este documento reúne os diagramas de modelagem técnica orientada a objetos para o sistema **ESM Forum**, elaborados com a notação **UML** utilizando a sintaxe **Mermaid** (compatível com a renderização nativa do GitHub).

---

## 1. Diagrama de Classes (Class Diagram)

O Diagrama de Classes representa a estrutura estática do domínio do **ESM Forum**, destacando as entidades de negócio, seus atributos, métodos e os relacionamentos de associação, agregação e composição.

```mermaid
classDiagram
    class Usuario {
        +Int id
        +String nome
        +String email
        +String senhaHash
        +String fotoUrl
        +String bio
        +DateTime dataCadastro
        +cadastrar()
        +autenticar()
        +obterEstatisticas()
    }

    class Pergunta {
        +Int id
        +String titulo
        +String conteudo
        +DateTime dataCriacao
        +Int idUsuario
        +Int saldoVotos
        +Boolean resolvida
        +criar()
        +editar()
        +adicionarTag(Tag tag)
        +atualizarSaldoVotos()
    }

    class Resposta {
        +Int id
        +String conteudo
        +DateTime dataCriacao
        +Int idPergunta
        +Int idUsuario
        +Int saldoVotos
        +Boolean marcadaComoMelhor
        +criar()
        +editar()
        +marcarComoMelhor()
        +atualizarSaldoVotos()
    }

    class Voto {
        +Int id
        +Int idUsuario
        +Int tipoVoto  %% +1 para Upvote, -1 para Downvote
        +String tipoEntidade %% "PERGUNTA" ou "RESPOSTA"
        +Int idEntidade
        +DateTime dataVoto
        +registrar()
        +alterar()
        +cancelar()
    }

    class Tag {
        +Int id
        +String nome
        +String descricao
        +criar()
        +listarPopulares()
    }

    class Notificacao {
        +Int id
        +Int idUsuario
        +String mensagem
        +Boolean lida
        +DateTime dataEnvio
        +marcarComoLida()
    }

    %% Relacionamentos
    Usuario "1" -- "0..*" Pergunta : "publica >"
    Usuario "1" -- "0..*" Resposta : "escreve >"
    Usuario "1" -- "0..*" Voto : "realiza >"
    Usuario "1" -- "0..*" Notificacao : "recebe >"
    
    Pergunta "1" *-- "0..*" Resposta : "contém (composição) >"
    Pergunta "0..*" o-- "1..5" Tag : "categorizada por (agregação) >"
    
    Pergunta "1" -- "0..*" Voto : "recebe >"
    Resposta "1" -- "0..*" Voto : "recebe >"
```

---

## 2. Diagrama de Sequência (Sequence Diagram) — UC01: Votar em Pergunta/Resposta

O Diagrama de Sequência descreve a troca dinâmica de mensagens entre os componentes em tempo de execução para o caso de uso **UC01: Votar em Pergunta ou Resposta**.

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário Autenticado
    participant F as Frontend (React UI)
    participant API as API Routes (/api/votos)
    participant S as VotoService
    participant DB as Banco de Dados (SQLite)

    U->>F: Clica no botão de Voto (Upvote/Downvote)
    F->>F: Verifica token JWT de autenticação no LocalStorage
    
    alt Usuário Não Autenticado
        F-->>U: Redireciona para /login e exibe alerta "Faça login para votar"
    else Usuário Autenticado
        F->>API: POST /api/votos { idEntidade, tipoEntidade, tipoVoto } [Bearer JWT]
        API->>S: registrarVoto(idUsuario, idEntidade, tipoEntidade, tipoVoto)
        
        S->>DB: SELECT * FROM perguntas/respostas WHERE id = idEntidade
        DB-->>S: Dados do Conteúdo (contém idUsuarioAutor)
        
        alt Tentativa de Auto-votação (idUsuario == idUsuarioAutor)
            S-->>API: Throw AutoVotacaoException("Não é permitido votar no próprio conteúdo")
            API-->>F: HTTP 403 Forbidden { error: "Auto-votação proibida" }
            F-->>U: Exibe mensagem de erro e reverte ícone
        else Votação Permitida
            S->>DB: SELECT * FROM votos WHERE idUsuario = x AND idEntidade = y
            
            alt Voto Inexistente (Novo Voto)
                S->>DB: INSERT INTO votos (idUsuario, idEntidade, tipoEntidade, tipoVoto)
                S->>DB: UPDATE perguntas/respostas SET saldoVotos = saldoVotos + tipoVoto
                DB-->>S: Sucesso (1 row updated)
                S-->>API: Return { saldoAtualizado, meuVoto: tipoVoto }
                API-->>F: HTTP 201 Created { saldoVotos, tipoVoto }
                F-->>U: Atualiza contador na tela e destaca botão
                
            else Voto Existente com Mesmo Tipo (Cancelamento)
                S->>DB: DELETE FROM votos WHERE id = idVoto
                S->>DB: UPDATE perguntas/respostas SET saldoVotos = saldoVotos - tipoVoto
                DB-->>S: Sucesso
                S-->>API: Return { saldoAtualizado, meuVoto: 0 }
                API-->>F: HTTP 200 OK { saldoVotos, tipoVoto: 0 }
                F-->>U: Remove destaque do botão e atualiza saldo
                
            else Voto Existente com Tipo Diferente (Inversão)
                S->>DB: UPDATE votos SET tipoVoto = novoTipoVoto WHERE id = idVoto
                S->>DB: UPDATE perguntas/respostas SET saldoVotos = saldoVotos + (novoTipoVoto * 2)
                DB-->>S: Sucesso
                S-->>API: Return { saldoAtualizado, meuVoto: novoTipoVoto }
                API-->>F: HTTP 200 OK { saldoVotos, tipoVoto: novoTipoVoto }
                F-->>U: Inverte cor do botão e atualiza saldo (+2 ou -2)
            end
        end
    end
```

---

## 3. Diagrama de Atividades (Activity Diagram)

O Diagrama de Atividades ilustra o fluxo de controle procedural do processo de votação, contemplando as tomadas de decisão, validações e caminhos de exceção.

```mermaid
flowchart TD
    Start([Início: Usuário clica em Votar]) --> CheckAuth{Usuário está autenticado?}
    
    CheckAuth -- Não --> Ex_Auth[Exibir Alerta: Login Necessário]
    Ex_Auth --> RedirectLogin[Redirecionar para /login]
    RedirectLogin --> End_Fail([Fim: Operação Abortada])
    
    CheckAuth -- Sim --> FetchContent[Buscar Autor do Conteúdo no BD]
    FetchContent --> CheckOwner{Usuário é o Autor do Conteúdo?}
    
    CheckOwner -- Sim --> Ex_Owner[Exibir Erro: Auto-votação Não Permitida]
    Ex_Owner --> End_Fail
    
    CheckOwner -- Não --> CheckVotoExistente{Já existe voto deste usuário para esta entidade?}
    
    CheckVotoExistente -- Não --> RegNovo[Inserir Registro na Tabela Voto]
    RegNovo --> CalcNovo[Somar tipoVoto (+1 ou -1) ao saldoVotos]
    CalcNovo --> UpdateDB[Atualizar Banco de Dados]
    
    CheckVotoExistente -- Sim --> CheckTipoVoto{O tipo do novo voto é IGUAL ao voto anterior?}
    
    CheckTipoVoto -- Sim (Cancelamento) --> DeleteVoto[Remover Registro da Tabela Voto]
    DeleteVoto --> CalcCancel[Subtrair tipoVoto do saldoVotos]
    CalcCancel --> UpdateDB
    
    CheckTipoVoto -- Não (Inversão) --> UpdateVoto[Atualizar Registro na Tabela Voto]
    UpdateVoto --> CalcInversao[Ajustar saldoVotos em +2 ou -2]
    CalcInversao --> UpdateDB
    
    UpdateDB --> RespSuccess[Retornar HTTP 200/201 com Novo Saldo]
    RespSuccess --> RenderUI[Atualizar Componente React na Tela]
    RenderUI --> End_Success([Fim: Voto Processado com Sucesso])
```

---

## 4. Diagrama de Estados (State Diagram) — Ciclo de Vida da `Pergunta`

O Diagrama de Estados mapeia as transições e os estados possíveis do objeto de domínio **Pergunta** no sistema, desde a sua criação até o encerramento.

```mermaid
stateDiagram-v2
    [*] --> Criada : Pergunta publicada pelo usuário
    
    state Criada {
        [*] --> SemRespostas
        SemRespostas --> ComRespostas : Primeira resposta enviada
    }
    
    Criada --> Editada : Autor altera título/conteúdo
    Editada --> Criada : Alterações salvas
    
    Criada --> EmDebate : Recebe > 3 respostas ou > 5 votos
    
    EmDebate --> Solucionada : Autor marca uma Resposta como "Melhor Resposta"
    ComRespostas --> Solucionada : Autor marca uma Resposta como "Melhor Resposta"
    
    Solucionada --> Fechada : Inatividade > 30 dias após solução
    
    Criada --> Arquivada : Moderador/Autor remove pergunta
    EmDebate --> Arquivada : Moderador/Autor remove pergunta
    
    Arquivada --> [*]
    Fechada --> [*]
```

---

## 5. Relação com a Arquitetura e Padrões de Projeto

1. **Desacoplamento e Baixo Acoplamento:** O Diagrama de Sequência demonstra a separação clara de responsabilidades entre a camada de apresentação (React UI), o roteamento HTTP e os serviços de domínio do backend.
2. **Consistência do Modelo de Dados:** O Diagrama de Classes estabelece chaves estrangeiras e cardinalidades estritas, garantindo que nenhum voto exista sem associação a um usuário e a uma entidade válida.
3. **Prevenção de Inconsistências:** O Diagrama de Atividades e de Estados garantem que a transição de saldo e regras de negócio (como proibição de auto-votação e imutabilidade de dados arquivados) sejam validadas antes de persistir o estado no banco de dados.
