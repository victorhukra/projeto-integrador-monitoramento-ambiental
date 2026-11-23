# Monitoramento Ambiental com Imagens de Satélite - Londrina/PR

Sistema integrador para monitoramento meteorológico de Londrina, visualização histórica de dados e previsão experimental de chuva na hora seguinte.

## Visao geral

O projeto combina três módulos:

- **Banco de Dados:** armazenamento de dados meteorologicos horarios no PostgreSQL 15 e no Supabase;
- **Backend:** API Flask para consulta dos dados, execucao do modelo v6 e persistencia das previsoes;
- **Frontend:** interface React para acompanhamento do clima, consultas por dia, mês e período e previsões experimentais.

O sistema foi desenvolvido no contexto do Projeto Integrador da UTFPR - Câmpus Londrina.

## Funcionalidades

- Consulta das condições meteorológicas atuais e históricas;
- Visualização por mapa, dia, semana, mês e período;
- Comparacao entre datas e analises historicas;
- Geracao de imagens GOES para o horario escolhido;
- Previsão experimental de chuva para a hora seguinte;
- Salvamento e consulta do histórico de previsões;
- Proteção contra duplicação por meio de chave de idempotência;
- Modos claro e escuro na interface.

## Arquitetura

```text
Frontend React/Vite
        |
        | HTTP
        v
Backend Flask + modelo v6
        |
        +----> Supabase/PostgreSQL
        |
        +----> Imagens GOES/NOAA
```

```text
.
|-- banco/
|   |-- init/             # Schema e carga inicial do PostgreSQL
|   |-- scripts/          # Importação da base completa
|   `-- doc/              # Documentação dos integrantes do banco
|-- backend/
|   |-- app/
|   |   |-- controllers/  # Rotas da API
|   |   |-- services/     # Regras de negocio e previsao
|   |   |-- models/       # Repositorio do Supabase
|   |   `-- views/        # Formatacao das respostas
|   |-- src/models/       # Modelo ensemble e manifesto v6
|   `-- docs/             # Documentação técnica do backend
|-- frontend/
|   |-- src/
|   |   |-- components/   # Componentes da interface
|   |   |-- services/     # Comunicacao com API e Supabase
|   |   `-- lib/          # Utilitários e formatação
|   `-- doc/              # Documentação dos integrantes do frontend
|-- compose.yaml           # Orquestração dos três containers
`-- .env.example           # Modelo das variaveis de ambiente
```

## Tecnologias

### Frontend

- React 18;
- Vite;
- JavaScript;
- Leaflet;
- Recharts;
- Supabase JS;
- Vitest;
- Nginx para entrega da build.

### Backend

- Python 3.13;
- Flask;
- scikit-learn;
- Joblib;
- NumPy;
- pandas;
- Supabase Python;
- NetCDF4 e bibliotecas de processamento GOES.

### Banco de Dados

- PostgreSQL 15;
- Supabase;
- SQL;
- Docker Compose;
- Row Level Security.

## Modelo de previsao

O modelo integrado utiliza um ensemble com três variantes de Random Forest, todas calibradas isotonicamente:

- `original`;
- `balanced`;
- `balanced_subsample`.

As variantes recebem pesos iguais de aproximadamente `1/3`.

Versao integrada:

```text
weather-v6-simple-mean-isotonic-v1
```

Entradas do modelo:

- temperatura a 2 metros;
- umidade relativa a 2 metros;
- pressão ao nível do mar;
- cobertura de nuvens;
- pressão ao nível do mar da hora anterior;
- produto temperatura x umidade;
- diferença de pressão entre T e T-1h;
- hora do dia.

O limiar atual de decisão é `0.5` e está identificado como provisório no manifesto do modelo.

> Esta tela apresenta uma simulação histórica. O modelo não representa previsão operacional aprovada e não utiliza dados em tempo real.

## Requisitos

- Docker Desktop;
- Docker Compose;
- Opcionalmente, Node.js 24 e Python 3.13 para execucao fora dos containers.

## Configuracao

Na raiz do projeto, copie o arquivo de exemplo:

```powershell
Copy-Item .env.example .env
```

Configure o arquivo `.env`:

```dotenv
MODEL_PATH=/app/src/models/weather_ensemble.joblib
MODEL_MANIFEST_PATH=/app/src/models/manifest.json

SUPABASE_URL=sua_url_do_supabase
SUPABASE_SERVICE_ROLE_KEY=sua_chave_service_role

VITE_SUPABASE_URL=sua_url_do_supabase
VITE_SUPABASE_ANON_KEY=sua_chave_anon_publica

FRONTEND_ORIGIN=http://localhost:5173,http://127.0.0.1:5173
VITE_API_URL=http://localhost:8000

DB_HOST=banco
DB_PORT=5432
DB_NAME=projeto_integrador
DB_USER=postgres
DB_PASSWORD=sua_senha_local
```

Nunca envie `.env`, chaves de serviço ou credenciais ao controle de versão.

## Executando com Docker

Na raiz do repositório:

```powershell
docker compose up --build -d 
```

Verifique os containers:

```powershell
docker compose ps
```

Acessos locais:

- Frontend: http://localhost:5173
- Backend: http://localhost:8000
- Banco PostgreSQL: `localhost:5432`

Para acompanhar os logs:

```powershell
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f banco
```

Para interromper os containers:

```powershell
docker compose down
```

Para interromper e apagar os dados locais persistidos:

```powershell
docker compose down -v
```

## API

Rotas principais:

| Metodo | Rota | Descricao |
|---|---|---|
| `GET` | `/health` | Estado do backend, banco e modelo |
| `GET` | `/available-times` | Periodo ou horarios disponiveis |
| `GET` | `/weather-inputs?reference_time=...` | Entradas meteorologicas de T e T-1h |
| `POST` | `/internal/predict` | Previsao sem persistencia |
| `POST` | `/predict` | Previsao persistida no historico |
| `GET` | `/predictions` | Historico paginado |
| `DELETE` | `/predictions/<prediction_id>` | Exclusao de uma previsao |
| `GET` | `/goes-image?reference_time=...` | Imagem GOES da Londrina |

Exemplo:

```text
http://localhost:8000/health
```


## Documentacao individual

A documentacao das contribuicoes de cada integrante esta organizada em:

- `frontend/doc/`;
- `backend/doc/`;
- `banco/doc/`.
