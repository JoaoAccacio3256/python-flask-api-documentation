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
4. [Configuração da aplicação](#4-configuração-da-aplicação--apppy)
5. [Autenticação JWT](#5-autenticação-jwt)
6. [Execução da aplicação](#6-execução-da-aplicação)
7. [Testes iniciais](#7-testes-iniciais--apppy)
8. [Autenticação — Login](#8-autenticação--login)

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

```sql
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

Copie o token retornado, sem as aspas. Ele será utilizado nas próximas requisições para acessar as rotas protegidas.

---
