FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
# Dependencies for Telerik Reporting on Linux
RUN apt-get update \
    && apt-get install -y --allow-unauthenticated \
        libc6-dev \
        libgdiplus \
        libx11-dev \
     && rm -rf /var/lib/apt/lists/*



FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
# Change work directory to the subfolder with this project
WORKDIR /src/MyAspNetCoreApp
# Copy over the code files
COPY . .
# Here we use a docker secret to update the 'Telerik_Feed' package source, then restore then build
RUN --mount=type=secret,id=telerik_key \
  echo $(cat /run/secrets/telerik_key) \
  && dotnet nuget update source "Telerik_Feed" -s "https://nuget.telerik.com/v3/index.json" -u "api-key" -p $(cat /run/secrets/telerik_key) --configfile "./NuGet.Config" --store-password-in-clear-text \
  && dotnet restore "MyAspNetCoreApp.csproj" --configfile "./NuGet.Config" \
  && dotnet build "MyAspNetCoreApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyAspNetCoreApp.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyAspNetCoreApp.dll"]
