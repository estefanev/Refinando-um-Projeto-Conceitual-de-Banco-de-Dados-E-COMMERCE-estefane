Modelo de Cliente, Pagamento e Entrega

1️⃣ Objetivo

O objetivo deste documento é refinar o modelo de dados de um sistema de vendas, incluindo:

Diferenciação entre clientes PJ (Pessoa Jurídica) e PF (Pessoa Física), garantindo que uma conta não possa ter os dois tipos simultaneamente.

Registro de formas de pagamento múltiplas por cliente.

Registro e controle de entregas, incluindo status e código de rastreio.

O resultado será um modelo que atende requisitos de negócio, mantém integridade de dados e facilita consultas e relatórios.

2️⃣ Modelagem de Clientes
Entidades e Atributos
Entidade	Atributos Principais	Observações
Cliente	id_cliente (PK)	Identificador único da conta
	tipo_cliente	Enum: 'PF' ou 'PJ' – não pode ter os dois
	nome	Nome da pessoa física ou razão social da PJ
	cpf_cnpj	CPF se PF, CNPJ se PJ
	email	Contato principal
	telefone	Contato secundário opcional
Regras de Integridade

tipo_cliente deve ser obrigatório e único por conta.

Se tipo_cliente = PF, então cpf_cnpj deve conter um CPF válido.

Se tipo_cliente = PJ, então cpf_cnpj deve conter um CNPJ válido.

Não permitir que a mesma conta tenha informações de PF e PJ ao mesmo tempo.

3️⃣ Modelagem de Pagamentos
Entidades e Atributos
Entidade	Atributos Principais	Observações
FormaPagamento	id_pagamento (PK)	Identificador único da forma de pagamento
	id_cliente (FK)	Referência ao cliente
	tipo_pagamento	Enum: 'Cartão', 'Boleto', 'Pix', 'Transferência'
	detalhes	Número do cartão, banco, etc.
	data_cadastro	Data em que a forma de pagamento foi cadastrada
Regras de Integridade

Um cliente pode ter múltiplas formas de pagamento.

Cada forma de pagamento deve estar linkada a apenas um cliente.

Não permitir duplicidade de forma de pagamento idêntica para o mesmo cliente (ex.: mesmo cartão registrado duas vezes).

4️⃣ Modelagem de Entregas
Entidades e Atributos
Entidade	Atributos Principais	Observações
Entrega	id_entrega (PK)	Identificador único da entrega
	id_pedido (FK)	Referência ao pedido relacionado
	status	Enum: 'Pendente', 'Em transporte', 'Entregue', 'Cancelado', 'Devolvido'
	codigo_rastreio	Código fornecido pela transportadora
	data_previsao	Data prevista de entrega
	data_entrega	Data em que a entrega foi realizada (se aplicável)
Regras de Integridade

Cada entrega deve estar associada a um pedido válido.

status deve ser atualizado conforme o progresso da entrega.

codigo_rastreio é obrigatório para todas as entregas que saíram para transporte.

5️⃣ Relacionamentos

Cliente → FormaPagamento: 1:N
Um cliente pode ter várias formas de pagamento.

Pedido → Entrega: 1:N
Um pedido pode ter várias entregas (ex.: pedidos fracionados).

Cliente → Pedido: 1:N
Um cliente pode ter vários pedidos registrados.

6️⃣ Exemplo de Estrutura em SQL
-- Tabela Cliente
CREATE TABLE Cliente (
    id_cliente INT PRIMARY KEY,
    tipo_cliente VARCHAR(2) CHECK (tipo_cliente IN ('PF','PJ')),
    nome VARCHAR(255) NOT NULL,
    cpf_cnpj VARCHAR(20) NOT NULL UNIQUE,
    email VARCHAR(255),
    telefone VARCHAR(20)
);

-- Tabela FormaPagamento
CREATE TABLE FormaPagamento (
    id_pagamento INT PRIMARY KEY,
    id_cliente INT,
    tipo_pagamento VARCHAR(50),
    detalhes TEXT,
    data_cadastro DATE,
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- Tabela Entrega
    id_entrega INT PRIMARY KEY,
    id_pedido INT,
    status VARCHAR(50) CHECK (status IN ('Pendente','Em transporte','Entregue','Cancelado','Devolvido')),
    codigo_rastreio VARCHAR(50),
    data_previsao DATE,
    data_entrega DATE,
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido)
);

7️⃣ Observações Finais

A separação de cliente, pagamento e entrega garante flexibilidade para adicionar novas funcionalidades futuramente (como múltiplos pedidos por cliente, promoções, históricos de pagamento).

É importante manter validações de CPF/CNPJ e integridade referencial para evitar inconsistências.

Para um sistema real, recomenda-se auditoria de alterações em pagamentos e entregas, garantindo rastreabilidade.
