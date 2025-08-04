

1️⃣ Criar Token pela API de Login

O QField Cloud possui endpoint para autenticação e obtenção de token temporário (JWT).
Você pode fazer isso do seu computador local sem precisar acessar o servidor.
Exemplo com curl

curl -X POST https://collect.seudominio.com/api/v1/auth/token/ \
  -H "Content-Type: application/json" \
  -d '{"username": "seu_usuario", "password": "sua_senha"}'

Se estiver correto, você receberá:

{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}

    Esse token já pode ser usado no Authorization: Bearer.

Exemplo com Python

import requests

url = "https://collect.seudominio.com/api/v1/auth/token/"
payload = {"username": "seu_usuario", "password": "sua_senha"}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Token:", response.json()["token"])
else:
    print("Erro:", response.status_code, response.text)



## --------------------

✅ Etapas para Diagnóstico

Você tentou autenticar via:

url = "https://collect.eccondata.com/api/v1/auth/token/"

Mas recebeu erro. Precisamos verificar algumas possibilidades:
1️⃣ Verificar se o endpoint está habilitado

A rota /api/v1/auth/token/ só funciona se a autenticação via JWT ou Token estiver habilitada na sua instância do QField Cloud. Em muitas instalações self-hosted, essa rota não vem habilitada por padrão.

🔎 Teste no navegador:
Acesse:

https://collect.eccondata.com/api/v1/auth/token/

Se aparecer:

{"detail":"Method \"GET\" not allowed."}

➡️ O endpoint existe ✅

Se der erro 404 ou 500:
➡️ O endpoint não está habilitado ❌
2️⃣ Verifique as credenciais

Confirme se:

    O username está correto (lucasmarcone) — sensível a maiúsculas/minúsculas.

    A senha está correta (sem espaços acidentais).

Tente autenticar manualmente no QField Cloud com essas credenciais:
👉 https://collect.eccondata.com/
3️⃣ Resposta completa do erro

Por favor, me diga qual foi exatamente a resposta do .text retornado no erro. Isso ajuda a identificar se é:

    erro de autenticação ({"non_field_errors":["..."]}),

    erro de endpoint (404),

    erro de permissões (403),

    erro do backend (500).


Perfeito, Lucas — essa resposta:

"detail": "Method 'GET' not allowed."

significa que o endpoint existe, mas ele só aceita POST (o que está correto).
Ou seja, a URL está funcionando ✅, mas a sua requisição anterior provavelmente teve algum problema de autenticação (login/senha ou payload).
🔧 Ajuste do Código

Aqui está um exemplo correto em Python:

import requests

url = "https://collect.eccondata.com/api/v1/auth/token/"
payload = {
    "username": "lucasmarcone",
    "password": "Weqe2323!"  # confirme se a senha está correta
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Token:", response.json().get("token"))
else:
    print("Erro:", response.status_code, response.text)

✅ Checklist de Debug

    Username e senha corretos
    → Teste no login web: https://collect.eccondata.com/

    Requisição POST (seu teste anterior pode ter sido GET acidentalmente)

    Resposta do servidor

        Se vier 401 Unauthorized → credenciais inválidas.

        Se vier 403 Forbidden → usuário sem permissão.

        Se vier 200 OK → token gerado com sucesso.

## --------------------


1️⃣ Testar o token

Verifique se ele funciona para acessar a API de projetos:

curl -H "Authorization: Bearer SEU_TOKEN" https://collect.eccondata.com/api/v1/projects/

Se tudo estiver certo, você receberá um JSON com seus projetos.

Exemplo de resposta:

[
  {
    "id": "12345",
    "name": "Projeto Manguezais",
    "description": "Coleta de dados de campo",
    "owner": "lucasmarcone",
    ...
  }
]

2️⃣ Obter os dados do projeto

Depois, você pode listar os arquivos sincronizados com esse projeto:

curl -H "Authorization: Bearer SEU_TOKEN" https://collect.eccondata.com/api/v1/projects/12345/files/

Isso retornará algo como:

[
  {
    "id": "file_1",
    "name": "camada_coleta.gpkg",
    "download_url": "https://collect.eccondata.com/api/v1/projects/12345/files/camada_coleta.gpkg"
  }
]

3️⃣ Baixar os dados

Agora, você pode baixar diretamente o arquivo (ex.: um GeoPackage) usando o token:

curl -H "Authorization: Bearer SEU_TOKEN" -o camada_coleta.gpkg \
https://collect.eccondata.com/api/v1/projects/12345/files/camada_coleta.gpkg

Ou com Python:

import requests

token = "SEU_TOKEN"
project_id = "12345"
file_url = f"https://collect.eccondata.com/api/v1/projects/{project_id}/files/"

headers = {"Authorization": f"Bearer {token}"}
response = requests.get(file_url, headers=headers)

print(response.json())

4️⃣ Conectar no WebGIS

Agora você pode:

    Ler o arquivo .gpkg ou .geojson baixado no backend Django e servir como endpoint para o frontend (Leaflet, OpenLayers, MapLibre).

    Criar um processo automático para buscar atualizações do QField Cloud.

    Ou até mesmo disponibilizar os arquivos diretamente via GeoServer usando a API para sincronização.


## --------------------

## Já testado por mim e funcionando

https://collect.eccondata.com.br/api/v1/projects/5b61ddb8-826d-4d58-969a-01c0154afe33/

https://collect.eccondata.com.br/api/v1/auth/token/

## --------------------

Excelente, Lucas! ✅ Isso confirma que o token está funcionando e você conseguiu acessar os dados do projeto via API.

Agora você pode avançar para baixar os dados coletados e integrá-los no seu WebGIS.
1️⃣ Listar os arquivos do projeto

Para ver todos os arquivos (camadas coletadas) dentro do projeto:

curl -H "Authorization: Bearer SEU_TOKEN" \
https://collect.eccondata.com.br/api/v1/projects/5b61ddb8-826d-4d58-969a-01c0154afe33/files/

Resposta esperada:

[
  {
    "id": "f1",
    "name": "dados_campo.gpkg",
    "download_url": "https://collect.eccondata.com.br/api/v1/projects/5b61ddb8-826d-4d58-969a-01c0154afe33/files/dados_campo.gpkg"
  },
  {
    "id": "f2",
    "name": "fotos.zip",
    "download_url": "..."
  }
]

2️⃣ Baixar o arquivo GPKG (ou GeoJSON)

curl -H "Authorization: Bearer SEU_TOKEN" \
-o dados_campo.gpkg \
https://collect.eccondata.com.br/api/v1/projects/5b61ddb8-826d-4d58-969a-01c0154afe33/files/dados_campo.gpkg

Agora você terá os dados de campo localmente.
3️⃣ Automatizar com Python

import requests

token = "SEU_TOKEN"
project_id = "5b61ddb8-826d-4d58-969a-01c0154afe33"
base_url = f"https://collect.eccondata.com.br/api/v1/projects/{project_id}/files/"
headers = {"Authorization": f"Bearer {token}"}

# Lista arquivos
files = requests.get(base_url, headers=headers).json()

for file in files:
    if file["name"].endswith(".gpkg"):
        download_url = file["download_url"]
        r = requests.get(download_url, headers=headers)
        with open(file["name"], "wb") as f:
            f.write(r.content)
        print(f"Arquivo {file['name']} baixado com sucesso!")

4️⃣ Conectar ao WebGIS

Agora você tem duas opções:
A) Servir os dados direto como GeoJSON

    Converta o GPKG para GeoJSON (via ogr2ogr ou Python com geopandas).

    Exponha via endpoint Django/Flask para o frontend React ou Leaflet.

B) Inserir no PostGIS

    Carregue o GPKG no banco via ogr2ogr:

    ogr2ogr -f "PostgreSQL" PG:"dbname=seubanco user=postgres password=123" dados_campo.gpkg

    Seu WebGIS pode então ler direto do PostGIS via GeoServer ou Django ORM.

Quer que eu te monte um script automático que:
✅ Baixa os arquivos do QField Cloud →
✅ Converte para GeoJSON →
✅ Atualiza direto no PostGIS →
✅ Fica pronto para consumo no WebGIS? (sem intervenção manual)


## --------------------


Beleza, Lucas! Vamos montar um fluxo para servir diretamente os dados do QField Cloud como GeoJSON, sem precisar fazer etapas manuais.

A ideia é:

    Baixar os arquivos GPKG do QField Cloud usando o token.

    Converter para GeoJSON.

    Criar um endpoint Django para servir esse GeoJSON para seu WebGIS (Leaflet ou OpenLayers).

1️⃣ Baixar os dados do QField Cloud

Podemos criar um script Python dentro do Django para fazer isso automaticamente.
Exemplo de script qfield_sync.py

import requests
import geopandas as gpd
from io import BytesIO
import os

QFIELD_URL = "https://collect.eccondata.com.br"
TOKEN = "SEU_TOKEN"
PROJECT_ID = "5b61ddb8-826d-4d58-969a-01c0154afe33"

def baixar_dados_qfield():
    headers = {"Authorization": f"Bearer {TOKEN}"}
    
    # Lista arquivos do projeto
    files_url = f"{QFIELD_URL}/api/v1/projects/{PROJECT_ID}/files/"
    response = requests.get(files_url, headers=headers)
    response.raise_for_status()
    files = response.json()

    for file in files:
        if file["name"].endswith(".gpkg"):
            download_url = file["download_url"]
            print(f"Baixando: {file['name']}")
            r = requests.get(download_url, headers=headers)
            r.raise_for_status()

            # Lê diretamente em memória com GeoPandas
            gpkg = BytesIO(r.content)
            gdf = gpd.read_file(gpkg, driver="GPKG")

            # Converte para GeoJSON
            geojson_path = f"data/{file['name'].replace('.gpkg', '.geojson')}"
            os.makedirs("data", exist_ok=True)
            gdf.to_file(geojson_path, driver="GeoJSON")
            print(f"GeoJSON salvo em {geojson_path}")
            return geojson_path

    raise Exception("Nenhum arquivo GPKG encontrado no projeto.")

Esse script:
✅ Autentica no QField Cloud
✅ Baixa o GPKG
✅ Converte para GeoJSON
2️⃣ Criar uma View no Django para servir o GeoJSON

Dentro do seu app Django (exemplo: webgis/views.py):

from django.http import JsonResponse
import json

def geojson_view(request):
    with open("data/dados_campo.geojson", "r", encoding="utf-8") as f:
        data = json.load(f)
    return JsonResponse(data, safe=False)

3️⃣ Configurar a URL no Django

No urls.py do app:

from django.urls import path
from .views import geojson_view

urlpatterns = [
    path('api/geojson/', geojson_view, name='geojson'),
]

Agora você pode acessar:

http://localhost:8000/api/geojson/

E vai obter o GeoJSON atualizado.
4️⃣ Exibir no WebGIS (Leaflet)

No seu frontend (React ou HTML simples):

import L from "leaflet";

const map = L.map("map").setView([-15, -47], 5);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png").addTo(map);

fetch("http://localhost:8000/api/geojson/")
  .then(res => res.json())
  .then(data => {
    L.geoJSON(data).addTo(map);
  });

✅ Resultado:

    Sempre que você rodar o script qfield_sync.py, ele vai baixar e atualizar o GeoJSON automaticamente.

    O endpoint Django vai servir esse GeoJSON direto para o seu WebGIS.