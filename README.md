# Simples
Projeto simples para aprendizado de docker, docker build e docker compose utilizando o .net core

## Dockerfile
O Dockerfile utilizando para .net core tem algumas peculiaridades:

O Build do Dockerfile deve ser realizado em 3 fases :

1. A partir de uma imagem contendo o SDK, copie o .csproj para o container e faça o restore das dependencias:

```dockerfile
# PARTE 1: BUILD
FROM mcr.microsoft.com/dotnet/core/sdk:2.1-alpine AS build-env
WORKDIR /app

# Copiar csproj e restaurar dependencias
COPY *.csproj ./
RUN dotnet restore
```
Caso tenha vários projetos dentro de uma solution, faça copia de cada um deles.

2. Rode o comando dotnet publish em modo Release

```dockerfile
# publish da aplicacao
COPY . ./
RUN dotnet publish -c Release -o out
```
Esse comando vai gerar o arquivo .dll executável

3. Gere a imagem final da sua aplicação, contendo apenas o Runtime (que é muito menor) e o código fonde de sua aplicação já compilado.

```dockerfile
# PARTE 2: IMAGEM FINAL
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1-alpine AS runtime
ENV ASPNETCORE_URLS=http://+:5000
WORKDIR /app
COPY --from=build-env /app/out .
EXPOSE  5000
ENTRYPOINT ["dotnet", "simples.dll"]
```
### Pontos de ação:
* Use os comandos abaixo para definir a porta onde a aplicação ficará exposta no container:
```dockerfile
ENV ASPNETCORE_URLS=http://+:5000
EXPOSE  5000
```
* O entrypoint deve apontar para a DLL principal do projeto.

```dockerfile
ENTRYPOINT ["dotnet", "simples.dll"]
```

## Docker Save
Para compartilhar uma imagem já pronta do seu projeto, pode usar o comando `docker save`. Dessa forma será gerado um arquivo.tar com sua imagem pronta para ser carregada em outro computador, de forma privada.


`docker save simples > simples.tar`

## Docker Load
Para carregar uma imagem previamente salva, no diretório onde o arquivo .tar está, use o comando `docker load`.


`docker load < simples.tar`

## Docker Compose
Docker compose lhe dá a possibilidade de subir uma aplicação com várias camadas e dependências. No nosso exemplo, subindo a aplicação que teve sua imagem previamente criada com o `docker build`.


```yaml
version: '3.1'

services:

  simples:
    image: simples
    ports:
      - 5005:5000
```
A configuração da aplicação fica no item `services`. Nessa camada você coloca todos os parametros que colocaria no comando `docker run`. Para utilizar o docker compose, use o comando abaixo:

`docker-compose up`

Abaixo o comando `docker run` equivalente ao `docker-compose` apresentado:

`docker run -p 5005:5000 simples`

Caso sua aplicação tenha integrações ou dependências, você pode incluir mais `services:`. No exemplo abaixo vamos incluir o Mysql e o Adminer:

```yaml
version: '3.1'
  
  simples:
    image: simples
    ports:
      - 5005:5000  
  
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```