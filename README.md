# CSSoftwareEngineering-Docker

## Portliste der MicroApps

|Anwendung               |Port |Anmerkung|
|------------------------|-----|---------|
|MSSQL                   |14330|         |
|                        |     |         |
|SnQTranslator-WebApi    |34785|         |
|SnQTranslator-AspMvc    |34790|         |
|                        |     |         |
|SnQMusicStore-WebApi    |34795|         |
|SnQMusicStore-AspMvc    |34800|         |
|                        |     |         |
|SnQTradingCompany-WebApi|34805|         |
|SnQTradingCompany-AspMvc|34810|         |
|                        |     |         |
|SnQHtmlStore-WebApi     |34815|         |
|SnQHtmlStore-AspMvc     |34820|         |

## Container-MSSQL

Der MSSQL-Server kann nach belieben konfiguriert werden und in einem eigenen Container gestartet werden. In diesem Scenario wird der Server Standard-Port 1433 nach außen mit 14330 gebunden. Als externes Verzeichnis ist **'C:\Share\Docker\MSSQL'** konfiguriert.

![ContainerSettings-Mssql](ContainerSettings-MSSQL.png)

Nachfolgend das Kommando für das Starten des MSSQL-Servers in einem eigenen Container:

### docker-command

> docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=passme!1234" -p 14330:1433 -v C:\Share\Docker\MSSQL:/var/opt/mssql/data -d mcr.microsoft.com/mssql/server --name mssql 

### [docker-compose](DbServer/docker-compose.yml) (DbServer)

```code
version: '3.4'

services:
  dbserver:
    image: "mcr.microsoft.com/mssql/server"
    container_name: mssql
    restart: always
    environment:
      SA_PASSWORD: "passme!1234"
      ACCEPT_EULA: "Y"
    ports:
      - "14330:1433"
    volumes:
      - "C:\\Share\\Docker\\MSSQL:/var/opt/mssql/data/"
```

## Container-SnQMusicStore

Die MicroApp **'SnQMusicStore'** wird in einem Container gestartet und interargiert mit dem Microsoft SQL-Server in einem anderen Container. Der Web-Service **'snqmusicstore.webapi'** ist mit dem Port 34795 konfiguriert und die Web-App **'snqmusicstore.aspmvc'** mit dem Port 34800. Das entsprechende Docker-compose File befindet sich nach der Abbildung.
 
![ContainerStettings-SnQMusicStore](ContainerSettings-SnQMusicStore.png)

### docker-compose (SnQMusicStore)

```code
version: '3.4'

services:
  snqmusicstore.webapi:
    image: "snqmusicstorewebapi"
    container_name: snqmusicstore.webapi
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQMusicStoreDb; User Id=sa;Password=passme!1234
    ports:
      - "34795:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
  snqmusicstore.aspmvc:
    image: "snqmusicstoreaspmvc"
    container_name: snqmusicstore.aspmvc
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQMusicStoreDb; User Id=sa;Password=passme!1234
      - ASPNETCORE_TRANSLATIONSERVER=http://host.docker.internal:34785/api
      - ASPNETCORE_STATICPAGESERVER=http://host.docker.internal:34820/api
    ports:
      - "34800:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
```
  
## SnQMusicStore with SnQTranslator

![ContainerStettings-SnQMusicStoreWithTranslator](ContainerSettings-SnQMusicStoreWithTranslator.png)

### docker-compose

```code
version: '3.4'

services:
  snqmusicstore.webapi:
    image: "snqmusicstorewebapi"
    container_name: snqmusicstore.webapi
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQMusicStoreDb; User Id=sa;Password=passme!1234
    ports:
      - "34795:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
  snqmusicstore.aspmvc:
    image: "snqmusicstoreaspmvc"
    container_name: snqmusicstore.aspmvc
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQMusicStoreDb; User Id=sa;Password=passme!1234
      - ASPNETCORE_TRANSLATIONSERVER=http://host.docker.internal:34785/api
      - ASPNETCORE_STATICPAGESERVER=http://host.docker.internal:34820/api
    ports:
      - "34800:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
```

## SnQMusicStore and SnQHtmlStore with SnQTranslator

![ContainerStettings-SnQMusicStoreWithTranslator](ContainerSettings-SnQMusicStoreAndSnQHtmlStoreWithTranslator.png)

### docker-compose

```code
version: '3.4'

services:
  snqhtmlstore.webapi:
    image: "snqhtmlstorewebapi"
    container_name: snqhtmlstore.webapi
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQHtmlStoreDb; User Id=sa;Password=passme!1234
    ports:
      - "34815:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
  snqmusicstore.aspmvc:
    image: "snqhtmlstoreaspmvc"
    container_name: snqhtmlstore.aspmvc
    restart: always
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ASPNETCORE_CONNECTIONSTRING=Server=host.docker.internal,14330; Database=SnQHtmlStoreDb; User Id=sa;Password=passme!1234
      - ASPNETCORE_TRANSLATIONSERVER=http://host.docker.internal:34785/api
      - ASPNETCORE_STATICPAGESERVER=http://host.docker.internal:34820/api
    ports:
      - "34820:80"
    volumes:
      - ${APPDATA}/Microsoft/UserSecrets:/root/.microsoft/usersecrets:ro
      - ${APPDATA}/ASP.NET/Https:/root/.aspnet/https:ro
```
