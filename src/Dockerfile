##### Pruner #####
# https://blog.nimbleways.com/docker-build-caching-for-dotnet-applications-done-right-with-dotnet-subset/
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS pruner
ARG PROJECT_FOLDER
ARG PROJECT_NAME
ENV PROJECT_PATH="${PROJECT_NAME:?err}/${PROJECT_NAME:?err}.csproj"

WORKDIR /src
ENV PATH="${PATH}:/root/.dotnet/tools"
RUN dotnet tool install --global --no-cache dotnet-subset --version 0.3.2

COPY . .

# Add a debug statement to show all directorie in current directory
#RUN ls -la > /tmp/ls-la.txt && cat /tmp/ls-la.txt && ls -la RiderEAPIssue2 && exit 1

RUN dotnet subset restore ${PROJECT_PATH} --root-directory ./ --output /app
WORKDIR /app


##### Builder #####
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS builder
WORKDIR /app
ARG PROJECT_FOLDER
ARG PROJECT_NAME
ENV PROJECT_PATH="${PROJECT_NAME:?err}/${PROJECT_NAME:?err}.csproj"

COPY --from=pruner /app .
RUN  \
    --mount=type=cache,target=$HOME/.nuget/packages \
    --mount=type=cache,target=$HOME/.local/share/NuGet/v3-cache \
    --mount=type=cache,target=$HOME/.local/share/NuGet/plugins-cache \
    --mount=type=cache,target=/tmp/NuGetScratch \
    dotnet restore ${PROJECT_PATH} /p:IsDockerBuild=true 

COPY . .

RUN \ 
    --mount=type=cache,target=$HOME/.nuget/packages \
    --mount=type=cache,target=$HOME/.local/share/NuGet/v3-cache \
    --mount=type=cache,target=$HOME/.local/share/NuGet/plugins-cache \
    --mount=type=cache,target=/tmp/NuGetScratch \
    dotnet publish ${PROJECT_PATH} -c Release -o /app/publish /p:IsDockerBuild=true

##### Runner #####
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app
ARG PROJECT_NAME
ENV PROJECT_NAME=${PROJECT_NAME}
RUN apt update && apt install -y curl
COPY --from=builder /app/publish .

ENTRYPOINT dotnet ${PROJECT_NAME}.dll