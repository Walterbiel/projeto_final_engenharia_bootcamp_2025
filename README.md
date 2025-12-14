# 1️⃣ Setup do Ambiente Local

## Pré-requisitos

- Python instalado  
- Acesso ao terminal (PowerShell ou Git Bash)  
- Docker instalado  

---

## Setup via Terminal

```bash
pip install uv
uv --version
```

```bash
cd .\1_local_setup\
```

```bash
uv venv .venv
```

```bash
.\.venv\Scripts\Activate.ps1
```

> **Importante**  
> Sempre executar os comandos **nesta ordem**.

---

## Instalação das Dependências

```bash
uv add dbt-core dbt-postgres duckdb faker pandas numpy
```

---

## Docker

- Criar o arquivo `docker-compose.yml` dentro da pasta `1_local_setup`

## .env

- Criar arquivo .env dentro da pasta `1_local_setup` com usuário e senha:

DBT_USER=postgres

DBT_PASSWORD=postgres

- Adicionar .env no .gitignore

## Docker
- Subir docker compose 
```bash
docker compose up -d
```

- Visualizar no docker desktop

# 2 - Data Warehouse

```bash
cd 2_data_warehouse
dbt init
```

- Digitar nome do projeto que será solicitado
- 1
- colocar host (local host)
- Porta: 5433
- user
- senha
- dbname = dbt_db
- schema = public
- threads = 4

Message reecbida: Profile dw_bootcamp written to C:\Users\Walter\.dbt\profiles.yml using target's profile_template.yml and your supplied values. Run 'dbt debug' to validate the connection.