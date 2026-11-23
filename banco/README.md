# Módulo de Banco de Dados — Monitoramento Ambiental (Londrina-PR)

Camada de banco de dados relacional (PostgreSQL 15) do Projeto Integrador — **Sistema Inteligente de Monitoramento Meteorológico e Previsão de Chuva com IA** (UTFPR Câmpus Londrina).

## Estrutura

```text
.
├── compose.yaml                      # Orquestração Docker do container PostgreSQL 15
├── .env.example                      # Modelo de variáveis de ambiente (copiar para .env)
├── init/
│   ├── 01_schema.sql                 # DDL completo (3 tabelas, 1 view, 6 índices, RLS)
│   └── 02_dados.sql                  # Carga inicial real (792 registros horários + amostras)
├── scripts/
│   └── importar_dataset_completo.sql # Script opcional para carga completa (59.160 registros)
├── doc/                              # Documentação individual dos integrantes do grupo de BD
│   ├── gustavo_costa.txt
│   ├── veronica_soreano.txt
│   ├── samuel_rosa.txt
│   └── luna_maria.txt
└── COMANDOS_DOCKER_AVALIACAO.md      # Guia de comandos para execução e testes em Docker
```

## Execução com Docker Compose

```bash
git clone https://github.com/projeto-monitoramento-ambiental/banco.git
cd banco
cp .env.example .env
docker compose up -d banco
```

## Validação via `psql`

```bash
docker compose exec banco psql -U postgres -d projeto_integrador
```

Dentro do `psql`:

```sql
\dt
\dv
SELECT * FROM clima_atual LIMIT 5;
SELECT COUNT(*) FROM dataset_weather_goes;
\q
```

Para encerrar (preservando os dados):

```bash
docker compose down
```

Para encerrar e apagar o volume (reset total):

```bash
docker compose down -v
```
