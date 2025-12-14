# 🚀 Projeto Final – Data Warehouse com dbt, Docker e Airflow

Este projeto tem como objetivo construir um **Data Warehouse completo**, utilizando **PostgreSQL**, **dbt**, **Docker** e **Airflow**, seguindo boas práticas de engenharia de dados.

---

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

## 🐳 Docker

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

Adicionar o arquivo `.env` ao `.gitignore`.

---

### Subir o Ambiente Docker

```bash
docker compose up -d
```

Validar se o container está rodando no Docker Desktop.

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

---

### Configuração Interativa do dbt

Durante a inicialização:

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

- Colocar arquivos CSV dentro da pasta `seeds/`
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

- **staging**: limpeza e padronização dos dados
- **intermediate**: criação de fatos e dimensões
- **mart**: modelos finais prontos para análise

---

## Execução do dbt

Pipeline completo:

```bash
dbt build
```

Rodar apenas um modelo específico:

```bash
dbt run -s stg_airline_delay_cause
```

Para rodar sem executar seeds novamente:

```bash
dbt build --exclude-resource-type seed
```

---

## Pacotes do dbt

Criar o arquivo `packages.yml`:

```yml
packages:

  - package: dbt-labs/dbt_utils
    version: "1.3.0"

  - package: metaplane/dbt_expectations
    version: "0.10.8"
```

Instalar os pacotes:

```bash
dbt deps
```

---

## 3️⃣ Airflow

Instalar o Astro CLI:

```bash
winget install -e --id Astronomer.Astro
```

Inicializar o Airflow:

```bash
cd 3_airflow
astro dev init
```

---

### Dockerfile do Airflow

Adicionar:

```dockerfile
RUN python -m venv dbt_venv \
    && . dbt_venv/bin/activate \
    && pip install --no-cache-dir dbt-postgres==1.9.0 \
    && deactivate
```

---

### Docker Compose Override

```yml
services:
  scheduler:
    volumes:
      - ./dbt/dw_bootcamp:/usr/local/airflow/dbt/dw_bootcamp

  dag-processor:
    volumes:
      - ./dbt/dw_bootcamp:/usr/local/airflow/dbt/dw_bootcamp
```

---

### Documentação do dbt

```bash
cd ..\2_data_warehouse\dw_bootcamp
dbt docs generate --target dev
dbt docs serve --port 8085
```
