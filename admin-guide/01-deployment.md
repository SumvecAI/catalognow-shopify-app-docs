# Deployment Guide

## Prerequisites

- Node.js >= 20.19
- Python 3.11+
- PostgreSQL 16
- Docker & Docker Compose (recommended)
- Shopify Partner account

## Local Development

```bash
# Clone the repo
git clone https://github.com/sourabhnow/catalognow-store-app.git
cd catalognow-store-app
npm install

# Start PostgreSQL (Docker Desktop must be running)
docker compose up db -d

# Run database migrations
npx prisma migrate dev

# Start AI Core (in a separate terminal)
cd ai_core && python3 main.py

# Start Shopify dev server
shopify app dev
```

## Production Deployment

For full production deployment instructions, see the comprehensive guide:

**[Production Deployment Guide](../DEPLOYMENT_GUIDE.md)**

Key details:
- **Domain**: `catalognow-store-app.sumvec.com`
- **Deploy path**: `/var/www/catalognow-store-app`
- **Architecture**: 3 Docker containers (Remix + FastAPI + PostgreSQL) behind Nginx
- **CI/CD**: Jenkins pipeline with SSH-based deployment
- **SSL**: Let's Encrypt via Certbot

### Quick Redeploy

```bash
cd /var/www/catalognow-store-app
bash scripts/deploy.sh
```

## Shopify App Store Submission

1. Complete all verification checklist items
2. Run `shopify app deploy` locally to push config to Shopify
3. Submit for review in the Shopify Partner Dashboard
