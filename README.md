<div align="center">

# API EM PYTHON COM FLASK

### Documentação técnica — SAEP 2026

Desenvolvimento de uma API REST com Flask e PostgreSQL,
autenticação JWT e operações CRUD.

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)

Documentação prática de desenvolvimento de uma API REST com Python e Flask, integrada ao PostgreSQL. Utiliza como referência um mini-mundo proposto no contexto educacional, abordando autenticação JWT, operações CRUD, relacionamentos entre tabelas, consultas SQL e testes com Thunder Client.

</div>



## SUMÁRIO

1. [Configuração do ambiente](#1-configuração-do-ambiente)
2. [Estrutura do projeto](#2-estrutura-do-projeto)
3. [Conexão com o banco de dados](#3-conexão-com-o-banco-de-dados)
4. [Configuração da aplicação — app.py](#4-configuração-da-aplicação--apppy)
5. [Autenticação JWT](#5-autenticação-jwt)
6. [Execução da aplicação](#6-execução-da-aplicação)
7. [Testes iniciais — app.py](#7-testes-iniciais--apppy)
8. [Autenticação — Login](#8-autenticação--login)
9. [Função — Listar](#9-função--listar)
10. [Função — Buscar por termo](#10-função--buscar-por-termo)
11. [Função — Cadastro](#11-função--cadastro)
12. [Função — Edição](#12-função--edição)
13. [Função — Exclusão](#13-função--exclusão)
14. [Função — Listagem de uma tabela dependente](#14-função--listagem-de-uma-tabela-dependente)
15. [Função — Cadastro de uma tabela dependente](#15-função--cadastro-de-uma-tabela-dependente)
16. [Função — Edição de uma tabela dependente](#16-função--edição-de-uma-tabela-dependente)
17. [Função — Listagem de uma tabela dependente de outras duas tabelas](#17-função--listagem-de-uma-tabela-dependente-de-outras-duas-tabelas)
18. [Função — Cadastro de atendimento com dupla referência](#18-função--cadastro-de-atendimento-com-dupla-referência)
19. [Função — Edição da data de atendimento](#19-função--edição-da-data-de-atendimento)
20. [Limitações e considerações para ambientes de produção](#20-limitações-e-considerações-para-ambientes-de-produção)
21. [Considerações finais](#21-considerações-finais)
---

## 1. CONFIGURAÇÃO DO AMBIENTE

### 1.1 Instalação das dependências

Abra o terminal e execute:

```bash
python -m pip install flask flask-cors pg8000 PyJWT
```

### 1.2 Bibliotecas utilizadas

| Biblioteca | Finalidade |
|---|---|
| Flask | Framework utilizado para a criação da API. |
| Flask-Cors | Permite requisições de diferentes origens. |
| pg8000 | Conexão e operações com o PostgreSQL. |
| PyJWT | Criação e validação de tokens JWT. |

### 1.3 Extensões do Visual Studio Code

Para auxiliar no desenvolvimento e nos testes, utilize:

- **Python:** extensão principal, que inclui Pylance, Python Debugger e Python Environments.
- **Thunder Client:** utilizada para realizar requisições HTTP e testar as rotas da API.

---

## 2. ESTRUTURA DO PROJETO

A organização inicial do projeto é composta por duas pastas principais e um arquivo em cada uma:

```text
Projeto/
├── backend/
│   └── app.py
└── database/
    └── database.py
```

- **app.py:** arquivo principal, responsável pela configuração do Flask e pelas rotas da API.
- **database.py:** módulo responsável pela conexão com o PostgreSQL.

### 2.1 Arquivos de cache

Durante a execução e os testes, o Python e o Flask podem gerar arquivos de cache automaticamente. Eles são temporários e não fazem parte do código principal da aplicação.

---

## 3. CONEXÃO COM O BANCO DE DADOS

### 3.1 Database — database.py

O arquivo `database.py` importa o módulo `pg8000`, utilizado para realizar a conexão com o PostgreSQL.

A função `get_db()` cria e retorna uma conexão com o banco `colegio_plural`, hospedado localmente.

**Código — database.py:**

```python
import pg8000.native

def get_db():
    return pg8000.native.Connection(
        host="localhost",
        port="5432",
        user="postgres",
        password="SUA_SENHA",
        database="colegio_plural"
    )
```

O arquivo `app.py` utiliza `get_db()` sempre que precisa acessar o banco de dados.

```text
app.py → get_db() → PostgreSQL
```

### 3.2 Acesso ao banco de dados pelo terminal

Para acessar o banco diretamente pelo terminal, utilize o executável `psql`.

No PowerShell, execute:

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" -U postgres -d colegio_plural
```

- `-U postgres`: usuário utilizado na conexão.
- `-d colegio_plural`: banco de dados acessado.

Após executar o comando, será solicitada a senha do usuário `postgres`.

> [!NOTE]
> Durante a digitação da senha, nenhum caractere será exibido. Isso é normal. Digite-a corretamente e pressione ENTER.
>
> Caso necessário, verifique o caminho do executável `psql.exe` e a versão do PostgreSQL instalada.

Para encerrar a conexão, utilize:

```text
\q
```

Ou pressione:

```text
Ctrl + C
```

---

## 4. CONFIGURAÇÃO DA APLICAÇÃO — APP.PY

O arquivo `app.py` concentra a configuração e o funcionamento da API.

### 4.1 Importação das bibliotecas

Inicialmente, são importados os módulos utilizados pelas funcionalidades da aplicação.

**Código — app.py:**

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
from database.database import get_db
import jwt
from datetime import datetime, timedelta, timezone
from functools import wraps
import hashlib
```

**Principais recursos utilizados:**

| Importação | Finalidade |
|---|---|
| Flask | Inicialização da aplicação e definição das rotas. |
| request | Acesso aos dados das requisições HTTP. |
| jsonify | Retorno de dados em formato JSON. |
| CORS | Configuração de requisições de diferentes origens. |
| get_db | Conexão com o PostgreSQL. |
| jwt | Geração e validação de tokens JWT. |
| datetime, timedelta e timezone | Manipulação de datas e horários. |
| wraps | Utilizado no decorador de proteção das rotas. |
| hashlib | Aplicação de hash MD5 no exemplo de senha. |

### 4.2 Inicialização do Flask

Esse trecho inicializa a aplicação Flask, habilita o CORS e define a chave secreta utilizada na geração e validação dos tokens JWT.

**Código — app.py:**

```python
app = Flask(__name__)
CORS(app)

SECRET = "SUA_CHAVE_SECRETA"
```

A variável `SECRET` é utilizada como chave para assinar e validar os tokens JWT.

> [!WARNING]
> Substitua os valores de exemplo pelos seus próprios dados. Não publique senhas ou chaves secretas reais no repositório.

### 4.3 Inicialização do servidor

> [!CAUTION]
> Esse é o último trecho de código do `app.py`, portanto, todas as funções e rotas devem ser definidas antes dele.

```python
if __name__ == "__main__":
    app.run(debug=True)
```

---

## 5. AUTENTICAÇÃO JWT

### 5.1 Proteção das rotas — token_required

Esse trecho cria um decorador responsável por proteger as rotas, verificando o token JWT enviado pelo Thunder Client no cabeçalho `Authorization`.

Caso o token não seja informado, esteja inválido ou expirado, o acesso à rota será negado.

**Código — app.py:**

```python
def token_required(f):
    @wraps(f)
    def verificar(*args, **kwargs):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")

        if not token:
            return jsonify({"erro": "Token não informado"}), 401

        try:
            jwt.decode(token, SECRET, algorithms=["HS256"])
        except jwt.InvalidTokenError:
            return jsonify({"erro": "Token inválido ou expirado"}), 401

        return f(*args, **kwargs)

    return verificar
```

O decorador será utilizado nas rotas que exigem autenticação.

---

## 6. EXECUÇÃO DA APLICAÇÃO

Para iniciar a API e realizar os testes das funcionalidades, abra o terminal na pasta principal do projeto e execute:

```bash
python -m backend.app
```

Após a execução, o servidor local estará disponível em:

```text
http://127.0.0.1:5000
```

Para encerrar o servidor, pressione:

```text
Ctrl + C
```

> [!NOTE]
> O `Ctrl + C` também pode ser utilizado para interromper a interação com o PostgreSQL pelo terminal. O comando `\q` é outra opção para sair do `psql`.

---

## 7. TESTES INICIAIS — APP.PY

Antes de testar as funcionalidades principais, são utilizadas duas rotas para verificar o funcionamento da aplicação e sua comunicação com o banco de dados.

### 7.1 Verificação da aplicação

Essa rota verifica se a aplicação Flask está funcionando corretamente, retornando uma mensagem simples de confirmação.

**Código — app.py:**

```python
@app.route("/")
def inicio():
    return "Funcionou carambolas"
```

Para testar, acesse:

```text
http://127.0.0.1:5000/
```

### 7.2 Teste de conexão com o banco de dados

Essa rota verifica a comunicação entre a API e o PostgreSQL, realizando uma consulta na tabela `atendimento`, fechando a conexão e retornando os registros encontrados em formato JSON.

**Código — app.py:**

```python
@app.route("/teste-banco")
def teste_banco():
    conn = get_db()
    resultado = conn.run("SELECT * FROM atendimento")
    conn.close()
    return jsonify(resultado)
```

Para testar, acesse:

```text
http://127.0.0.1:5000/teste-banco
```

> [!NOTE]
> Essa rota possui finalidade de teste e não integra diretamente as funcionalidades principais da API.

---

## 8. AUTENTICAÇÃO — LOGIN

### 8.1 Implementação da rota de login

A função `login()` utiliza o e-mail e a senha cadastrados para autenticar o usuário e permitir o acesso às funcionalidades da API.

Neste projeto, considera-se que as senhas foram previamente armazenadas utilizando hash MD5.

> [!WARNING]
> O MD5 é utilizado apenas como exemplo didático e não é recomendado para o armazenamento seguro de senhas em sistemas reais.

**Código — app.py:**

```python
@app.route("/login", methods=["POST"])
def login():
    dados = request.json
    conn = get_db()

    senha_md5 = hashlib.md5(
        dados["senha"].encode()
    ).hexdigest()

    usuario = conn.run(
        """
        SELECT id, username
        FROM usuario
        WHERE email = :email AND senha = :senha
        """,
        email=dados["email"],
        senha=senha_md5
    )

    conn.close()

    if not usuario:
        return jsonify({"erro": "Email ou senha inválidos"}), 401
```

A função recebe os dados enviados pelo Thunder Client, transforma a senha em hash MD5 e consulta o banco para verificar as credenciais. Caso não sejam encontradas, retorna o código HTTP `401` (Não autorizado).

### 8.2 Geração do token JWT

Após o login bem-sucedido, a API gera um token JWT contendo o ID e o nome do usuário, com validade de duas horas.

**Continuação da função `login()`:**

```python
    token = jwt.encode(
        {
            "id": usuario[0][0],
            "username": usuario[0][1],
            "exp": datetime.now(timezone.utc) + timedelta(hours=2)
        },
        SECRET,
        algorithm="HS256"
    )

    return jsonify({
        "mensagem": "Login realizado com sucesso",
        "token": token
    })
```

Após a geração, a API retorna uma mensagem de sucesso e o token, que será utilizado no cabeçalho `Authorization` para acessar as rotas protegidas.

### 8.3 Testando o login com Thunder Client

**Passo 1 — Criar uma requisição**

Abra a extensão Thunder Client e selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método padrão `GET` para `POST` e informe a URL:

```text
http://127.0.0.1:5000/login
```

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body` e informe o e-mail e a senha de um usuário cadastrado no banco.

```json
{
    "email": "usuario@exemplo.com",
    "senha": "sua_senha"
}
```

**Passo 4 — Enviar a requisição**

Clique em `Send`. Caso as credenciais estejam corretas, a API retornará uma mensagem de sucesso acompanhada do token JWT.


**Passo 5 — Copiar o token**

Copie o token retornado, **sem as aspas**. Ele será utilizado nas próximas requisições para acessar as rotas protegidas.

---

**Passo 6 — Configurar os Headers**

Após copiar o token, acesse a aba `Headers` da requisição no Thunder Client.

Exclua os cabeçalhos que estiverem preenchidos por padrão e adicione os seguintes:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

No campo `Authorization`, escreva `Bearer`, dê um espaço e cole o token completo, sem as aspas.

<img width="1689" height="328" alt="Resultado - Config Thunder Client" src="https://github.com/user-attachments/assets/55080ddc-c388-4e71-ba27-459b0835d927" />


<p>
    
</p>

Com essa configuração, o Thunder Client estará preparado para realizar as próximas requisições às rotas protegidas da API.

---

## 9. FUNÇÃO — LISTAR

A função `listar_responsaveis()` consulta todos os responsáveis cadastrados no banco de dados, organizando os registros em ordem alfabética pelo nome. Os resultados são retornados no formato JSON e a rota exige um token JWT válido para ser acessada.

### 9.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/responsaveis", methods=["GET"]) #listar responsáveis
@token_required
def listar_responsaveis():
    conn = get_db()

    responsaveis = conn.run("""
        SELECT id, nome, cpf, telefone, email, endereco
        FROM responsavel
        ORDER BY nome
    """)
    conn.close()

    campos = [
        "id",
        "nome",
        "cpf",
        "telefone",
        "email",
        "endereco"
    ]

    return jsonify([
        dict(zip(campos, r))
        for r in responsaveis
    ])
```

### 9.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método da requisição para `GET` e informe a seguinte URL:

```text
http://127.0.0.1:5000/responsaveis
```

**Passo 3 — Verificar os Headers**

Acesse a aba `Headers` e confira se os cabeçalhos configurados anteriormente continuam presentes:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 4 — Enviar a requisição**

Como a função utiliza o método `GET`, basta clicar em `Send` para executar a consulta.

**Resultado:**
<p></p>

<img width="1042" height="634" alt="Resultado - Listar1" src="https://github.com/user-attachments/assets/422b5e64-9879-4bba-8b0c-19322a11a6af" />


## 10. FUNÇÃO — BUSCAR POR TERMO

A função `buscar_responsavel()` permite pesquisar responsáveis cadastrados no banco de dados utilizando um termo informado pelo usuário. A pesquisa pode encontrar resultados correspondentes ao nome, CPF ou e-mail, retornando os registros encontrados em formato JSON.

### 10.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/responsaveis/buscar", methods=["GET"]) 
@token_required
def buscar_responsavel():
    termo = request.args.get("termo", "")

    conn = get_db()

    responsaveis = conn.run("""
        SELECT id, nome, cpf, telefone, email, endereco
        FROM responsavel
        WHERE nome ILIKE :termo
           OR cpf ILIKE :termo
           OR email ILIKE :termo
        ORDER BY nome
    """, termo=f"%{termo}%")

    conn.close()

    campos = [
        "id",
        "nome",
        "cpf",
        "telefone",
        "email",
        "endereco"
    ]

    return jsonify([
        dict(zip(campos, r))
        for r in responsaveis
    ])
```

### 10.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `GET` e informe a seguinte URL:

```text
http://127.0.0.1:5000/responsaveis/buscar?termo=Maria
```

O parâmetro `termo` deve ser informado imediatamente após a rota, utilizando `?` e sem espaços. Substitua `Maria` pelo termo desejado.

**Passo 3 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 4 — Enviar a requisição**

Como a função utiliza o método `GET`, basta clicar em `Send` para executar a pesquisa.

**Resultado:**
<p></p>
<img width="671" height="496" alt="Resultado - Buscar por termo" src="https://github.com/user-attachments/assets/5744fbb7-e9dc-4e94-a4fb-5e0c5eceda6d" />



---

## 11. FUNÇÃO — CADASTRO

A função `cadastrar_responsavel()` recebe os dados de um novo responsável em formato JSON e insere o registro na tabela `responsavel`.

A função exige autenticação JWT e retorna o código HTTP `201` quando o cadastro é realizado com sucesso.

### 11.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/responsaveis", methods=["POST"])                          
@token_required
def cadastrar_responsavel():
    dados = request.json
    conn = get_db()

    conn.run("""
        INSERT INTO responsavel
        (nome, cpf, telefone, email, endereco)
        VALUES
        (:nome, :cpf, :telefone, :email, :endereco)
    """,
    nome=dados["nome"],
    cpf=dados["cpf"],
    telefone=dados["telefone"],
    email=dados["email"],
    endereco=dados["endereco"])

    conn.close()

    return jsonify({
        "mensagem": "Responsável cadastrado com sucesso"
    }), 201
```

### 11.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `POST` e informe a seguinte URL:

```text
http://127.0.0.1:5000/responsaveis
```

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione a opção `JSON` e informe os dados do novo responsável:

```json
{
    "nome": "Ricardo Almeida",
    "cpf": "789.012.345-66",
    "telefone": "(32) 99999-4444",
    "email": "ricardo@email.com",
    "endereco": "Rua Nova, 400"
}
```

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para realizar o cadastro.

**Visão geral:**
<p></p>
<img width="1490" height="417" alt="Resultado - Cadastro1" src="https://github.com/user-attachments/assets/f79d6762-a4dd-48bb-b949-85abc0c8599d" />


---

## 12. FUNÇÃO — EDIÇÃO

A função `editar_responsavel()` recebe o ID do responsável pela URL e os novos dados em formato JSON, atualizando o registro correspondente no banco de dados.

A função exige autenticação JWT e utiliza o ID informado para garantir que apenas o responsável selecionado seja alterado.

### 12.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/responsaveis/<int:id>", methods=["PUT"])   
@token_required
def editar_responsavel(id):
    dados = request.json
    conn = get_db()

    conn.run("""
        UPDATE responsavel
        SET nome = :nome,
            cpf = :cpf,
            telefone = :telefone,
            email = :email,
            endereco = :endereco
        WHERE id = :id
    """,
    nome=dados["nome"],
    cpf=dados["cpf"],
    telefone=dados["telefone"],
    email=dados["email"],
    endereco=dados["endereco"],
    id=id)

    conn.close()

    return jsonify({
        "mensagem": "Responsável atualizado com sucesso"
    })
```

### 12.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `PUT` e informe a seguinte URL:

```text
http://127.0.0.1:5000/responsaveis/1
```

O número `1` representa o ID do responsável que será atualizado.

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione `JSON` e informe os novos dados:

```json
{
    "nome": "Maria Oliveira Atualizada",
    "cpf": "123.456.789-00",
    "telefone": "(32) 98888-1111",
    "email": "maria.nova@email.com",
    "endereco": "Rua Nova, 500"
}
```

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para atualizar o registro selecionado.


**Visão geral:**
<p></p>
<img width="1272" height="350" alt="Resultado -Edição1" src="https://github.com/user-attachments/assets/ea751904-25b6-4d3c-84af-22cc8d894e94" />



---

## 13. FUNÇÃO — EXCLUSÃO

A função `deletar_responsavel()` recebe o ID de um responsável pela URL e remove o registro correspondente da tabela `responsavel`.

A função exige autenticação JWT e utiliza o ID informado para identificar o registro que será excluído.

### 13.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/responsaveis/<int:id>", methods=["DELETE"])                        
@token_required
def deletar_responsavel(id):
    conn = get_db()

    conn.run(
        "DELETE FROM responsavel WHERE id = :id",
        id=id
    )

    conn.close()

    return jsonify({
        "mensagem": "Responsável deletado com sucesso"
    })
```

### 13.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `DELETE` e informe a seguinte URL:

```text
http://127.0.0.1:5000/responsaveis/4
```

O número `4` representa o ID do responsável que será excluído.

**Passo 3 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 4 — Enviar a requisição**

Como a função utiliza o método `DELETE`, não é necessário preencher nenhum JSON. Basta clicar em `Send`.

**Resultado:**

<p></p>
<img width="863" height="280" alt="Resultado - Exclusão1" src="https://github.com/user-attachments/assets/3c2c4030-d968-41d5-97b3-1dac61f75d8e" />


---

## 14. FUNÇÃO — LISTAGEM DE UMA TABELA DEPENDENTE

A função `listar_alunos()` consulta os alunos cadastrados e relaciona cada um ao seu respectivo responsável por meio de um `INNER JOIN`.

A função retorna os dados em formato JSON, ordenados pelo nome do aluno, e exige autenticação JWT.

### 14.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/alunos", methods=["GET"])                        
@token_required
def listar_alunos():
    conn = get_db()

    alunos = conn.run("""
        SELECT
            aluno.id,
            aluno.nome,
            aluno.data_nascimento,
            aluno.cpf,
            aluno.matricula,
            responsavel.nome AS responsavel
        FROM aluno
        INNER JOIN responsavel
            ON aluno.responsavel_id = responsavel.id
        ORDER BY aluno.nome
    """)

    conn.close()

    campos = [
        "id",
        "nome",
        "data_nascimento",
        "cpf",
        "matricula",
        "responsavel"
    ]

    return jsonify([
        dict(zip(campos, aluno))
        for aluno in alunos
    ])
```

### 14.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `GET` e informe a seguinte URL:

```text
http://127.0.0.1:5000/alunos
```

**Passo 3 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 4 — Enviar a requisição**

Como a função utiliza o método `GET`, basta clicar em `Send` para executar a consulta.

**Resultado:**

<p></p>
<img width="612" height="727" alt="Resultado - Listagem2" src="https://github.com/user-attachments/assets/14366b78-3d1a-470f-a31d-55e758de29b2" />


---

## 15. FUNÇÃO — CADASTRO DE UMA TABELA DEPENDENTE

A função `cadastrar_aluno()` recebe os dados de um novo aluno e insere o registro na tabela `aluno`, associando-o a um responsável existente.

A função exige autenticação JWT e retorna o código HTTP `201` quando o cadastro é realizado com sucesso. Portanto, segue a mesma lógica da função de cadastro anterior, com a diferença de depender de outra tabela para existir.

### 15.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/alunos", methods=["POST"])                                   
@token_required
def cadastrar_aluno():
    dados = request.json
    conn = get_db()

    conn.run("""
        INSERT INTO aluno
        (nome, data_nascimento, cpf, matricula, responsavel_id)
        VALUES
        (:nome, :data_nascimento, :cpf, :matricula, :responsavel_id)
    """,
    nome=dados["nome"],
    data_nascimento=dados["data_nascimento"],
    cpf=dados["cpf"],
    matricula=dados["matricula"],
    responsavel_id=dados["responsavel_id"])

    conn.close()

    return jsonify({
        "mensagem": "Aluno cadastrado com sucesso"
    }), 201

```

### 15.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `POST` e informe a seguinte URL:

```text
http://127.0.0.1:5000/alunos
```

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione `JSON` e informe os dados do novo aluno:

```json
{
    "nome": "Pedro Almeida",
    "data_nascimento": "2012-11-20",
    "cpf": "789.012.345-66",
    "matricula": "ALU004",
    "responsavel_id": 2
}
```

O campo `responsavel_id` identifica o responsável ao qual o aluno será associado.

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para realizar o cadastro.



**Visão geral:**

<p></p>
<img width="1259" height="365" alt="Resultado - Cadastro 2" src="https://github.com/user-attachments/assets/9274b279-b7c9-4eb3-b1e7-44aad0fff510" />


---

## 16. FUNÇÃO — EDIÇÃO DE UMA TABELA DEPENDENTE

A função `editar_aluno()` recebe o ID do aluno pela URL e atualiza seus dados no banco de dados, incluindo o responsável associado.

A função exige autenticação JWT e utiliza o ID informado para alterar somente o aluno selecionado.

### 16.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/alunos/<int:id>", methods=["PUT"])                             
@token_required
def editar_aluno(id):
    dados = request.json
    conn = get_db()

    conn.run("""
        UPDATE aluno
        SET nome = :nome,
            data_nascimento = :data_nascimento,
            cpf = :cpf,
            matricula = :matricula,
            responsavel_id = :responsavel_id
        WHERE id = :id
    """,
    nome=dados["nome"],
    data_nascimento=dados["data_nascimento"],
    cpf=dados["cpf"],
    matricula=dados["matricula"],
    responsavel_id=dados["responsavel_id"],
    id=id)

    conn.close()

    return jsonify({
        "mensagem": "Aluno atualizado com sucesso"
    })
```

### 16.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `PUT` e informe a seguinte URL:

```text
http://127.0.0.1:5000/alunos/4
```

O número `4` representa o ID do aluno que será atualizado.

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione `JSON` e informe os novos dados:

```json
{
    "nome": "Pedro Almeida Atualizado",
    "data_nascimento": "2012-11-20",
    "cpf": "789.012.345-66",
    "matricula": "ALU004",
    "responsavel_id": 2
}
```

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para atualizar o aluno selecionado.


**Visão geral:**

<p></p>
<img width="1260" height="366" alt="Resultado - Edição2" src="https://github.com/user-attachments/assets/ebd2e89f-0a58-40c7-b8d8-3005f4edef72" />


---

## 17. FUNÇÃO — LISTAGEM DE UMA TABELA DEPENDENTE DE OUTRAS DUAS TABELAS

A função `listar_atendimentos()` consulta os atendimentos cadastrados e relaciona cada registro ao aluno e ao responsável correspondente por meio de dois `INNER JOIN`.

Neste exemplo, são utilizados relacionamentos entre as tabelas `atendimento`, `aluno` e `responsavel`.

Existem diferentes tipos de `JOIN`, como `INNER`, `LEFT`, `RIGHT` e `FULL`, que podem ser utilizados conforme o resultado desejado. Neste projeto, o `INNER JOIN` foi utilizado para retornar apenas registros que possuem relacionamentos correspondentes.

### 17.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/atendimentos", methods=["GET"])                    
@token_required
def listar_atendimentos():
    conn = get_db()

    atendimentos = conn.run("""
        SELECT
            atendimento.id,
            atendimento.data_atendimento,
            atendimento.descricao,
            aluno.nome AS aluno,
            responsavel.nome AS responsavel
        FROM atendimento
        INNER JOIN aluno
            ON atendimento.aluno_id = aluno.id
        INNER JOIN responsavel
            ON aluno.responsavel_id = responsavel.id
        ORDER BY atendimento.data_atendimento
    """)

    conn.close()

    campos = [
        "id",
        "data_atendimento",
        "descricao",
        "aluno",
        "responsavel"
    ]

    return jsonify([
        dict(zip(campos, atendimento))
        for atendimento in atendimentos
    ])
```

### 17.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `GET` e informe a seguinte URL:

```text
http://127.0.0.1:5000/atendimentos
```

**Passo 3 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 4 — Enviar a requisição**

Como a função utiliza o método `GET`, basta clicar em `Send` para executar a consulta.



**Visão geral:**

<p></p>
<img width="1874" height="782" alt="Resultado -Listar3" src="https://github.com/user-attachments/assets/f49f1d1c-3646-4838-ba8c-da13dd09c84a" />



---

## 18. FUNÇÃO — CADASTRO DE ATENDIMENTO COM DUPLA REFERÊNCIA

A função `cadastrar_atendimento()` realiza o cadastro de um atendimento, relacionando-o a um aluno e ao usuário responsável pelo registro.

### 18.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/atendimentos", methods=["POST"])                                    
@token_required
def cadastrar_atendimento():
    dados = request.json
    conn = get_db()

    conn.run("""
        INSERT INTO atendimento
        (data_atendimento, descricao, aluno_id, usuario_id)
        VALUES
        (:data_atendimento, :descricao, :aluno_id, :usuario_id)
    """,
    data_atendimento=dados["data_atendimento"],
    descricao=dados["descricao"],
    aluno_id=dados["aluno_id"],
    usuario_id=dados["usuario_id"])

    conn.close()

    return jsonify({
        "mensagem": "Atendimento cadastrado com sucesso"
    }), 201
```

### 18.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `POST` e informe a seguinte URL:

```text
http://127.0.0.1:5000/atendimentos
```

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione `JSON` e informe os dados do atendimento:

```json
{
    "data_atendimento": "2026-09-15",
    "descricao": "Novo atendimento para acompanhamento do aluno.",
    "aluno_id": 1,
    "usuario_id": 1
}
```

Os campos `aluno_id` e `usuario_id` identificam, respectivamente, o aluno relacionado ao atendimento e o usuário responsável pelo registro.

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para realizar o cadastro.


**Visão geral:**

<p></p>
<img width="1430" height="377" alt="Resultado - Cadastro3" src="https://github.com/user-attachments/assets/6cea0329-24d4-40ea-bf98-3b8847cfe9f1" />


---

## 19. FUNÇÃO — EDIÇÃO DA DATA DE ATENDIMENTO

A função `editar_atendimento()` permite atualizar a data de um atendimento já cadastrado.

Essa funcionalidade segue a mesma lógica da função de edição anterior, recebendo o ID do atendimento pela URL e a nova data em formato JSON.

### 19.1 Implementação no app.py

Adicione o seguinte código ao arquivo `app.py`:

```python
@app.route("/atendimentos/<int:id>", methods=["PUT"])       
@token_required
def editar_atendimento(id):
    dados = request.json
    conn = get_db()

    conn.run("""
        UPDATE atendimento
        SET data_atendimento = :data_atendimento
        WHERE id = :id
    """,
    data_atendimento=dados["data_atendimento"],
    id=id)

    conn.close()

    return jsonify({
        "mensagem": "Data do atendimento atualizada com sucesso"
    })
```

### 19.2 Testando a função com Thunder Client

**Passo 1 — Criar uma nova requisição**

No Thunder Client, selecione `New Request`.

**Passo 2 — Configurar o método e a rota**

Altere o método para `PUT` e informe a seguinte URL:

```text
http://127.0.0.1:5000/atendimentos/4
```

O número `4` representa o ID do atendimento que terá sua data atualizada.

**Passo 3 — Preencher o corpo da requisição**

Acesse a aba `Body`, selecione `JSON` e informe a nova data:

```json
{
    "data_atendimento": "2026-09-11"
}
```

**Passo 4 — Verificar os Headers**

Confira se os cabeçalhos de autenticação continuam configurados:

| Header | Value |
|---|---|
| `Authorization` | `Bearer SEU_TOKEN_JWT` |
| `Content-Type` | `application/json` |

**Passo 5 — Enviar a requisição**

Clique em `Send` para atualizar a data do atendimento.

**Visão Geral:**

<p></p>
<img width="1072" height="391" alt="Resultado - Data1" src="https://github.com/user-attachments/assets/82cfa4f3-543e-4a6e-b045-d204c587ac78" />
<p></p>
<img width="962" height="258" alt="Resultado - DAta2" src="https://github.com/user-attachments/assets/49a47646-37dc-44fc-b83b-7dc62b91325e" />


---

## 21. Limitações e considerações para ambientes de produção

Este projeto foi desenvolvido com finalidade educacional, priorizando a simplicidade e a demonstração dos conceitos de desenvolvimento de APIs com Flask e PostgreSQL. Por esse motivo, algumas implementações foram simplificadas e não devem ser reproduzidas sem adaptações em ambientes profissionais.

A seguir, estão algumas considerações importantes para uma possível utilização em produção:

- **Chave secreta (SECRET):** neste projeto, a chave secreta utilizada na geração e validação dos tokens JWT está definida diretamente no código. Em ambientes profissionais, recomenda-se armazená-la em variáveis de ambiente, evitando sua exposição no código-fonte.

- **Modo de depuração (debug):** a aplicação utiliza `debug=True` para facilitar o desenvolvimento e a identificação de erros. Em produção, esse recurso deve ser desativado, pois pode expor informações internas da aplicação.

- **Configuração do CORS:** o CORS foi configurado de maneira ampla para facilitar as requisições durante os testes. Em ambientes profissionais, é importante restringir as origens autorizadas a acessar a API.

- **Armazenamento de senhas:** o projeto utiliza o algoritmo MD5 para demonstrar o processo de comparação de senhas durante a autenticação. Entretanto, MD5 não é adequado para armazenamento seguro de senhas. Em sistemas profissionais, recomenda-se utilizar algoritmos específicos para essa finalidade, como Argon2id ou bcrypt.

- **Autenticação e identificação do usuário:** na criação de atendimentos, o identificador do usuário é recebido por meio dos dados enviados pelo cliente. Em uma aplicação profissional, essa informação deve ser obtida diretamente dos dados validados do token JWT, evitando que um usuário informe o identificador de outra pessoa.

- **Validação de dados:** algumas operações possuem validações simplificadas. Em produção, é necessário verificar os campos obrigatórios, os formatos dos dados recebidos e a existência dos registros antes de realizar alterações ou exclusões.

- **Tratamento de erros:** o tratamento de exceções foi mantido simples para atender ao escopo do projeto. Em aplicações profissionais, recomenda-se implementar respostas padronizadas para diferentes situações de erro, além de garantir o tratamento adequado de falhas na comunicação com o banco de dados.

- **Rotas de teste:** a rota `/teste-banco` foi criada exclusivamente para verificar a comunicação entre a aplicação e o PostgreSQL, não fazendo parte das funcionalidades principais da API. Em ambientes de produção, rotas de teste que possam expor informações do banco de dados devem ser removidas ou devidamente protegidas.

Essas limitações não comprometem a finalidade didática do projeto, mas representam pontos que devem ser considerados em uma eventual evolução da aplicação para um ambiente de produção.

---

## CONSIDERAÇÕES FINAIS

Esta documentação apresentou o desenvolvimento e os testes de uma API REST utilizando Python, Flask e PostgreSQL, abordando autenticação JWT, operações CRUD, relacionamentos entre tabelas e consultas SQL.

O projeto foi desenvolvido com base em um mini-mundo educacional, permitindo explorar diferentes funcionalidades e possibilidades de integração entre tabelas em um banco de dados relacional.
