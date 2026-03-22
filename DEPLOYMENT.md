# CatalogNow — Production Deployment Guide

Deploy CatalogNow to a VPS (Ubuntu) with Docker, Nginx, and Let's Encrypt SSL.

**Target domain**: `catalognow-store-app.sumvec.com`
**Architecture**: 3 Docker containers (Remix app + FastAPI AI Core + PostgreSQL) behind Nginx reverse proxy.

---

## Prerequisites

- Ubuntu VPS with 4GB+ RAM (e.g., Hostinger KVM 4)
- Docker and Docker Compose installed
- Nginx installed
- Domain DNS access for `sumvec.com`
- Shopify Partner Dashboard access

---

## Step 1: DNS Setup

Add an A record in your domain registrar:

```
Type: A
Name: catalognow-store-app
Value: <YOUR_VPS_IP>
TTL: 300
```

Verify propagation:
```bash
dig catalognow-store-app.sumvec.com +short
```

---

## Step 2: Add Swap (recommended for Docker builds)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Step 3: Clone Repository

```bash
cd /var/www
git clone <YOUR_REPO_URL> catalognow
```

---

## Step 4: Create Production .env

Generate secrets:
```bash
# Database password
openssl rand -hex 16

# AI Core secret
openssl rand -hex 32

# Encryption key
openssl rand -hex 32
```

Create the `.env` file:
```bash
cat > /var/www/catalognow-store-app/.env << 'EOF'
# Database
POSTGRES_PASSWORD=<generated-db-password>

# Shopify (from Partner Dashboard → App → API credentials)
SHOPIFY_API_KEY=<your-api-key>
SHOPIFY_API_SECRET=<your-api-secret>
SHOPIFY_APP_URL=https://catalognow-store-app.sumvec.com

# Security
AI_CORE_SECRET=<generated-ai-secret>
ENCRYPTION_KEY=<generated-encryption-key>

# OpenRouter (optional — each shop can set its own key in app settings)
# OPENROUTER_API_KEY=sk-or-v1-...
# OPENROUTER_MODEL=stepfun/step-3.5-flash:free
EOF

chmod 600 /var/www/catalognow-store-app/.env
```

---

## Step 5: Install Nginx Config

```bash
sudo cp /var/www/catalognow-store-app/scripts/nginx-catalognow.conf /etc/nginx/sites-available/catalognow-store-app.sumvec.com.conf
sudo ln -s /etc/nginx/sites-available/catalognow-store-app.sumvec.com.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 6: SSL with Let's Encrypt

```bash
sudo certbot --nginx -d catalognow-store-app.sumvec.com
```

This will automatically update the Nginx config with SSL certificates.

---

## Step 7: Build & Start Docker Containers

```bash
cd /var/www/catalognow-store-app
docker compose up -d --build
```

Run initial database migrations:
```bash
docker compose exec app npx prisma migrate deploy
```

Check all containers are running:
```bash
docker compose ps
```

---

## Step 8: Firewall

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

---

## Step 9: Shopify Partner Dashboard

Update these URLs in [Partner Dashboard](https://partners.shopify.com) → Apps → CatalogNow → Configuration:

| Setting | Value |
|---------|-------|
| App URL | `https://catalognow-store-app.sumvec.com` |
| Allowed redirection URL(s) | `https://catalognow-store-app.sumvec.com/api/auth` |
| Customer data request endpoint | `https://catalognow-store-app.sumvec.com/webhooks/customers/data_request` |
| Customer data erasure endpoint | `https://catalognow-store-app.sumvec.com/webhooks/customers/redact` |
| Shop data erasure endpoint | `https://catalognow-store-app.sumvec.com/webhooks/shop/redact` |

---

## Step 10: Deploy App Config

Run locally to sync webhooks and scopes to Shopify:
```bash
shopify app deploy
```

---

## Verification

```bash
# 1. App responds
curl -I https://catalognow-store-app.sumvec.com

# 2. AI Core port is NOT exposed externally
curl https://catalognow-store-app.sumvec.com:8100  # should fail

# 3. All containers running
docker compose ps

# 4. View logs
docker compose logs -f app
docker compose logs -f ai-core
```

Then install the app on a dev store and verify:
- OAuth flow completes
- App loads in Shopify admin
- Product sync works
- AI enrichment works

---

## Redeployment

After pushing changes to the repo:

```bash
cd /var/www/catalognow-store-app
./scripts/deploy.sh
```

Or manually:
```bash
git pull origin main
docker compose up -d --build
docker compose exec app npx prisma migrate deploy
```

---

## Troubleshooting

**Container won't start**: Check logs with `docker compose logs <service>`

**Database connection errors**: Ensure the `POSTGRES_PASSWORD` in `.env` matches what was used when the volume was first created. To reset: `docker compose down -v` (destroys data) then `docker compose up -d --build`.

**502 Bad Gateway**: The app container isn't ready yet. Wait 30s and retry, or check `docker compose logs app`.

**SSL certificate renewal**: Certbot auto-renews via systemd timer. Verify with `sudo certbot renew --dry-run`.
