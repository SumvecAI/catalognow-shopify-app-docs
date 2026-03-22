# CatalogNow Shopify App — Deployment Guide

## Architecture Overview

```
Internet
   │
   ▼
Nginx (port 443, SSL termination)
   │
   ▼ proxy_pass
Docker Compose
   ├── app        (Node.js 20, Remix)  → port 3000 internal, 3100 on host
   ├── ai-core    (Python 3.12, FastAPI) → port 8100 internal
   └── db         (PostgreSQL 16)       → port 5432 internal
```

- **app**: Shopify embedded Remix app (frontend + API routes)
- **ai-core**: Python AI service that calls OpenRouter LLM APIs for product enrichment
- **db**: PostgreSQL database for products, AI suggestions, audit logs

---

## Prerequisites

The VPS must have:
- **OS**: Ubuntu 22.04+ (or similar Linux)
- **Docker** and **Docker Compose** (v2+)
- **Nginx**
- **Certbot** (for Let's Encrypt SSL)
- **Git**
- **curl** (for health checks)

### Install prerequisites (Ubuntu)
```bash
# Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Nginx + Certbot
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx

# Git (usually pre-installed)
sudo apt install -y git curl
```

---

## Step 1: Clone the Repository

```bash
cd /var/www
git clone https://github.com/sourabhnow/catalognow-store-app.git catalognow-store-app
cd catalognow-store-app
```

---

## Step 2: Configure Environment Variables

```bash
cp .env.example .env
nano .env
```

Fill in the following values:

```env
# Database (keep "db" as hostname — it's the Docker service name)
DATABASE_URL="postgresql://catalognow:YOUR_STRONG_DB_PASSWORD@db:5432/catalognow"
POSTGRES_PASSWORD="YOUR_STRONG_DB_PASSWORD"

# Shopify App Credentials (from Shopify Partners dashboard)
SHOPIFY_API_KEY=983df363c48f6d7a267b94281951bf47
SHOPIFY_API_SECRET=your-shopify-api-secret-here
SHOPIFY_APP_URL=https://catalognow-store-app.sumvec.com
SCOPES=read_products,write_products,read_metaobjects

# AI Core (internal communication between containers)
AI_CORE_URL=http://ai-core:8100
AI_CORE_SECRET=generate-a-random-secret-string

# Encryption key for securing merchant API keys at rest
# Generate with: openssl rand -hex 32
ENCRYPTION_KEY=your-64-char-hex-string-here
```

### Where to find Shopify credentials
1. Go to https://partners.shopify.com
2. Navigate to **Apps** → **CatalogNow**
3. **Client ID** = `SHOPIFY_API_KEY` (already set: `983df363c48f6d7a267b94281951bf47`)
4. **Client secret** = `SHOPIFY_API_SECRET`

### Generate secure random values
```bash
# For POSTGRES_PASSWORD
openssl rand -base64 24

# For ENCRYPTION_KEY (must be 64 hex chars)
openssl rand -hex 32

# For AI_CORE_SECRET
openssl rand -base64 24
```

---

## Step 3: Set Up SSL Certificate

The domain `catalognow-store-app.sumvec.com` must point to the VPS IP address (DNS A record).

```bash
# Stop nginx temporarily if running (certbot needs port 80)
sudo systemctl stop nginx

# Get SSL certificate
sudo certbot certonly --standalone -d catalognow-store-app.sumvec.com

# Verify certificate was created
ls /etc/letsencrypt/live/catalognow-store-app.sumvec.com/
# Should show: fullchain.pem  privkey.pem
```

### Auto-renewal
Certbot sets up auto-renewal by default. Verify:
```bash
sudo certbot renew --dry-run
```

---

## Step 4: Configure Nginx

```bash
# Copy the nginx config
sudo cp /var/www/catalognow-store-app/scripts/nginx-catalognow.conf \
        /etc/nginx/sites-available/catalognow-store-app

# Enable it
sudo ln -sf /etc/nginx/sites-available/catalognow-store-app /etc/nginx/sites-enabled/

# Remove default site if it conflicts
sudo rm -f /etc/nginx/sites-enabled/default

# Test and reload
sudo nginx -t
sudo systemctl start nginx
sudo systemctl enable nginx
```

### What the nginx config does
- Redirects HTTP (80) → HTTPS (443)
- SSL termination with Let's Encrypt certs
- Proxies all traffic to `127.0.0.1:3100` (the Docker app container)
- 120-second timeouts for AI operations
- WebSocket support
- Security headers (HSTS, X-Frame-Options, etc.)

---

## Step 5: Deploy the Application

```bash
cd /var/www/catalognow-store-app
bash scripts/deploy.sh
```

### What the deploy script does
1. `git pull origin main` — pulls latest code
2. `docker compose up -d --build` — builds and starts all 3 containers
3. `docker compose exec app npx prisma migrate deploy` — runs database migrations
4. Health check — waits up to 60 seconds for app to respond
5. `docker image prune -f` — cleans up old Docker images

### First deploy takes longer
The initial build downloads Docker images and installs all dependencies. Expect 3-5 minutes. Subsequent deploys are faster due to Docker layer caching.

---

## Step 6: Verify Deployment

```bash
# Check all containers are running
docker compose ps

# Expected output:
# NAME                              STATUS
# catalognow-store-app-db-1         Up (healthy)
# catalognow-store-app-ai-core-1    Up
# catalognow-store-app-app-1        Up

# Test the app responds
curl -sf http://127.0.0.1:3100 && echo "OK" || echo "FAIL"

# Test via domain
curl -sf https://catalognow-store-app.sumvec.com && echo "OK" || echo "FAIL"
```

---

## Updating the App (Future Deploys)

After code is pushed to `main`:

```bash
cd /var/www/catalognow-store-app
bash scripts/deploy.sh
```

That's it — the script handles pull, build, migrate, and health check.

---

## Troubleshooting

### View container logs
```bash
cd /var/www/catalognow-store-app

# All services
docker compose logs --tail=100

# Specific service
docker compose logs --tail=100 app
docker compose logs --tail=100 ai-core
docker compose logs --tail=100 db

# Follow logs in real-time
docker compose logs -f app
```

### Restart services
```bash
# Restart everything
docker compose restart

# Restart just the app
docker compose restart app

# Full rebuild (if something is really broken)
docker compose down
docker compose up -d --build
```

### Database access
```bash
# Connect to PostgreSQL
docker compose exec db psql -U catalognow -d catalognow

# Run migrations manually
docker compose exec app npx prisma migrate deploy
```

### Check deploy log
```bash
cat /var/www/catalognow-store-app/deploy.log
```

### Common issues

| Issue | Cause | Fix |
|-------|-------|-----|
| `port 3100 already in use` | Another process on that port | `sudo lsof -i :3100` then stop it |
| `db health check failing` | PostgreSQL not ready | Wait, or check `docker compose logs db` |
| `prisma migrate` fails | Missing DATABASE_URL | Check `.env` file, ensure `db` hostname |
| `SSL certificate error` | DNS not pointed to VPS | Verify A record: `dig catalognow-store-app.sumvec.com` |
| `502 Bad Gateway` | App container not running | `docker compose ps`, then `docker compose logs app` |
| AI enrichment timeouts | OpenRouter API slow | Normal — external API dependency, retries help |

### Disk space
```bash
# Check disk usage
df -h

# Clean Docker artifacts if low on space
docker system prune -a
```

---

## Port Reference

| Service | Internal Port | Host Port | Access |
|---------|--------------|-----------|--------|
| App (Remix) | 3000 | 3100 (localhost only) | Via nginx |
| AI Core (FastAPI) | 8100 | Not exposed | Internal only |
| PostgreSQL | 5432 | Not exposed | Internal only |
| Nginx | 80, 443 | 80, 443 | Public |

---

## Security Notes

- The `.env` file contains secrets — never commit it to git
- PostgreSQL and AI Core are not exposed to the internet (Docker internal network only)
- The app port (3100) is bound to `127.0.0.1` — only accessible via nginx
- Merchant API keys are encrypted at rest using `ENCRYPTION_KEY`
- All traffic is HTTPS with HSTS headers
