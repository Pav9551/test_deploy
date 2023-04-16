# test_deploy
## Это инструкция по развертыванию Web-сервиса nocodb и тестирование его из скрипта на python .
#### Сервис nocodb создает веб-приложение для управления базой данных и API для предоставления доступа сторонних приложений
```
```
- Сервис позволяет с помощью веб-интерфейса работать с базой данных;
- В данном случае развернута база данных postgresql;
- В сервисе реализовано REST API, принимающее на вход GET запросы согласно документации<sup>[2](#myfootnote2)</sup>.

## Оглавление

1. [Требования к операционной системе](#Требования-к-операционной-системе)
2. [Описание файла docker-compose.yaml](#Описание-файла-docker-compose.yaml)
3. [Установка веб-сервиса](#Установка-веб-сервиса)
4. [Пример использования](#Пример-использования)

## Требования к операционной системе
Тестирование сервиса проводилось на операционной системе ubuntu 20.04 установленной на виртуальном выделенном сервере. Перед началом работы необходимо установить на операционную систему docker и docker-compose<sup>[1](#myfootnote1)</sup>
## Описание файла docker-compose.yaml

```yaml
version: '2.1'
services: 
  nocodb: 
    depends_on: 
      root_db: 
        condition: service_healthy
    environment: 
      NC_DB: "pg://root_db:5432?u=postgres&p=password&d=root_db"
    image: "nocodb/nocodb:latest"
    ports: 
      - "8100:8080"
    restart: always
    volumes: 
      - "nc_data:/usr/app/data"
  root_db: 
    environment: 
      POSTGRES_DB: root_db
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
    healthcheck: 
      interval: 10s
      retries: 10
      test: "pg_isready -U \"$$POSTGRES_USER\" -d \"$$POSTGRES_DB\""
      timeout: 2s
    image: postgres:14.0-alpine
    restart: always
    volumes: 
      - "db_data:/var/lib/postgresql/data"
volumes: 
  db_data: {}
  nc_data: {}
```
## Сборка контейнеров

 - содержимое скрипта restart.sh
```curl
docker-compose down
docker-compose build
docker-compose up -d 
```
 - сделать файл restart.sh исполняемым:
```curl 
 sudo chmod +x restart.sh
 ```
 - Собрать и запустить контейнеры скриптом ./restart.sh

 - и проверить их работу
```curl
docker-compose ps
```


 - убедиться, что подняты сервисы согласно документу docker-compose.yaml.

## Пример использования
Чтобы протестировать веб-сервис необходимо отправить GET запрос. Для этого необходимо знать IP адрес хоста, порт и шаблон запроса:

```Python
#Python
#
!pip install nocodb
from nocodb.nocodb import NocoDBProject, APIToken, JWTAuthToken
from nocodb.filters import InFilter, EqFilter
from nocodb.infra.requests_client import NocoDBRequestsClient

# Usage with API Token
client = NocoDBRequestsClient(
        # Your API Token retrieved from NocoDB conf
        APIToken("mX5Cb4bSC7****************"),
        # Your nocodb root path
        "http://212.***.***.**:8100"
)

project = NocoDBProject(
        "noco", # org name. noco by default
        "my_project_car" # project name. Case sensitive!!
)
table_name = "Sheet-1"
table_rows = client.table_row_list(project, table_name)
print(table_rows)
#http://localhost:8080/api/v1/db/data/{orgs}/{projectName}/{tableName}

```
<a name="myfootnote1">1</a> Информация по установке сервисов docker и docker-compose взята с сайта https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ru и https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04 (step1)
```
```
<a name="myfootnote2">2</a> Информация по работе с API для nocodb взята с сайта https://apis.nocodb.com/ и https://giters.com/elchicodepython/python-nocodb: