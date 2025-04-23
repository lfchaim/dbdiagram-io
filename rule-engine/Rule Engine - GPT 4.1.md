Pergunta: Desenvolva uma modelagem de dados para a criação de um \"Motor de Regras\", atendendo às seguintes especificações:

1. Proposta da Solução  
   - Descreva o objetivo do motor de regras e seus principais componentes.  
   - Explique como as condições lógicas (AND, OR, XOR, NOT) serão modeladas e avaliadas.  
   - Detalhe como a flexibilidade e expansibilidade do motor poderão ser mantidas.  
   - Especifique possíveis integrações com sistemas externos (caso aplicável).  

2. Documentação Básica  
   a. Definição das entidades principais (como Regra, Condição, Grupo de Regras, Ação).  
   b. Relacionamentos entre entidades.  
   c. Fluxo básico de execução da avaliação das regras.  
   d. Exemplo de cadastro de uma regra composta por múltiplas condições lógicas (AND, OR, XOR).  
   e. Explicação das opções para customização de operadores lógicos.  

3. DDL das Tabelas Necessárias  
   - Defina o script SQL (DDL) para criação das tabelas essenciais, incluindo as colunas, tipos de dados, chaves primárias e estrangeiras.  
   - Implemente tabelas que representem:  
       3.1. Regras (identificação, status, descrição)  
       3.2. Condições (tipo, valor de comparação, operador lógico)  
       3.3. Relacionamento entre regras e condições  
       3.4. Ações disparadas pelo motor  
   - Inclua exemplos de inserção de dados (opcional).  

4. Requisitos Adicionais  
   - Estruture a modelagem para suportar aninhamento/multinível de condições e regras.  
   - Garanta normalização suficiente para flexibilidade e performance.  
   - Considere cenários de versionamento ou histórico de regras.

Entregue a modelagem com os tópicos acima, utilizando linguagem clara e objetiva, mantendo a padronização de nomenclatura e boas práticas de modelagem relacional.
Resposta: ## 1. Proposta da Solução

### Objetivo do Motor de Regras
O objetivo do **Motor de Regras** é permitir a definição dinâmica, gerenciamento e execução de regras de negócio de maneira flexível, desacoplada da lógica principal da aplicação. Um motor de regras avalia condições pré-definidas e executa ações conforme critérios especificados.

### Principais Componentes

- **Regra**: Unidade lógica que define quando uma ou mais ações devem ser disparadas.
- **Condição**: Expressão avaliada (verdadeira/falsa) para definir o resultado da regra.
- **Grupo de Regras**: Organização hierárquica de regras para estruturação e reaproveitamento.
- **Ação**: Consequência da avaliação de uma regra (ex.: acionar workflow, enviar e-mail).

### Modelagem e Avaliação das Condições Lógicas

- **Condições lógicas** como AND, OR, XOR e NOT são modeladas em entidades associativas e via uma estrutura de árvore (grafo), permitindo o aninhamento e multiníveis.
- Cada “nó” da árvore pode ser um operador lógico que agrupa filhos (ou subcondições) ou uma condição atômica (uma expressão simples).
- **Avaliação recursiva**: a árvore é percorrida e cada nó operador avalia seus filhos, suportando complexidade arbitrária.

### Flexibilidade e Expansibilidade

- **Tabelas normalizadas**, relacionamentos polimórficos e uso de tipos genéricos suportam novas condições, operadores e ações (princípio Open/Closed).
- **Versionamento** e histórico podem ser incluídos para rastreabilidade.
- O modelo permite a inclusão de novos tipos de condições e integração de ações customizadas.

### Integração com Sistemas Externos

- Utilização de “webhooks”, filas ou APIs para acionar sistemas externos (por exemplo, disparar uma action via REST).
- Possibilidade de cadastrar ações integradoras (ações de chamada externa).

---

## 2. Documentação Básica

### a. Definição das Entidades Principais

| Entidade    | Descrição                                                         |
|-------------|-------------------------------------------------------------------|
| Regra       | Conjunto de condições; define quando uma ação deve ocorrer        |
| Condição    | Expressão lógica (ex.: campo > valor)                             |
| GrupoRegra  | Estrutura agrupar e versionar regras                              |
| Ação        | Procedimento disparado ao validar a regra                         |
| NóCondição  | Estrutura para aninhamento de condições e operadores lógicos      |

---

### b. Relacionamentos entre Entidades

- **GrupoRegra** contém várias **Regras**
- **Regra** se relaciona com um ou mais **NóCondição**
- **NóCondição** pode ser do tipo operador lógico (AND, OR, XOR, NOT) ou condição atômica
- **Regra** possui uma ou mais **Ações**
- **NóCondição** pode referenciar outros **NóCondição** (nó pai/filho → aninhamento multinível)

---

### c. Fluxo Básico de Execução

1. Um evento de negócio dispara a avaliação do grupo de regras aplicável.
2. Para cada regra ativa, o motor percorre a árvore de **NóCondições** associada, avaliando recursivamente operadores e condições.
3. Caso a expressão principal da **Regra** seja verdadeira, as ações ligadas a ela são executadas.

---

### d. Exemplo de Cadastro de Regra Composta

**Regra:** \"Se (Idade > 18 **AND** (Saldo > 1000 **OR** Status='VIP'))\"

- Nó raiz: AND
  - Filho 1: Condição (Idade > 18)
  - Filho 2: OR
    - Condição (Saldo > 1000)
    - Condição (Status = 'VIP')

---

### e. Opções para Customização de Operadores Lógicos

- Cadastro de tipos de operadores na tabela **OperatorType**.
- Possibilidade de criar novos operadores customizados, como **NAND** ou operadores exclusivos para tipos de dados específicos.
- Parâmetros customizáveis nos operadores, ajustando a lógica sem alteração de código.

---

## 3. DDL das Tabelas Necessárias

### 3.1. Tabela de Regras

```sql
CREATE TABLE Rule (
    rule_id           INT PRIMARY KEY AUTO_INCREMENT,
    group_rule_id     INT,
    version           INT NOT NULL,
    name              VARCHAR(100) NOT NULL,
    description       VARCHAR(255),
    status            VARCHAR(20) NOT NULL, -- ativa, inativa, rascunho
    root_node_id      INT, -- nó raiz da árvore de condições
    created_at        DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at        DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (group_rule_id) REFERENCES RuleGroup(group_rule_id)
);
```

### 3.2. Tabela de Grupos de Regras

```sql
CREATE TABLE RuleGroup (
    group_rule_id     INT PRIMARY KEY AUTO_INCREMENT,
    name              VARCHAR(100) NOT NULL,
    description       VARCHAR(255)
);
```

### 3.3. Tabela de Condições

```sql
CREATE TABLE Condition (
    condition_id      INT PRIMARY KEY AUTO_INCREMENT,
    field_name        VARCHAR(100) NOT NULL,
    operator          VARCHAR(20) NOT NULL, -- Exemplo: '=', '>', '<', 'contains'
    comparison_value  VARCHAR(255) NOT NULL
);
```

### 3.4. Tabelas para Modelagem do Aninhamento Lógico (Nó de Condição)

```sql
CREATE TABLE ConditionNode (
    node_id           INT PRIMARY KEY AUTO_INCREMENT,
    parent_node_id    INT,
    node_type         VARCHAR(20) NOT NULL, -- 'OPERATOR' ou 'CONDITION'
    operator_type     VARCHAR(10), -- Exemplo: 'AND', 'OR', 'XOR', 'NOT' (para tipo 'OPERATOR')
    condition_id      INT, -- apenas se node_type = 'CONDITION'
    rule_id           INT NOT NULL,
    position          INT, -- para ordem entre filhos
    FOREIGN KEY (parent_node_id) REFERENCES ConditionNode(node_id),
    FOREIGN KEY (condition_id) REFERENCES Condition(condition_id),
    FOREIGN KEY (rule_id) REFERENCES Rule(rule_id)
);
```

### 3.5. Tabela de Ações

```sql
CREATE TABLE Action (
    action_id         INT PRIMARY KEY AUTO_INCREMENT,
    rule_id           INT NOT NULL,
    action_type       VARCHAR(50) NOT NULL, -- Exemplo: 'EMAIL', 'WEBHOOK', 'LOG'
    parameters        TEXT, -- JSON com parâmetros dinâmicos
    status            VARCHAR(20),
    created_at        DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (rule_id) REFERENCES Rule(rule_id)
);
```

---

### Exemplos de Inserção

```sql
-- Grupo de regras
INSERT INTO RuleGroup (name, description) VALUES ('Regras de Clientes', 'Regras relacionadas a clientes');

-- Condições atômicas
INSERT INTO Condition (field_name, operator, comparison_value) VALUES ('idade', '>', '18');
INSERT INTO Condition (field_name, operator, comparison_value) VALUES ('saldo', '>', '1000');
INSERT INTO Condition (field_name, operator, comparison_value) VALUES ('status', '=', 'VIP');

-- Regra
INSERT INTO Rule (group_rule_id, version, name, description, status) VALUES (1, 1, 'Maioridade e Saldo/Status', 'Regra composta por múltiplas condições', 'ativa');

-- Estrutura dos nós da regra:
-- Supondo IDs gerados automaticamente para as condições: 1 (idade), 2 (saldo), 3 (status)
-- Nó raiz (AND): node_id=1, parent_node_id=NULL
INSERT INTO ConditionNode (parent_node_id, node_type, operator_type, rule_id, position)
VALUES (NULL, 'OPERATOR', 'AND', 1, 1);
-- Nó OR filho do AND: node_id=2, parent_node_id=1
INSERT INTO ConditionNode (parent_node_id, node_type, operator_type, rule_id, position)
VALUES (1, 'OPERATOR', 'OR', 1, 2);
-- Condição idade > 18 filho do AND: node_id=3, parent_node_id=1, condition_id=1
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (1, 'CONDITION', 1, 1, 1);
-- Condição saldo > 1000 filho do OR: node_id=4, parent_node_id=2, condition_id=2
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (2, 'CONDITION', 2, 1, 1);
-- Condição status = 'VIP' filho do OR: node_id=5, parent_node_id=2, condition_id=3
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (2, 'CONDITION', 3, 1, 2);

-- Ação da regra
INSERT INTO Action (rule_id, action_type, parameters, status) VALUES (1, 'EMAIL', '{\"to\":\"admin@acme.com\",\"template\":\"cliente_aceito\"}', 'ativa');
```

---

## 4. Requisitos Adicionais

### Suporte a Aninhamento e Multinível
- A tabela `ConditionNode` usa relacionamento hierárquico (parent/child), permitindo quantos níveis forem necessários de operadores e subcondições.
- Cada nó pode ser um operador lógico (juntando subnós) ou uma condição atômica, permitindo construção livre de expressões complexas.

### Normalização e Performance
- Separação clara das entidades (`Rule`, `Condition`, `Action`, `ConditionNode`, `RuleGroup`).
- Uso de chaves estrangeiras para integridade, possibilitando fácil expansão sem
Pergunta: Por favor, continue gerando a sua resposta.
Resposta: Claro! Seguindo do ponto da normalização e performance:  

---

### Normalização e Performance

- **Normalização:**  
  - Estrutura modular permite reutilização de condições e operadores.
  - Separação de regras, grupos, condições e ações evita duplicidade e facilita manutenção.
- **Performance:**  
  - Permite criação de índices nas colunas mais consultadas (`status`, `rule_id`, `group_rule_id`).
  - Suporte à consulta eficiente por regras ativas, versão mais recente ou por grupo.

---

### Versionamento/Historicidade de Regras

- A coluna `version` em `Rule` permite manter versões diferentes de uma mesma regra.
- Para controle de histórico de alterações, pode-se expandir com:
  ```sql
  CREATE TABLE RuleHistory (
      history_id        INT PRIMARY KEY AUTO_INCREMENT,
      rule_id           INT NOT NULL,
      changed_by        VARCHAR(50) NOT NULL,
      changed_at        DATETIME DEFAULT CURRENT_TIMESTAMP,
      old_data          TEXT, -- JSON com snapshot da regra antes da alteração
      new_data          TEXT, -- JSON com snapshot da regra após a alteração
      action            VARCHAR(30) -- 'CREATE', 'UPDATE', 'DELETE'
  );
  ```
- Cada alteração relevante em `Rule` (ou em seus nós/ações) deve registrar a versão anterior, promovendo rastreabilidade e rollback em caso de inconsistências.

---

## Resumo Visual das Entidades e Relacionamentos

```
[RuleGroup] 1 -- N [Rule] 1 -- N [ConditionNode]
                                      |
                                      N
                                  [Condition]
[Rule] 1 -- N [Action]
[ConditionNode] N -- 1 [ConditionNode] (aninhamento)
```

---

## Conclusão/Resumo dos Pontos-Chave

- **Modelagem proposta é flexível e expansível**, suportando operadores lógicos aninhados e estrutura multinível.
- Suporte integrado para versionamento, histórico e integração com sistemas externos.
- Organizada por entidades bem definidas, separando conceitos de regra, condição, ação, agrupamento e nós hierárquicos.
- O fluxo de avaliação baseada em árvore permite expressar qualquer combinação lógica possível.
- Facilita manutenção, monitoramento e adaptação rápida a mudanças de negócio.

---

**Caso precise de exemplos de consultas SQL para avaliação das regras ou de mais detalhes sobre como realizar integrações específicas, é só pedir!**
Pergunta: O DDL acima é compatível com qual SGBD?
Resposta: Ótima pergunta! O DDL fornecido acima segue o **padrão SQL básico** e é compatível principalmente com os seguintes bancos relacionais:

- **MySQL/MariaDB:**  
  Os exemplos (`AUTO_INCREMENT`, tipos de dados como `VARCHAR`, `INT`, `DATETIME`, etc.) estão totalmente alinhados com o dialeto MySQL.

- **PostgreSQL:**  
  Pequenas adaptações são necessárias:
  - Em vez de `AUTO_INCREMENT`, use `SERIAL` ou o comando `GENERATED ALWAYS AS IDENTITY`.
  - As expressões de data/hora (`CURRENT_TIMESTAMP`) também são compatíveis.
  - Aspas duplas são reservadas para nomes de colunas/tabelas no PostgreSQL; valores de texto usam aspas simples, como no padrão SQL.

- **SQL Server:**  
  Troque `AUTO_INCREMENT` por `IDENTITY(1,1)`.
  Tipos de dados como `VARCHAR`, `INT` e `DATETIME` estão de acordo, mas o SQL Server utiliza `GETDATE()` ao invés de `CURRENT_TIMESTAMP`.

---

### Exemplos de Ajustes

#### Para **PostgreSQL**:
```sql
CREATE TABLE Rule (
    rule_id SERIAL PRIMARY KEY,
    ...
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Para **SQL Server**:
```sql
CREATE TABLE Rule (
    rule_id INT IDENTITY(1,1) PRIMARY KEY,
    ...
    created_at DATETIME DEFAULT GETDATE(),
    updated_at DATETIME DEFAULT GETDATE()
);
```

---

### Resumindo:
- O DDL fornecido está pronto para uso em **MySQL/MariaDB**.
- Com pequenas adaptações, pode ser usado em **PostgreSQL** ou **SQL Server**.
- Caso deseje a versão integral para um SGBD específico, me informe qual é o seu banco de preferência que posso gerar sem adaptações necessárias!

Se precisar da versão exata para PostgreSQL, SQL Server, Oracle ou outro SGBD, é só pedir!
Pergunta: Regerar o conteúdo da proposta
Resposta: Claro! Segue a modelagem da proposta para um Motor de Regras, contemplando todos os tópicos solicitados, com padronização e clareza.

---

## 1. Proposta da Solução

### Objetivo do Motor de Regras
O Motor de Regras tem por objetivo viabilizar a definição, organização, versionamento e execução de regras de negócio de maneira flexível e escalável, permitindo rápida adaptação de lógica sem necessidade de alteração de código-fonte.

### Principais Componentes

- **Regra:** Define um conjunto de condições que, quando satisfeitas, disparam uma ou mais ações.
- **Condição:** Critério lógico-atômico ou grupo de condições, avaliado como verdadeiro ou falso.
- **Grupo de Regras:** Permite categorizar e versionar regras.
- **Ação:** Atividade executada quando uma regra é validada, podendo integrar com sistemas externos.

### Modelagem e Avaliação das Condições Lógicas

- Condições lógicas (AND, OR, XOR, NOT) são representadas por uma estrutura em árvore (nós de condição), permitindo aninhamento e construção de expressões complexas.
- Cada nó pode ser um operador lógico (agrupador) ou uma condição atômica.
- A avaliação recursiva da árvore simula o processamento lógico dos operadores, permitindo aninhar quantos níveis forem necessários.

### Flexibilidade e Expansibilidade

- Estrutura modular, com entidades bem separadas e normalizadas, para que novos operadores, tipos de condição e ações possam ser adicionados sem impacto estrutural.
- Versionamento de regras permite ajustes e auditoria sem perda de histórico.
- Cadastro dinâmico de operadores, condições e ações.

### Integração com Sistemas Externos

- Permite associar ações que disparam integrações externas (via chamadas HTTP, mensagens em fila, etc).
- Estrutura dos parâmetros da ação é flexível para comportar integrações diversas.

---

## 2. Documentação Básica

### a. Definição das Entidades Principais

| Entidade       | Descrição                                                                    |
|----------------|------------------------------------------------------------------------------|
| Grupo de Regras| Agrupamento lógico e versionamento das regras                                |
| Regra          | Regra de negócio composta por uma (ou mais) condição(ões) lógico(s)          |
| Condição       | Expressão atômica (campo, operador, valor)                                   |
| Nó de Condição | Nó na árvore lógica: operador lógico (AND, OR, etc) ou referência a condição |
| Ação           | Procedimento disparado ao validar uma regra                                  |

### b. Relacionamentos entre Entidades

- **Grupo de Regras** possui várias **Regras**.
- **Regra** possui uma árvore de **Nós de Condição** (raiz aponta para o nó principal).
- **Nó de Condição** pode ser operador lógico (contendo outros nós) ou condição atômica.
- **Regra** possui uma ou mais **Ações** associadas.
- **Nó de Condição** pode estar aninhado a partir de outro nó (referência ao nó pai).

### c. Fluxo Básico de Execução

1. Um evento no sistema aciona a avaliação de determinado grupo de regras.
2. Para cada regra ativa, a árvore de condições associada é percorrida recursivamente.
3. Caso toda a expressão lógica avaliada seja verdadeira, as ações vinculadas à regra são executadas.
4. Ações podem ser locais (internas) ou integrar sistemas externos.

### d. Exemplo de Cadastro de Regra Composta

**Regra:**  
Se (idade > 18) **AND** ((saldo > 1000) **OR** (status = 'VIP')), então enviar notificação.

- [AND]
  - [CONDIÇÃO] idade > 18
  - [OR]
    - [CONDIÇÃO] saldo > 1000
    - [CONDIÇÃO] status = 'VIP'

### e. Opções para Customização de Operadores Lógicos

- Operadores podem ser cadastrados em tabela própria, possibilitando extensão ou customização por tipo de dado.
- Parâmetros adicionais por operador, caso necessário (ex: operador temporizado, operador de limite mínimo de condições verdadeiras, etc).

---

## 3. DDL das Tabelas Necessárias

_Adaptado para MySQL/_MariaDB com indicações para ajustes em outros bancos, se necessário._

### 3.1. Grupo de Regras

```sql
CREATE TABLE RuleGroup (
    group_rule_id   INT PRIMARY KEY AUTO_INCREMENT,
    name            VARCHAR(100) NOT NULL,
    description     VARCHAR(255)
);
```

### 3.2. Regras

```sql
CREATE TABLE Rule (
    rule_id         INT PRIMARY KEY AUTO_INCREMENT,
    group_rule_id   INT NOT NULL,
    version         INT NOT NULL,
    name            VARCHAR(100) NOT NULL,
    description     VARCHAR(255),
    status          VARCHAR(20) NOT NULL, -- exemplo: ativa, inativa
    root_node_id    INT, -- aponta para nó raiz da expressão lógica
    created_at      DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (group_rule_id) REFERENCES RuleGroup(group_rule_id)
);
```

### 3.3. Condições

```sql
CREATE TABLE Condition (
    condition_id        INT PRIMARY KEY AUTO_INCREMENT,
    field_name          VARCHAR(100) NOT NULL,
    operator_symbol     VARCHAR(20) NOT NULL, -- Ex: '=', '>', '<', etc.
    comparison_value    VARCHAR(255) NOT NULL
);
```

### 3.4. Nós de Condição (Estrutura de árvore lógica)

```sql
CREATE TABLE ConditionNode (
    node_id         INT PRIMARY KEY AUTO_INCREMENT,
    parent_node_id  INT,
    node_type       VARCHAR(20) NOT NULL, -- 'OPERATOR' ou 'CONDITION'
    operator_type   VARCHAR(10), -- 'AND', 'OR', 'XOR', 'NOT' (para node_type 'OPERATOR')
    condition_id    INT, -- para node_type 'CONDITION'
    rule_id         INT NOT NULL,
    position        INT, -- ordem entre filhos
    FOREIGN KEY (parent_node_id) REFERENCES ConditionNode(node_id),
    FOREIGN KEY (condition_id) REFERENCES Condition(condition_id),
    FOREIGN KEY (rule_id) REFERENCES Rule(rule_id)
);
```

### 3.5. Ações

```sql
CREATE TABLE Action (
    action_id       INT PRIMARY KEY AUTO_INCREMENT,
    rule_id         INT NOT NULL,
    action_type     VARCHAR(50) NOT NULL, -- exemplo: 'EMAIL', 'WEBHOOK'
    parameters      TEXT, -- parâmetros customizáveis (JSON, por exemplo)
    status          VARCHAR(20),
    created_at      DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (rule_id) REFERENCES Rule(rule_id)
);
```

### 3.6. Histórico de Versão das Regras (Opcional)

```sql
CREATE TABLE RuleHistory (
    history_id          INT PRIMARY KEY AUTO_INCREMENT,
    rule_id             INT NOT NULL,
    changed_by          VARCHAR(50) NOT NULL,
    changed_at          DATETIME DEFAULT CURRENT_TIMESTAMP,
    old_data            TEXT, -- snapshot JSON da versão anterior
    new_data            TEXT, -- snapshot JSON da versão nova
    action              VARCHAR(30), -- CREATE, UPDATE, DELETE
    FOREIGN KEY (rule_id) REFERENCES Rule(rule_id)
);
```

---

### Exemplo de Inserção

```sql
-- Grupo
INSERT INTO RuleGroup (name, description) VALUES ('Clientes', 'Regras para clientes');

-- Condições
INSERT INTO Condition (field_name, operator_symbol, comparison_value) VALUES ('idade', '>', '18');
INSERT INTO Condition (field_name, operator_symbol, comparison_value) VALUES ('saldo', '>', '1000');
INSERT INTO Condition (field_name, operator_symbol, comparison_value) VALUES ('status', '=', 'VIP');

-- Regra
INSERT INTO Rule (group_rule_id, version, name, description, status) VALUES (1, 1, 'Validação Cliente Adulto com Saldo/Status', 'Regra composta', 'ativa');

-- Nó AND (raiz)
INSERT INTO ConditionNode (parent_node_id, node_type, operator_type, rule_id, position)
VALUES (NULL, 'OPERATOR', 'AND', 1, 1);

-- Nó OR (filho do AND)
INSERT INTO ConditionNode (parent_node_id, node_type, operator_type, rule_id, position)
VALUES (1, 'OPERATOR', 'OR', 1, 2);

-- Condição idade > 18 (filho do AND)
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (1, 'CONDITION', 1, 1, 1);

-- Condição saldo > 1000 (filho do OR)
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (2, 'CONDITION', 2, 1, 1);

-- Condição status = 'VIP' (filho do OR)
INSERT INTO ConditionNode (parent_node_id, node_type, condition_id, rule_id, position)
VALUES (2, 'CONDITION', 3, 1, 2);

-- Ação
INSERT INTO Action (rule_id, action_type, parameters, status) VALUES (1, 'EMAIL', '{"to":"admin@empresa.com"}', 'ativa');
```

---

## 4. Requisitos Adicionais

- **Aninhamento e multinível de condições**: Estrutura em árvore, via ConditionNode, permite qualquer grau de complexidade na lógica.
- **Normalização**: Entidades separadas sem redundância, apoiando performance e facilidade de manutenção.
- **Versionamento e Histórico**: Campos `version` e tabela `RuleHistory` para auditoria de alterações e reversão, se necessário.

---

## Resumo

A modelagem proposta permite:

- **Estrutura flexível e expansível** — adaptável a novas regras, condições e integrações sem alterações estruturais.
- **Desempenho e normalização** — design eficiente para consultas e manutenção.
- **Versionamento e histórico** — total rastreabilidade das alterações de regras.
- **Modelagem lógica robusta** — operadores lógicos aninhados e multiníveis para atender regras complexas de negócios.

Se precisar da versão ajustada para outro banco de dados ou de consultas SQL para execução do motor, é só me avisar!
