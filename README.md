# Docker Minecraft Homelab Server

## Overview

This project documents a Minecraft server running in Docker on a self-hosted Ubuntu Server homelab machine.

The goal of the project is to learn Docker Compose, persistent volumes, container networking, and basic game server operations in a local infrastructure environment.

## Architecture

```mermaid
graph TD

    Player[Minecraft Client] --> LAN[Local Network]
    LAN --> Host[HP EliteDesk<br>Ubuntu Server]
    Host --> Docker[Docker Engine]
    Docker --> Container[PaperMC Minecraft Container]
    Container --> Volume[Persistent Data Volume<br>./data]
```

## Tech Stack

- HP EliteDesk
- Ubuntu Server
- Docker Engine
- Docker Compose
- PaperMC
- Minecraft Java Edition
- Local network access

## Setup Process

- Installed Docker Engine on Ubuntu Server
- Created a Docker Compose configuration
- Deployed a PaperMC Minecraft server container
- Exposed Minecraft port `25565`
- Mounted persistent server data using a local volume
- Configured container restart policy with `unless-stopped`
- Verified the server using Docker logs
- Connected successfully from a Minecraft client on the local network

## Docker Compose Configuration

```yaml
services:
  minecraft:
    image: itzg/minecraft-server:latest
    container_name: minecraft-server
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      VERSION: "1.21.4"
      MEMORY: "4G"
      ONLINE_MODE: "TRUE"
      MOTD: "Tobias Homelab Minecraft Server"
    volumes:
      - ./data:/data
    restart: unless-stopped
```

## Useful Commands

Start the server:

```bash
docker compose up -d
```

View live logs:

```bash
docker logs -f minecraft-server
```

View recent logs:

```bash
docker logs --tail 50 minecraft-server
```

Check running containers:

```bash
docker ps
```

Restart the server:

```bash
docker compose restart
```

Stop the server:

```bash
docker compose down
```

## Current Status

The Minecraft server is running in Docker on the homelab server and is accessible from the local network.

Local server address:

```text
192.168.1.41:25565
```

## Screenshots

The screenshots below show the Docker Compose configuration, the running Minecraft container, and a successful Minecraft client connection on the local network.

### Docker Installation Verified
![Docker Installation Verified](./screenshots/docker-installation-verified.png)

### Docker Compose Configuration
![Docker Compose Configuration](./screenshots/docker-compose-config.png)

### Docker Container Running
![Docker Container Running](./screenshots/docker-container-running.png)

### Minecraft Client Connected
![Minecraft Client Connected](./screenshots/minecraft-client-connected.png)

## Key Learnings

- Docker Compose service configuration
- Container port mapping
- Persistent volume usage for server data
- Basic container lifecycle management
- Reading container logs for troubleshooting
- Hosting a service on a self-managed Linux server

## Future Improvements

- Configure Cloudflare DNS and a custom domain
- Configure router port forwarding for external access
- Add automated backups
- Add monitoring with Prometheus and Grafana
- Add Uptime Kuma for service status monitoring
- Add server hardening and SSH key authentication
