# Context

This repo contains a .NET 7 WebAPI that has been created using the latest .NET SDK using the `webapi` template

```bash
# Create project
dotnet new webapi -n MutliArch.Sample -o ./multi-arch-sample
```

## Dockerfile

The `Dockerfile` and `.dockerignore` have been created using the official Docker Extension for VSCode. (`[CMD]+Shift+P` -> `Docker: Add Dockerfiles to Workspace` -> `.NET: ASP.NET Core` -> `Linux` -> `8080` -> `No Docker Compose`)

# Development machine

- MacBook Pro (M1) ARM64

Docker Version:
```
Client:
 Cloud integration: v1.0.29
 Version:           20.10.21
 API version:       1.41
 Go version:        go1.18.7
 Git commit:        baeda1f
 Built:             Tue Oct 25 18:01:18 2022
 OS/Arch:           darwin/arm64
 Context:           desktop-linux
 Experimental:      true

Server: Docker Desktop 4.14.1 (91661)
 Engine:
  Version:          20.10.21
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.7
  Git commit:       3056208
  Built:            Tue Oct 25 17:59:41 2022
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.6.9
  GitCommit:        1c90a442489720eec95342e1789ee8a5e1b9536f
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Building the ARM64 Image

Building the ARM64 image is not a problem at all. I can simple run `docker build . -t test:0.0.1` and get my ASP.NET Core application containerized in a few secs.



### Building the AMD64 Image using BuildX

Create a new Builder:

```bash
docker buildx create samplebuilder --bootstrap --use
```

Verify that the builder is able to build for `linux/amd64`(Receiving one as result means that the current BuildX builder supports `linux/amd64`):

```bash
docker buildx inspect | grep linux/amd64 | wc -
    1
```

Try to build the image:

```bash
buildx build --platform linux/amd64 . -t test:0.0.2

WARNING: No output specified with docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
[+] Building 225.7s (12/18)
 => [internal] load .dockerignore                                                   0.0s
 => => transferring context: 383B                                                   0.0s
 => [internal] load build definition from Dockerfile                                0.0s
 => => transferring dockerfile: 923B                                                0.0s
 => [internal] load metadata for mcr.microsoft.com/dotnet/sdk:7.0                   0.1s
 => [internal] load metadata for mcr.microsoft.com/dotnet/aspnet:7.0                0.1s
 => [build 1/7] FROM mcr.microsoft.com/dotnet/sdk:7.0@sha256:c99629b03411c7b0aa532  0.0s
 => => resolve mcr.microsoft.com/dotnet/sdk:7.0@sha256:c99629b03411c7b0aa5324abd01  0.0s
 => [internal] load build context                                                   0.0s
 => => transferring context: 700B                                                   0.0s
 => [base 1/3] FROM mcr.microsoft.com/dotnet/aspnet:7.0@sha256:b017897d7702fd57ce8  0.0s
 => => resolve mcr.microsoft.com/dotnet/aspnet:7.0@sha256:b017897d7702fd57ce869914  0.0s
 => CACHED [base 2/3] WORKDIR /app                                                  0.0s
 => CACHED [base 3/3] RUN adduser -u 5678 --disabled-password --gecos "" appuser &  0.0s
 => CACHED [final 1/2] WORKDIR /app                                                 0.0s
 => CACHED [build 2/7] WORKDIR /src                                                 0.0s
 => CACHED [build 3/7] COPY [MutliArch.Sample.csproj, ./]                           0.0s
 => [build 4/7] RUN dotnet restore "MutliArch.Sample.csproj"                      225.6s

 ```

 I cancelled the build process when `dotnet restore` stuck for 250secs. However, I had tests running a couple of weeks ago... Back then I cancelled after 17 mins.

 ## GitHub Actions

 Same issue can be reproduced in GitHub Actions
