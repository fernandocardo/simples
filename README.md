# Simples
Projeto simples para aprendizado de docker, docker build e docker compose utilizando o .net core

## Dockerfile
O Dockerfile tem algumas caracteristicas 

```yaml
FROM mcr.microsoft.com/dotnet/core/sdk:2.1-alpine AS build-env
WORKDIR /app

# Copiar csproj e restaurar dependencias
COPY *.csproj ./
RUN dotnet restore

# Build da aplicacao
COPY . ./
RUN dotnet publish -c Release -o out

# Build da imagem
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1-alpine AS runtime
ENV ASPNETCORE_URLS=http://+:5000
WORKDIR /app
COPY --from=build-env /app/out .
EXPOSE  5000
ENTRYPOINT ["dotnet", "simples.dll"]
```
