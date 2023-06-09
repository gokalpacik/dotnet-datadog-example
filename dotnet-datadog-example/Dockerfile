#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["dotnet-datadog-example/dotnet-datadog-example.csproj", "dotnet-datadog-example/"]
RUN dotnet restore "dotnet-datadog-example/dotnet-datadog-example.csproj"
COPY . .
WORKDIR "/src/dotnet-datadog-example"
RUN dotnet build "dotnet-datadog-example.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "dotnet-datadog-example.csproj" -c Release -o /app/publish /p:UseAppHost=false

RUN TRACER_VERSION=2.15.0 && curl -Lo /tmp/datadog-dotnet-apm.deb https://github.com/DataDog/dd-trace-dotnet/releases/download/v${TRACER_VERSION}/datadog-dotnet-apm_${TRACER_VERSION}_amd64.deb

FROM base AS final

# Copy the tracer from build target
COPY --from=publish /tmp/datadog-dotnet-apm.deb /tmp/datadog-dotnet-apm.deb
# Install the tracer
RUN mkdir -p /opt/datadog \
    && mkdir -p /var/log/datadog \
    && dpkg -i /tmp/datadog-dotnet-apm.deb \
    && rm /tmp/datadog-dotnet-apm.deb

WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "dotnet-datadog-example.dll"]