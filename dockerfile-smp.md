# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# Base runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

COPY ["src/KwikNesta.Infrastruture.Svc.API/KwikNesta.Infrastruture.Svc.API.csproj", "KwikNesta.Infrastruture.Svc.API/"]
COPY ["src/KwikNesta.Infrastruture.Svc.Application/KwikNesta.Infrastruture.Svc.Application.csproj", "KwikNesta.Infrastruture.Svc.Application/"]
COPY ["src/KwikNesta.Infrastruture.Svc.Domain/KwikNesta.Infrastruture.Svc.Domain.csproj", "KwikNesta.Infrastruture.Svc.Domain/"]
COPY ["src/KwikNesta.Infrastruture.Svc.Infrastructure/KwikNesta.Infrastruture.Svc.Infrastructure.csproj", "KwikNesta.Infrastruture.Svc.Infrastructure/"]
COPY ["src/KwikNesta.Infrastruture.Svc.Worker/KwikNesta.Infrastruture.Svc.Worker.csproj", "KwikNesta.Infrastruture.Svc.Worker/"]

RUN dotnet restore "KwikNesta.Infrastruture.Svc.API/KwikNesta.Infrastruture.Svc.API.csproj"

COPY src/ .
WORKDIR "/src/KwikNesta.Infrastruture.Svc.API"
RUN dotnet build "KwikNesta.Infrastruture.Svc.API.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "KwikNesta.Infrastruture.Svc.API.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "KwikNesta.Infrastruture.Svc.API.dll"]