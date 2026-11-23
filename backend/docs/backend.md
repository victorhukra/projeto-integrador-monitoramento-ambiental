# Como funciona o backend

## Visão geral

O backend é uma API Flask servida por Gunicorn na porta 8000. Ele recebe horários de referência, consulta dados meteorológicos históricos no Supabase, prepara as entradas e calcula a probabilidade de chuva na próxima hora com o modelo v6. O resultado pode ser devolvido sem gravação ou salvo no histórico, conforme a rota usada.

## Inicialização

O `Dockerfile` instala as dependências, copia `app/`, `src/` e `config.py` para a imagem e inicia `app.main:app`. O pacote do modelo e o manifesto estão em `src/models/`; `MODEL_PATH` e `MODEL_MANIFEST_PATH` apontam para esses caminhos dentro do contêiner.

Para executar localmente com Docker, copie `.env.example` para `.env`, preencha `SUPABASE_URL` e `SUPABASE_SERVICE_ROLE_KEY`, e execute:

```sh
docker build -t monitoramento-backend .
docker run --env-file .env -p 8000:8000 monitoramento-backend
```

A chave `SUPABASE_SERVICE_ROLE_KEY` é privilegiada e deve ficar somente no ambiente do backend. Não publique o arquivo `.env`.

## Configuração

`app/settings.py` carrega as variáveis de ambiente, usando o arquivo `.env` local quando ele existe. As variáveis configuram o pacote de modelo, acesso ao Supabase e a origem de navegador permitida pelo CORS. O modelo e o banco são opcionais para iniciar o processo, mas as rotas que dependem deles não ficam operacionais sem a configuração correspondente.

## Organização do código

- `app/controllers/`: rotas HTTP, validação de parâmetros e coordenação dos serviços.
- `app/services/`: consulta e preparação dos dados meteorológicos, carregamento do modelo, disponibilidade e previsão.
- `app/models/`: acesso ao Supabase e persistência do histórico.
- `app/views/`: formato público das respostas e paginação.
- `app/main.py`: criação da aplicação, configuração de CORS e tratamento de erros.
- `src/models/`: adaptador, manifesto e pacote do modelo v6.
- `sql/`: migração para proteger as operações de leitura e gravação do histórico.

## Rotas principais

- `GET /health`: estado do modelo, do banco e da integração.
- `GET /available-times`: período disponível; com `?date=AAAA-MM-DD`, lista horários válidos.
- `GET /weather-inputs?reference_time=...`: consulta as entradas meteorológicas de um horário e da hora anterior.
- `POST /internal/predict?reference_time=...`: calcula a previsão sem gravá-la.
- `POST /predict`: calcula e grava uma previsão histórica; exige `Idempotency-Key` com UUID para evitar duplicatas.
- `GET /predictions`: consulta histórico por intervalo de criação, com paginação por cursor.
- `DELETE /predictions/<prediction_id>`: exclui um registro do histórico.
- `GET /goes-image?reference_time=...`: obtém e renderiza a imagem GOES associada ao horário.

Os parâmetros de data/hora devem incluir fuso quando a rota exigir um instante. A API responde erros conhecidos com status HTTP e um código/mensagem em JSON.

## Dependência do banco

As rotas meteorológicas, de previsão e histórico usam Supabase via PostgREST e a chave `service_role`. Para gravação/histórico, aplique a migração SQL em `sql/` ao projeto do banco antes de usar essas operações. O endpoint `/goes-image` também consulta o Supabase para localizar o arquivo de satélite e depois obtém os dados públicos da NOAA.
