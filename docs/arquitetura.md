# Diagrama de arquitetura: Achados & Perdidos

```mermaid
flowchart TB
    cliente["Navegador / curl.exe<br/>(FORA da rede interna)"]

    subgraph host["Máquina host (Windows + Docker Desktop)"]
        initsql["./db/init.sql<br/>(arquivo do repositório)"]

        subgraph rede["rede_interna (bridge definida pelo usuário, DNS interno)"]
            web["<b>web</b><br/>Flask + Gunicorn<br/>escuta 0.0.0.0:5000<br/>usuário: appuser<br/>imagem própria (./app)"]
            db["<b>db</b><br/>postgres:16-alpine<br/>porta 5432 (NÃO publicada)"]
        end

        vfotos[("volume nomeado<br/><b>fotos</b>")]
        vdados[("volume nomeado<br/><b>dados_banco</b>")]
    end

    cliente -- "HTTP localhost:8080 → 5000<br/>(única porta publicada)" --> web
    web -- "SQL: host 'db', porta 5432" --> db
    web -- "montado em /app/uploads" --- vfotos
    db -- "montado em /var/lib/postgresql/data" --- vdados
    initsql -. "bind mount somente leitura (:ro)<br/>/docker-entrypoint-initdb.d/init.sql" .-> db
```

## Legenda

| Elemento | Dentro da rede interna? | Observação |
|---|---|---|
| `web` | Sim | Publica 8080 (host) → 5000 (contêiner). Roda como `appuser` |
| `db` | Sim | Nenhuma porta publicada no host; só o `web` o alcança pelo nome `db` |
| Navegador / curl | Não | Só acessa a porta 8080 |
| Volume `fotos` | Fora (armazenamento) | Montado em `/app/uploads` no `web` |
| Volume `dados_banco` | Fora (armazenamento) | Montado em `/var/lib/postgresql/data` no `db` |
| `init.sql` | Fora | Bind mount `:ro` em `/docker-entrypoint-initdb.d/`; executa só na primeira execução (volume vazio) |

Os volumes sobrevivem a `docker compose down` e só são apagados com `docker compose down -v`.
A única seta que atravessa a fronteira da rede é a da porta 8080.