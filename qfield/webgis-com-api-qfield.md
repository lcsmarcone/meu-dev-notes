## Webgis com api do QFIELD (ainda não testado) 

O objetivo é automatizar a coleta de dados atualizados diretamente do QField Cloud e disponibilizá-los em um WebGIS (PostGIS + GeoServer ou API customizada).


1️⃣ Pré-requisitos

Antes de iniciar, garanta que você tem:

    ✅ Instância QField Cloud Self-Hosted rodando na Digital Ocean (guia oficial).

    ✅ Acesso SSH ao servidor.

    ✅ Banco PostgreSQL/PostGIS configurado.

    ✅ Usuário administrador do QField Cloud para criar tokens de API.

    ✅ Python 3.10+ (recomendado para scripts de automação).

2️⃣ Obter Token de Autenticação da API

A API do QField Cloud usa JWT para autenticação.
Passos:

    Acesse a sua instância do QField Cloud:

    https://sua-instancia-qfieldcloud.com

    Faça login com seu usuário admin.

    Vá até as Configurações da Conta → API Tokens.

    Gere um token e salve-o em um arquivo seguro (.env ou variável de ambiente).

Exemplo .env:

QFIELD_API_URL=https://sua-instancia-qfieldcloud.com/api/v1
QFIELD_API_TOKEN=seu_token_aqui

3️⃣ Explorar a API

Os principais endpoints da API do QField Cloud são:
Endpoint	Método	Descrição
/api/v1/projects/	GET	Lista todos os projetos
/api/v1/projects/{id}/	GET	Detalhes de um projeto específico
/api/v1/projects/{id}/download/	GET	Baixa o snapshot (GeoPackage) do projeto
/api/v1/files/	GET	Lista arquivos armazenados no projeto

📌 Exemplo de chamada:

curl -H "Authorization: Bearer SEU_TOKEN" https://sua-instancia-qfieldcloud.com/api/v1/projects/

Isso retorna um JSON com a lista de projetos.
4️⃣ Baixar Dados via API (Python)

Aqui está um script para automatizar o download do GeoPackage atualizado do QField Cloud:

import os
import requests
from dotenv import load_dotenv

# Carregar variáveis de ambiente
load_dotenv()

API_URL = os.getenv("QFIELD_API_URL")
API_TOKEN = os.getenv("QFIELD_API_TOKEN")
PROJECT_ID = "id_do_projeto"  # Obtido pelo endpoint /projects/

headers = {"Authorization": f"Bearer {API_TOKEN}"}

# 1. Baixar snapshot do projeto
download_url = f"{API_URL}/projects/{PROJECT_ID}/download/"
response = requests.get(download_url, headers=headers)

if response.status_code == 200:
    with open("projeto.gpkg", "wb") as f:
        f.write(response.content)
    print("GeoPackage baixado com sucesso!")
else:
    print("Erro ao baixar:", response.status_code, response.text)

5️⃣ Importar para PostGIS

Depois de baixar o GeoPackage, importe-o para o PostGIS.
Use ogr2ogr:

ogr2ogr -f "PostgreSQL" PG:"host=localhost dbname=seubanco user=usuario password=senha" \
  projeto.gpkg -overwrite -progress

Ou via Python:

import subprocess

subprocess.run([
    "ogr2ogr",
    "-f", "PostgreSQL",
    "PG:host=localhost dbname=seubanco user=usuario password=senha",
    "projeto.gpkg",
    "-overwrite"
])

6️⃣ Publicar no GeoServer

Agora que os dados estão no PostGIS:

    Acesse o GeoServer.

    Crie uma Store apontando para o banco PostGIS.

    Publique as camadas.

    Ative WMS/WFS/WMTS para consumo no WebGIS.

Se preferir tiles vetoriais:

    Use pg_tileserv para servir dados diretamente como MVT.

7️⃣ Consumir no WebGIS

No frontend (React Leaflet ou OpenLayers), consuma:

    WMS ou WMTS do GeoServer.

    Ou MVT via pg_tileserv.

    Ou crie uma API com Django + Django Rest Framework GIS para expor GeoJSON.

Exemplo com React Leaflet:

<TileLayer
  url="https://seu-geoserver.com/geoserver/workspace/wms?service=WMS&request=GetMap&layers=workspace:layer&format=image/png&transparent=true&version=1.1.1&srs=EPSG:3857&width=256&height=256"
  attribution="&copy; GeoServer"
/>

8️⃣ Automatizar o Pipeline

Para que tudo rode sozinho:

    Crie um cron job ou use Celery (Django) para:

        Baixar os dados via API.

        Atualizar o PostGIS.

        Forçar um reload no GeoServer (se necessário).

Exemplo de cron job no Linux:

0 * * * * /usr/bin/python3 /home/usuario/scripts/qfield_sync.py

(Executa a cada hora).
9️⃣ Segurança e Performance

    Use um usuário somente leitura no PostGIS para o WebGIS.

    Configure cache no GeoServer ou use GeoWebCache para melhor performance.

    Se for trabalhar com grandes volumes de dados, considere particionar camadas ou usar tiles vetoriais.