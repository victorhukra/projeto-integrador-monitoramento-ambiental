# Como funciona o backend

## Visão geral

O backend é uma API Flask executada em um container Docker. Ele consulta dados meteorológicos no Supabase, executa o modelo v6 e retorna ou grava previsões de chuva.

## Configuração

Na raiz do projeto, copie o arquivo de exemplo:

```powershell
Copy-Item .env.example .env
```

Preencha o `.env` (Não publique o arquivo `.env` nem a chave `SUPABASE_SERVICE_ROLE_KEY`).

## Inicialização

Na raiz do projeto:

```powershell
docker compose up --build -d
```

Verificar os containers:

```powershell
docker compose ps
```

Ver logs do backend:

```powershell
docker compose logs -f backend
```

A API fica disponível em `http://localhost:8000` e o Frontend em `http://localhost:5173`.

Os caminhos do modelo são configurados automaticamente pelo `compose.yaml`.

## Rotas principais

```text
GET  /health
GET  /available-times
GET  /weather-inputs?reference_time=...
POST /predict
GET  /predictions
DELETE /predictions/<prediction_id>
GET  /goes-image?reference_time=...
```

Exemplo de consulta no navegador:

```text
http://localhost:8000/health
```

## Fluxo de previsão

1. O Frontend envia uma requisição ao Backend.
2. O Backend valida o horário e consulta `T` e `T−1h`.
3. O modelo v6 calcula a probabilidade de chuva.
4. O resultado é devolvido e, em `/predict`, salvo no histórico.

## Organização

- `app/controllers/`: rotas e validações.
- `app/services/`: regras de negócio e previsão.
- `app/models/`: acesso ao Supabase.
- `src/models/`: modelo v6 e manifesto.
- `Dockerfile`: imagem do backend.
- `sql/`: migração do histórico de previsões.
