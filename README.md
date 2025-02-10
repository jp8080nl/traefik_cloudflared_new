# Traefik Reverse Proxy Setup

A Docker-based Traefik setup with Cloudflare DNS integration, automatic SSL certificates, and secure configuration.

## Setup

1. Copy `example.env` to `.env` and fill in your values:
```bash
cp example.env .env
```

2. Update the following variables in `.env`:
- `CF_API_EMAIL`: Your Cloudflare email
- `CF_API_KEY`: Your Cloudflare API key
- `DOMAIN`: Your domain name
- `TRAEFIK_BASIC_AUTH`: Generate with `echo $(htpasswd -nb user password) | sed -e s/\\$/\\$\\$/g`

3. Create the Docker network:
```bash
docker network create traefik_proxy
```

4. Start the services:
```bash
docker-compose up -d
```

## Adding New Services

To add a new service to Traefik, use these Docker labels in your service configuration:

```yaml
labels:
  # Enable Traefik for this service
  - "traefik.enable=true"
  
  # Router configuration
  - "traefik.http.routers.your-service.rule=Host(`your-service.${DOMAIN}`)"
  - "traefik.http.routers.your-service.entrypoints=websecure"
  - "traefik.http.routers.your-service.tls=true"
  - "traefik.http.routers.your-service.tls.certresolver=cloudflare"
  
  # Service configuration (if port is not 80)
  - "traefik.http.services.your-service.loadbalancer.server.port=YOUR_PORT"
  
  # Add common middlewares for security and performance
  - "traefik.http.routers.your-service.middlewares=secure-headers,compression,rate-limit"
```

### Available Middlewares

The following middlewares are pre-configured and ready to use:

1. `secure-headers`: Adds security headers
   - SSL redirect
   - HSTS
   - XSS protection
   - Content type nosniff
   - Frame options
   
2. `compression`: Enables gzip compression for responses

3. `rate-limit`: Basic rate limiting
   - 100 requests per second average
   - 50 request burst

### Example Service

Here's an example of a service configuration:

```yaml
services:
  myapp:
    image: nginx
    container_name: myapp
    networks:
      - traefik_proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.${DOMAIN}`)"
      - "traefik.http.routers.myapp.entrypoints=websecure"
      - "traefik.http.routers.myapp.tls=true"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.routers.myapp.middlewares=secure-headers,compression,rate-limit"
```

## Accessing Services

- Traefik Dashboard: https://traefik.${DOMAIN}
- Example Whoami Service: https://whoami.${DOMAIN}

## Security Features

- Automatic HTTPS redirection
- Let's Encrypt SSL certificates via Cloudflare DNS challenge
- Security headers enabled by default
- Rate limiting to prevent abuse
- Basic authentication for Traefik dashboard 