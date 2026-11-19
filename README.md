# Hanys

Hanys is a self-hosted developer portfolio and read-only homelab monitoring dashboard built with .NET and Blazor.

> Work in progress. Features and architecture may change as the project develops.

## Planned stack

- .NET 10 / ASP.NET Core / Blazor
- PostgreSQL and Entity Framework Core
- Prometheus for metrics collection
- Grafana for local metrics visualization
- SignalR for live dashboard updates
- Docker Compose for local infrastructure
- GitHub Actions for CI

## Repository layout

```text
HomePulse/
├── .github/
│   └── workflows/
├── deploy/
│   ├── prometheus/
│   ├── .env.example
│   └── docker-compose.yml
├── docs/
├── src/
├── tests/
├── .dockerignore
├── .gitignore
└── README.md
```

The `src/` and `tests/` directories will contain the .NET solution and projects.

## Local infrastructure (Windows 11 + Docker Desktop)

### 1. Configure environment variables

From the `deploy` directory, copy the example file:

```powershell
Copy-Item .env.example .env
```

Edit `deploy/.env` and replace `POSTGRES_PASSWORD` with a unique, strong password. The real `.env` file is ignored by Git.

### 2. Start services

```powershell
cd deploy
docker compose up -d
docker compose ps
```

Current local port mappings:

| Service | Local address |
|---|---|
| PostgreSQL | `localhost:15432` |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3001 |

These host ports can be changed in `deploy/docker-compose.yml`. PostgreSQL still listens on port `5432` inside its container; Grafana listens on `3000` inside its container.

### 3. Check Prometheus

Open http://localhost:9090/targets.

If the Windows Exporter is installed and running, the `windows` target should become `UP`. The Prometheus scrape target is configured in `deploy/prometheus/prometheus.yml`.

### 4. Configure Grafana

Open http://localhost:3001 and sign in. On a fresh Grafana installation, the initial credentials are commonly `admin` / `admin`; change the password when prompted.

Add a Prometheus data source with this URL:

```text
http://prometheus:9090
```

Use the Docker service name `prometheus` here, not `localhost`, because Grafana connects to Prometheus over the Docker network.

## Useful Docker commands

Run these from `deploy/`:

```powershell
docker compose up -d
docker compose ps
docker compose logs -f
docker compose restart
docker compose stop
docker compose start
docker compose down
```

**Warning:** `docker compose down -v` removes named volumes and deletes the persisted local database and monitoring data. Do not use it unless you intend to reset that data.

## .NET development

Once the solution has been created:

```powershell
dotnet restore
dotnet build
dotnet test
```

The GitHub Actions workflow expects a solution file named `Hanys.sln` in the repository root and targets .NET 10.

## Security notes

- Never commit `.env`, passwords, API tokens, certificates, private keys, or production connection strings.
- Keep local infrastructure ports bound to `127.0.0.1` unless you intentionally need network access.
- Do not expose the Docker socket to the public web application.
- Treat public monitoring data as untrusted and expose only sanitized, read-only information.
- Before deploying publicly, review firewall rules, authentication, secrets handling, and reverse-proxy configuration.

## Project status

Early setup / infrastructure phase. See the project issues or Kanban board for planned work.
