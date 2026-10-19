# Evidências

Horários dos arquivos `12`, `14` e `16` a `18` são do Windows (UTC-3); o `criado_em` dos itens vem do banco, em UTC. A numeração não tem o número 08: não há evidência com esse prefixo.

## Tarefa 4: subida e testes funcionais

| Arquivo | Legenda |
| --- | --- |
| 01-compose-ps.txt | Os dois serviços em execução, db healthy; só o web publica porta (8080->5000), o db mostra apenas 5432/tcp. |
| 01b-build-docker-desktop.png | Build da imagem do web no Docker Desktop (11 etapas, concluído). |
| 02-health.txt | GET /health retorna status ok e banco conectado. |
| 03-itens.txt | GET /itens inicial: os três itens de exemplo do init.sql. |
| 03b-itens-apos-cadastro.txt | GET /itens após cadastrar dois itens com foto. |
| 04-itens-filtro.txt | GET /itens?local=biblio retorna só o item da Biblioteca. |
| 05-post.txt | Resposta do POST /itens com foto: id e nome do arquivo gerado. |
| 06-estatisticas.txt | 5 itens no banco e 2 fotos no volume. |
| 07-logs.txt | Últimas linhas do log do web, com o Gunicorn escutando em 0.0.0.0:5000 e as requisições recebidas. |
| foto-navegador-antes.png | Foto 6194647f...jpg aberta no navegador em localhost:8080/fotos/..., confirmando GET /fotos/\<nome>. |
| oculos.jpg | Imagem de teste enviada no POST /itens do item "Oculos de grau". |
| garrafa.jpg | Imagem de teste enviada no POST /itens do item "Garrafa verde". |

## Validações do ambiente

| Arquivo | Legenda |
| --- | --- |
| 09-network-inspect.txt | Rede rede\_interna (bridge) com os contêineres db e web conectados. |
| 10-image-ls.txt | Imagem achados-perdidos-web gerada e seu tamanho (206 MB em disco, 50,4 MB de conteúdo). |
| 11-whoami.txt | O processo do web roda como appuser, não como root (RNF04). |

## Tarefa 5: persistência

| Arquivo | Legenda |
| --- | --- |
| 12-antes-down.txt | Estado antes do down: horário, contêiner, estatísticas, itens, fotos no volume e IDs dos contêineres. |
| 12b-foto-antes-down.png | A foto 6194647f...jpg aberta no navegador antes do down. |
| 13-down-up.txt | Saída de docker compose down seguido de up -d (sem -v). |
| 14-depois-up.txt | Mesmo estado após recriar: dados iguais, contêineres com IDs diferentes. |
| 14b-foto-apos-recriar.png | A mesma foto, com a mesma URL, aberta após down + up. |
| 15-volume-ls.txt | Volumes do Docker, incluindo os dois do projeto: dados\_banco e fotos. |
| 15-volume-inspect-dados\_banco.txt | Inspeção do volume do banco: driver local e Mountpoint no disco do Docker. |
| 15-volume-inspect-fotos.txt | Inspeção do volume de fotos: driver local e Mountpoint em /var/lib/docker/volumes/.../\_data. |
| 15-ls-uploads.txt | Conteúdo de /app/uploads no contêiner web, arquivos de propriedade de appuser. |
| 16-antes-down-v.txt | Estado imediatamente antes do down -v. |
| 17-down-v.txt | Saída do down -v (remove contêineres, rede e volumes) e do up -d seguinte. |
| 17b-volume-ls-apos-down-v.txt | Lista de volumes entre o down -v e o up: os dois volumes do projeto não existem mais. |
| 18-depois-down-v.txt | Após down -v + up: 3 itens, 0 fotos, criado\_em novo (os itens de exemplo foram recriados pelo init.sql). |
| 18b-foto-apos-down-v.txt | A foto antiga retorna HTTP 404 após a remoção do volume fotos. |
| 18c-volume-inspect-fotos-recriado.txt | Volume fotos recriado após o down -v, com novo CreatedAt (22:20:46Z) e o mesmo Mountpoint. |

## Questão Q7

| Arquivo | Legenda |
| --- | --- |
| 19-scale-web3.txt | docker compose up -d --scale web=3 falha: as réplicas disputam a mesma porta 8080 do host. |
