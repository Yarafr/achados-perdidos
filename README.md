# Achados & Perdidos: ambiente em contêineres

Aplicação web (Flask + Gunicorn) com banco PostgreSQL para controlar objetos perdidos do campus, empacotada com Docker Compose. Um único comando sobe o ambiente completo, e os dados (itens e fotos) ficam em volumes nomeados, sobrevivendo à recriação dos contêineres.

Documentação adicional:

- Diagrama: [`docs/arquitetura.md`](docs/arquitetura.md)
- Relatório técnico: [`docs/RELATORIO.md`](docs/RELATORIO.md)
- Reflexão e equivalência em nuvem: [`docs/REFLEXAO.md`](docs/REFLEXAO.md)
- Evidências dos testes: [`docs/evidencias/`](docs/evidencias/LEIAME.md)

## Pré-requisitos

- Docker Desktop (ou Docker Engine) com Docker Compose v2, **em execução**
- Porta **8080** livre no host
- Nenhuma outra instalação é necessária (nem Python, nem PostgreSQL)

Teste: `docker --version` e `docker compose version`.

## Estrutura

```
achados-perdidos/
|-- app/
|   |-- app.py
|   |-- requirements.txt
|   |-- Dockerfile
|   `-- .dockerignore
|-- db/
|   `-- init.sql
|-- docs/                 diagrama, relatório, reflexão e evidências
|-- docker-compose.yml
|-- .env.example          modelo de configuração (sem senha real)
|-- .gitignore
`-- README.md
```

## Configurar o `.env`

O `.env` guarda as credenciais e **não é versionado**. Crie-o a partir do modelo:

```bash
cp .env.example .env          # Linux / macOS / Git Bash
```

```powershell
Copy-Item .env.example .env   # PowerShell
```

Abra o `.env` e troque os valores:

```
DB_NAME=achados
DB_USER=usuario_exemplo
DB_PASSWORD=troque_esta_senha
```

Regras: sem espaços ao redor do `=`, sem aspas e sem `$` ou `#` na senha. A senha só é aplicada na **primeira** criação do banco; se for alterada depois, rode `docker compose down -v` (apaga os dados) para recriá-lo.

## Subir o ambiente

```bash
docker compose up -d --build
docker compose ps
```

Esperado: `db` como `running (healthy)` e `web` como `running`, com `0.0.0.0:8080->5000/tcp`. O `web` só inicia depois que o banco estiver saudável. Na primeira vez, o download das imagens pode levar alguns minutos.

## Testar

No PowerShell use `curl.exe` (o `curl` puro é outro comando).

```bash
curl http://localhost:8080/health
curl http://localhost:8080/itens
curl "http://localhost:8080/itens?local=biblio"
curl -F descricao="Oculos de grau" -F local="Bloco B" -F foto=@oculos.jpg http://localhost:8080/itens
curl http://localhost:8080/estatisticas
docker compose logs web --tail 20
docker compose exec web whoami        # deve retornar appuser, não root
```

Para ver uma foto, pegue o campo `foto` de `GET /itens` (por exemplo `/fotos/abc123.jpg`) e abra `http://localhost:8080/fotos/abc123.jpg` no navegador.

## Rotas da API

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Status da API e do banco: `{"status":"ok","banco":"conectado",...}` |
| GET | `/itens` | Lista os objetos cadastrados. `?local=texto` filtra por local (sem diferenciar maiúsculas) |
| POST | `/itens` | Cadastra um objeto. Campos de formulário: `descricao` e `local` (obrigatórios) e `foto` (arquivo, opcional). Retorna `201` com `id` e nome da foto |
| GET | `/fotos/<nome>` | Devolve a foto armazenada |
| GET | `/estatisticas` | Total de itens no banco e de fotos no volume |

## Parar e destruir

| Comando | O que faz | Dados |
|---|---|---|
| `docker compose stop` | Para os contêineres | Preservados |
| `docker compose down` | Remove contêineres e rede | **Preservados** (volumes mantidos) |
| `docker compose up -d` | Sobe de novo | Itens e fotos continuam lá |
| `docker compose down -v` | Remove contêineres, rede **e volumes** | **Apagados**: voltam só os 3 itens de exemplo do `init.sql` |

> Atenção: `docker compose down -v` apaga definitivamente os itens cadastrados e as fotos.

## Observações

- Só a aplicação web é acessível do host (porta 8080). O banco não publica porta e fica isolado na `rede_interna`.
- O `init.sql` só é executado quando o volume `dados_banco` está vazio. Alterá-lo depois não recria a tabela; aplique a mudança com `docker compose exec db psql ...` ou recrie o volume.
- O processo da aplicação roda como usuário sem privilégios (`appuser`).
- Nenhuma senha está no Dockerfile, no `docker-compose.yml` ou no código.