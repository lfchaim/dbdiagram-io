# Prompt:
Desenvolva uma modelagem de dados para a criação de um "Motor de Regras", atendendo às seguintes especificações:

1. Proposta da Solução
* Descreva o objetivo do motor de regras e seus principais componentes.
* Explique como as condições lógicas (AND, OR, XOR, NOT) serão modeladas e avaliadas.
* Detalhe como a flexibilidade e expansibilidade do motor poderão ser mantidas.
* Especifique possíveis integrações com sistemas externos (caso aplicável).

2. Documentação Básica
a. Definição das entidades principais (como Regra, Condição, Grupo de Regras, Ação).
b. Relacionamentos entre entidades.
c. Fluxo básico de execução da avaliação das regras.
d. Exemplo de cadastro de uma regra composta por múltiplas condições lógicas (AND, OR, XOR).
e. Explicação das opções para customização de operadores lógicos.

3. DDL das Tabelas Necessárias
* Defina o script SQL (DDL) para criação das tabelas essenciais, incluindo as colunas, tipos de dados, chaves primárias e estrangeiras.
* Implemente tabelas que representem:
  3.1. Regras (identificação, status, descrição)
  3.2. Condições (tipo, valor de comparação, operador lógico)
  3.3. Relacionamento entre regras e condições
  3.4. Ações disparadas pelo motor
* Inclua exemplos de inserção de dados (opcional).

4. Requisitos Adicionais
  * Estruture a modelagem para suportar aninhamento/multinível de condições e regras.
  * Garanta normalização suficiente para flexibilidade e performance.
  * Considere cenários de versionamento ou histórico de regras.

Entregue a modelagem com os tópicos acima, utilizando linguagem clara e objetiva, mantendo a padronização de nomenclatura e boas práticas de modelagem relacional.

# AI:


# DBdiagram
Table "Rule" {
  "rule_id" INT [pk, increment]
  "group_rule_id" INT
  "version" INT [not null]
  "name" VARCHAR(100) [not null]
  "description" VARCHAR(255)
  "status" VARCHAR(20) [not null]
  "root_node_id" INT
  "created_at" DATETIME [default: `CURRENT_TIMESTAMP`]
  "updated_at" DATETIME [default: `CURRENT_TIMESTAMP`]
}

Table "RuleGroup" {
  "group_rule_id" INT [pk, increment]
  "name" VARCHAR(100) [not null]
  "description" VARCHAR(255)
}

Table "Condition" {
  "condition_id" INT [pk, increment]
  "field_name" VARCHAR(100) [not null]
  "operator" VARCHAR(20) [not null]
  "comparison_value" VARCHAR(255) [not null]
}

Table "ConditionNode" {
  "node_id" INT [pk, increment]
  "parent_node_id" INT
  "node_type" VARCHAR(20) [not null]
  "operator_type" VARCHAR(10)
  "condition_id" INT
  "rule_id" INT [not null]
  "position" INT
}

Table "Action" {
  "action_id" INT [pk, increment]
  "rule_id" INT [not null]
  "action_type" VARCHAR(50) [not null]
  "parameters" TEXT
  "status" VARCHAR(20)
  "created_at" DATETIME [default: `CURRENT_TIMESTAMP`]
}

Ref:"RuleGroup"."group_rule_id" < "Rule"."group_rule_id"

Ref:"ConditionNode"."node_id" < "ConditionNode"."parent_node_id"

Ref:"Condition"."condition_id" < "ConditionNode"."condition_id"

Ref:"Rule"."rule_id" < "ConditionNode"."rule_id"

Ref:"Rule"."rule_id" < "Action"."rule_id"
