# Laboratório Docker

API ASP.NET Core usada para praticar construção de imagens, execução por ambiente, health check, falha controlada e rollback.

> Execute os comandos a partir da pasta do projeto, onde estão o `Dockerfile` e o `compose.yaml`.

## Pré-requisitos

- Docker instalado e em execução.
- Docker Compose disponível pelo comando `docker compose`.

## Construção

Construa a imagem da versão inicial:

```powershell
docker build -t devops-webapi:1.0.0 .
```

A configuração do Dockerfile expõe a API na porta `8080` do contêiner.

## Execução

Execute a versão inicial localmente, publicando-a na porta `8080` do computador:

```powershell
docker run -d --name devops-api -p 8080:8080 -e APP_ENVIRONMENT=development -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

Consulte a API e confira a versão e o ambiente:

```powershell
curl http://localhost:8080/
curl http://localhost:8080/version
```

Para parar e remover o contêiner:

```powershell
docker stop devops-api
docker rm devops-api
```

## Staging

Crie a imagem usada pelos ambientes staging e production:

```powershell
docker build -t devops-webapi:1.1.0 .
```

Execute staging na porta `8081`:

```powershell
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

Valide o ambiente:

```powershell
curl http://localhost:8081/
```

## Production

Execute production na porta `8082`:

```powershell
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

Valide o ambiente:

```powershell
curl http://localhost:8082/
```

## Health Check

Consulte o endpoint de saúde:

```powershell
curl -i http://localhost:8080/health
```

**Estado atual:** o endpoint `/health` está programado para responder com HTTP `500 Internal Server Error`. Portanto, esse comando confirma a falha controlada atual; ele não representa um health check saudável.

## Nova versão

Construa uma nova tag para representar a versão `1.1.0`:

```powershell
docker build -t devops-webapi:1.1.0 .
```

Confira as imagens locais:

```powershell
docker image ls devops-webapi
```

## Falha controlada

Tente executar a tag `1.2.0` antes de construí-la. Como essa tag ainda não existe localmente, o Docker não conseguirá iniciar o contêiner (e tentará buscar a imagem no registry padrão):

```powershell
docker run -d --name devops-api-failure -p 8083:8080 devops-webapi:1.2.0
```

## Rollback

Volte para a última versão disponível, `1.1.0`, na porta `8083`:

```powershell
docker run -d --name devops-api-rollback -p 8083:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

Valide o rollback:

```powershell
curl http://localhost:8083/version
```

## Docker Compose

O arquivo `compose.yaml` configura staging na porta `8081` e production na porta `8082`, ambos usando a imagem `devops-webapi:1.1.0`. Construa essa tag antes de iniciar os serviços.

Suba os ambientes em segundo plano:

```powershell
docker compose up -d
```

Confira o estado dos serviços:

```powershell
docker compose ps
```

Encerre e remova os contêineres:

```powershell
docker compose down
```
