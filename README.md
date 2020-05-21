# Simples
Projeto simples para aprendizado de docker, docker build e docker compose utilizando o .net core

## Dockerfile
O Dockerfile utilizando para .net core tem algumas peculiaridades:

O Build do Dockerfile deve ser realizado em 3 fases :

1. A partir de uma imagem contendo o SDK, copie o .csproj para o container e faça o restore das dependencias:

```yaml
# PARTE 1: BUILD
FROM mcr.microsoft.com/dotnet/core/sdk:2.1-alpine AS build-env
WORKDIR /app

# Copiar csproj e restaurar dependencias
COPY *.csproj ./
RUN dotnet restore
```
Caso tenha vários projetos dentro de uma solution, faça copia de cada um deles.

2. Rode o comando dotnet publish em modo Release

```yaml
# publish da aplicacao
COPY . ./
RUN dotnet publish -c Release -o out
```
Esse comando vai gerar o arquivo .dll executável

3. Gere a imagem final da sua aplicação, contendo apenas o Runtime (que é muito menor) e o código fonde de sua aplicação já compilado.

```yaml
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
```yaml
ENV ASPNETCORE_URLS=http://+:5000
EXPOSE  5000
```
* O entrypoint deve apontar para a DLL principal do projeto.

```yaml
ENTRYPOINT ["dotnet", "simples.dll"]
```

## Docker Save
Para compartilhar uma imagem já pronta do seu projeto, pode usar o comando `docker save`. Dessa forma será gerado um arquivo.tar com sua imagem pronta para ser carregada em outro computador, de forma privada.


`docker save simples > simples.tar`

## Docker Load
Para carregar uma imagem previamente salva, no diretório onde o arquivo .tar está, use o comando `docker load`.


`docker load < simples.tar`