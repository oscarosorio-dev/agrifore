# Setup local

## Requisitos
Docker, Docker Compose, Python 3.12, Node 22.23.4 o superior (lo exige `frontend/package.json`), `uv`, pnpm.

## Pasos
```bash
git clone <repo> && cd <repo>
cp .env.example .env
docker compose up -d --build
docker compose exec api alembic upgrade head
docker compose exec api python -m app.ingestion.run --desde 2020-02-01
docker compose exec api python -m app.forecasting.train
```

## Verificar
- API: http://localhost:8000/docs
- Frontend: http://localhost:4321
- Grafana: http://localhost:3000

## Pruebas
```bash
docker compose exec api pytest
```