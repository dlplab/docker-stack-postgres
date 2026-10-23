# docker-stack-postgres

PostgreSQL 18 + pgvector (`pgvector/pgvector:0.8.7-pg18-trixie`), deployed as a Portainer stack.

## Deploy

Portainer > Stacks > Add stack > Repository: `dlplab/docker-stack-postgres`, compose path `stack.yaml`.
Set stack env vars `POSTGRES_PASSWORD` and `HONCHO_DB_PASSWORD` (see `.env.example`).

## Layout

- Volume `postgres_data` mounted at `/var/lib/postgresql` (PG18 image layout: PGDATA = `/var/lib/postgresql/18/docker`).
- Network `postgres` (created by this stack). Client stacks join it:
  ```yaml
  networks:
    postgres:
      external: true
      name: postgres
  ```
- Not published on the host. Admin: `docker exec -it postgres psql -U postgres`.
- Locale provider `builtin` (C.UTF-8): no glibc collation drift, no REINDEX on image/OS upgrades.
- Data checksums on (PG18 default), scram-sha-256 auth.

## Tenants

| DB | Role | Extensions | Connection string |
|---|---|---|---|
| honcho | honcho | vector, pg_trgm, btree_gin | `postgresql+psycopg://honcho:<pw>@postgres:5432/honcho` |

The init script (`configs.initdb-honcho`) runs only on first start with an empty volume.
To add a tenant later, run the SQL manually:

```sql
CREATE ROLE app LOGIN PASSWORD '...';
CREATE DATABASE app OWNER app;
\c app
CREATE EXTENSION IF NOT EXISTS vector;
```
