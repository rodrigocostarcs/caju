# Caju - Modelo Relacional Banco de Ddados
![Diagrama da Aplicação](diagrama.jpg)

## Descrição das Tabelas do Banco de Dados

### 1. Tabela `contas`
**Chave Primária (PK):** `id`

**Campos:**
- `numero_conta`
- `nome_titular`
- `criado_em`

Essa tabela armazena as contas dos usuários.

---

### 2. Tabela `transacoes_food`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `id_conta` → referencia `contas(id)`

**Campos:**
- `tipo`
- `valor`
- `status`
- `criado_em`

Registra as transações relacionadas a alimentação.

---

### 3. Tabela `transacoes_meal`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `id_conta` → referencia `contas(id)`

**Campos:**
- `tipo`
- `valor`
- `status`
- `criado_em`

Registra transações de refeição.

---

### 4. Tabela `transacoes_cash`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `id_conta` → referencia `contas(id)`

**Campos:**
- `tipo`
- `valor`
- `status`
- `criado_em`

Registra transações em dinheiro.

---

### 5. Tabela `carteiras`
**Chave Primária (PK):** `id_carteira`

**Campos:**
- `tipo_beneficio`
- `descricao`
- `criado_em`

Representa os diferentes tipos de carteiras, como alimentação, refeição, etc.

---

### 6. Tabela `contas_carteiras`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `conta_id` → referencia `contas(id)`

**Chave Estrangeira (FK):** `carteira_id` → referencia `carteiras(id_carteira)`

**Campos:**
- `saldo`
- `saldo_reservado`
- `atualizado_em`
- `ativo`
- `criado_em`

Liga as contas dos usuários às suas carteiras, armazenando saldos e status.

---

### 7. Tabela `Extratos`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `id_conta` → referencia `contas(id)`

**Chave Estrangeira (FK):** `id_carteira` → referencia `carteiras(id_carteira)`

**Campos:**
- `debito`
- `credito`
- `data_transacao`
- `descricao`

Armazena o histórico de movimentações.

---

### 8. Tabela `mccs`
**Chave Primária (PK):** `id`

**Campos:**
- `codigo_mcc`
- `nome_estabelecimento`

Representa os códigos e nomes dos estabelecimentos comerciais.

---

### 9. Tabela `carteiras_mccs`
**Chave Primária (PK):** `id`

**Chave Estrangeira (FK):** `carteira_id` → referencia `carteiras(id_carteira)`

**Chave Estrangeira (FK):** `mcc_id` → referencia `mccs(id)`

**Campos:**
- `criado_em`

Relaciona as carteiras aos estabelecimentos (MCCs).

---

### 10. Tabela `estabelecimentos`
**Chave Primária (PK):** `uuid`

**Campos:**
- `nome_estabelecimento`
- `senha_hash`
- `criado_em`

Armazena dados dos estabelecimentos cadastrados.

---

## Resumo das Relações
- `contas` se relaciona com `transacoes_food`, `transacoes_meal`, `transacoes_cash` e `Extratos`.
- `contas_carteiras` liga `contas` e `carteiras`.
- `Extratos` também se relaciona com `carteiras`.
- `carteiras_mccs` relaciona `carteiras` com `mccs`.
- `estabelecimentos` armazena informações dos estabelecimentos.


# Caju - Guia de Configuração e Execução

## Configuração do Ambiente MySQL no Ubuntu

### Atualize os pacotes:
```bash
sudo apt update
```

### Instale o MySQL Server:
```bash
sudo apt install mysql-server
```

### Inicie e habilite o MySQL:
```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### Acesse o MySQL:
```bash
mysql -u root -p
```

### Execute os comandos dos arquivos `comandos.sql` e `inserts.sql` para configurar o banco de dados.

---

## Instalação do Elixir e Erlang no Ubuntu

### Atualize os pacotes:
```bash
sudo apt update
```

### Instale Erlang e Elixir:
```bash
sudo apt install erlang elixir
```

### Verifique a versão:
```bash
elixir --version
```

### Instale as ferramentas do Phoenix Framework:
```bash
mix archive.install hex phx_new 1.7.0
```

---

## Configuração do Projeto

### Clone o repositório e acesse a pasta do projeto:
```bash
cd caju
```

### Instale as dependências:
```bash
mix deps.get
```

### Configure o arquivo `config/dev.exs` com as credenciais do banco de dados:
```elixir
username: "root",
password: "senha",
hostname: "localhost",
database: "caju_dev",
stacktrace: true,
show_sensitive_data_on_connection_error: true,
pool_size: 10
```

---

## Execução do Projeto

### Execute o servidor Phoenix:
```bash
mix phx.server
```

A API estará disponível em: [http://127.0.0.1:4000](http://127.0.0.1:4000)

---

## Testando a API com Postman

- Utilize o Postman para testar as requisições REST.
- O arquivo `caju.postman_collection.json`, localizado na raiz do projeto, contém todas as requisições configuradas.

### Fluxo de Teste:
1. Realizar login com os dados do estabelecimento (já inseridos no banco de dados pelos scripts).
2. A autenticação gera um token JWT.
3. O token JWT deve ser utilizado nas demais requisições.
4. O tempo de expiração do token está definido como 1 minuto para fins de teste.
5. Cada requisição no arquivo Postman contém uma situação de teste específica, conforme o proposto.

### Dica:
Sempre verifique se o banco de dados está ativo antes de iniciar o servidor Phoenix.

### L4. Questão aberta

```Para garantir que apenas uma transação por conta fosse processada em um determinado momento, em uma transação síncrona, poderia ser utilizado um sistema de reserva de saldo, no qual a transação chegaria e, no primeiro passo, faria a reserva do saldo com base no valor da transação. Em seguida, seriam realizadas todas as validações necessárias e, no momento do lançamento, o saldo reservado seria descontado do saldo real e retirado do saldo reservado, sempre validando se o saldo reservado é maior ou igual ao valor da transação no momento do lançamento. No desafio proposto, implementei essa estrutura e também adicionei as operações dentro de uma transaction do Ecto, para garantir que, caso ocorra algum erro, seja dado rollback em todas as operações, incluindo na reserva do saldo, no lançamento na tabela de transações e no extrato.
```