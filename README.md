# Traefik with Cloudflare DNS and Let's Encrypt

This repository contains a Docker Compose configuration for running Traefik as a reverse proxy with automatic SSL certificate generation using Let's Encrypt and Cloudflare DNS.

## Prerequisites

- Docker and Docker Compose installed
- A Cloudflare account with your domain
- API token from Cloudflare with DNS permissions

## Setup Instructions

1. Clone this repository:
   ```bash
   git clone https://github.com/jp8080nl/cloudflared-and-traefik.git
   cd cloudflared-and-traefik
   ```

2. Create the external network:
   ```bash
   docker network create traefik_proxy
   ```

3. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

4. Edit the `.env` file with your credentials:
   - Add your Cloudflare email
   - Add your Cloudflare API key
   - Generate and add basic auth credentials for the dashboard:
     ```bash
     echo $(htpasswd -nb user password) | sed -e s/\\$/\\$\\$/g
     ```

5. Create the ACME JSON file for Let's Encrypt:
   ```bash
   touch data/acme.json
   chmod 600 data/acme.json
   ```

6. Start Traefik:
   ```bash
   docker-compose up -d
   ```

## Configuration

- Traefik dashboard is available at: `traefik.your-domain.com`
- All services should be on the `traefik_proxy` network
- Add labels to your containers to enable Traefik routing

## Example Service Labels

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`app.your-domain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
```

## Security Notes

- Never commit the `.env` file with real credentials
- Keep your API keys and passwords secure
- The `acme.json` file contains your SSL certificates 