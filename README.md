# Expert eGestor Academy

Plataforma de treinamento de funcionários do eGestor: React + Vite + Tailwind/shadcn no frontend,
Supabase **self-hosted** no backend (Postgres, Auth, Storage e Edge Functions).

## Onde roda

| Parte | Local | Endereço |
|-------|-------|----------|
| Site (build estático) | `mateus@10.1.1.17:/var/www/expert` | https://expert.egestor.com.br |
| Banco / API (Docker) | `root@76.13.174.32:/opt/expert-db` | https://srv1988116.hstgr.cloud/expert-api |

O `.env` aponta o frontend para a API. A chave dele é a `ANON_KEY` pública; a `SERVICE_ROLE_KEY`
fica só no `/opt/expert-db/.env` da VPS e nunca deve ir para o repositório.

## Desenvolvimento

```sh
npm ci
npm run dev        # http://localhost:8080 (usa o banco de produção do .env)
```

## Publicar o site

```sh
npm run build
rsync -rltz --delete --exclude='.well-known' dist/ mateus@10.1.1.17:/var/www/expert/
```

## Edge Functions

Ficam em `supabase/functions/`. Para publicar uma alteração:

```sh
scp -r supabase/functions/<nome> root@76.13.174.32:/opt/expert-db/volumes/functions/
ssh root@76.13.174.32 'cd /opt/expert-db && docker compose restart functions'
```

## Banco de dados

- Serviços: `cd /opt/expert-db && docker compose ps` (ajustes locais em `docker-compose.expert.yml`).
- Painel (Studio): `ssh -L 8000:127.0.0.1:8000 root@76.13.174.32` e abrir http://localhost:8000
  (usuário/senha em `DASHBOARD_USERNAME`/`DASHBOARD_PASSWORD` no `.env` da VPS).
- Backup diário às 03:30 em `/opt/expert-db/backup/daily` (banco + arquivos do storage, 14 dias),
  feito por `/opt/expert-db/backup.sh`.
- Restaurar um backup:
  `docker exec -i supabase-db pg_restore -U supabase_admin -d postgres --clean --if-exists < db-AAAA-MM-DD.dump`
