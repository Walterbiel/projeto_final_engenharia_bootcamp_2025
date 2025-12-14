
## 1️⃣ Setup do Ambiente Local

### Pré-requisitos

- Python instalado  
- Acesso ao terminal (PowerShell ou Git Bash)  
- Docker e Docker Desktop instalados  

---

### Estrutura de Pastas do Projeto

Criar as seguintes pastas na raiz do repositório:

```
1_local_setup
2_data_warehouse
3_airflow
```

---

### Setup do Ambiente Python

Instalar o `uv` (gerenciador de dependências e ambientes virtuais):

```bash
pip install uv
uv --version
```

Entrar na pasta de setup local:

```bash
cd .\1_local_setup\
```

Criar e ativar o ambiente virtual:

```bash
uv venv .venv
.\.venv\Scripts\Activate.ps1
```

> ⚠️ **Importante**  
> Sempre execute os comandos **nesta ordem** para evitar problemas de ambiente.

---

### Instalação das Dependências

```bash
uv add dbt-core dbt-postgres duckdb faker pandas numpy
```

---

## Docker

### Docker Compose

- Criar o arquivo `docker-compose.yml` dentro da pasta `1_local_setup`
- Responsável por subir o PostgreSQL local

---

### Arquivo `.env`

Criar o arquivo `.env` dentro da pasta `1_local_setup`:

```env
DBT_USER=postgres
DBT_PASSWORD=postgres
```

Adicionar o `.env` ao `.gitignore`.

---

### Subir o Ambiente Docker

```bash
docker compose up -d
```

Validar containers no Docker Desktop.

---

## 2️⃣ Data Warehouse com dbt

Entrar na pasta:

```bash
cd 2_data_warehouse
```

Inicializar o projeto dbt:

```bash
dbt init
```

### Configuração Interativa

- Nome do projeto
- Opção: PostgreSQL
- Host: `localhost`
- Porta: `5433`
- User: conforme `.env`
- Senha: conforme `.env`
- Database: `dbt_db`
- Schema: `public`
- Threads: `4`

---

### Validação da Conexão

```bash
cd dw_bootcamp
dbt debug
```

---

## Seeds

- Colocar arquivos CSV em `seeds/`
- Seeds representam dados de origem para estudo e testes

---

## Configuração do `dbt_project.yml`

```yml
name: 'dw_bootcamp'
version: '1.0.0'

profile: 'dw_bootcamp'

model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

vars:
  "dbt_date:time_zone": "America/Sao_Paulo"

clean-targets:
  - "target"
  - "dbt_packages"

models:
  dw_bootcamp:
    staging:
      +materialized: view
    intermediate:
      +materialized: table
    mart:
      +materialized: table
```

---

## Estrutura de Models

```
models/
├── staging
├── intermediate
└── mart
```

- **staging**: limpeza e padronização
- **intermediate**: fatos e dimensões
- **mart**: modelos finais para análise

---

## Documentação (Opcional)

Criar `_stg_models.yml` para documentação e testes.

---

## Execução

Pipeline completo:

```bash
dbt build
```

Modelo específico:

```bash
dbt run -s stg_airline_delay_cause
```

---

## Pacotes do dbt

Criar `packages.yml`:

```yml
packages:

  - package: dbt-labs/dbt_utils
    version: "1.3.0"

  - package: metaplane/dbt_expectations
    version: "0.10.8"
```

Instalar dependências:

```bash
dbt deps
```

---

## O que é `dbt deps`

Comando responsável por baixar e instalar pacotes definidos em `packages.yml`.

Cria a pasta `dbt_packages/` com macros e testes reutilizáveis.

----

dbt build --exclude-resource-type seed para não rodar seeds de novo

---

## 3️⃣ Airflow

Instalar astro cli – https://www.astronomer.io/docs/astro/cli/install-cli

```bash
winget install -e --id Astronomer.Astro
```

Run astro version to confirm the Astro CLI is installed properly.

```bash
cd 3_airflow 
astro dev init
```

Adicionar no Dockerfile do airflow:

```dockerfile
RUN python -m venv dbt_venv \
    && . dbt_venv/bin/activate \
    && pip install --no-cache-dir dbt-postgres==1.9.0 \
    && deactivate
```

Criar pasta no `3_airflow` chamada `dbt` para colocar o projeto e copiar a pasta `dw_bootcamp` do dbt para a pasta criada no airflow, arrasta com o ctrl para cima da outra pasta para copiar

---

-- Criar DAG airflow para rodar o fluxo de cargas do DW automaticamente
criar arquivo na pasta de dags , dag.py

---

Criar  docker-compose.override.yml

services:
  scheduler:
    volumes:
      - ./dbt/dw_bootcamp:/usr/local/airflow/dbt/dw_bootcamp

  dag-processor:
    volumes:
      - ./dbt/dw_bootcamp:/usr/local/airflow/dbt/dw_bootcamp


depois rodar astro dev start 

Matar porta 5432 se precisar

airflow vai abrir a interface

Vai dar erro na dag:

Timestamp: 2025-12-14 18:44:01

Traceback (most recent call last):
  File "<frozen importlib._bootstrap>", line 488, in _call_with_frames_removed
  File "/usr/local/airflow/dags/dag.py", line 10, in <module>
    from cosmos import DbtDag, ProjectConfig, ProfileConfig, ExecutionConfig
ModuleNotFoundError: No module named 'cosmos'

----

Adicionar conexão no airfloe do banco
-docker_postgres_db
-postgres
host: host.docker.internal

login: postgres
senha: postgres
porta: 5433
schema: dbt_db

adicionar astronomer-cosmos no requeriments.txt da pasta do airflow.

astro dev stop
astro dev start --no-cache


rodar dag na interface do airflow

---

rodar documentação do dbt

cd ..\2_data_warehouse\dw_bootcamp
dbt deps
dbt docs generate --target dev
dbt docs serve --port 8085
