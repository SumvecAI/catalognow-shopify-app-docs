# Configuration

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `SHOPIFY_API_KEY` | Yes | Shopify app API key (auto-set by `shopify app dev`) |
| `SHOPIFY_API_SECRET` | Yes | Shopify app secret (auto-set by `shopify app dev`) |
| `SCOPES` | Yes | `read_products,write_products,read_metaobjects` |
| `SHOPIFY_APP_URL` | Yes | App URL (auto-set by `shopify app dev`) |
| `ENCRYPTION_KEY` | Yes | 64-char hex string for AES-256 encryption |
| `AI_CORE_URL` | Yes | URL of the Python AI service |
| `AI_CORE_SECRET` | Yes | Shared secret for AI Core authentication |
| `SHOP_CUSTOM_DOMAIN` | No | Custom shop domain if applicable |

## Generating an Encryption Key

```bash
openssl rand -hex 32
```

This produces a 64-character hex string (32 bytes).

## Database

Default local connection:
```
DATABASE_URL="postgresql://catalognow:catalognow_dev@localhost:5432/catalognow"
```

## AI Core Service

Default local URL:
```
AI_CORE_URL=http://localhost:8100
```

The AI Core runs as a separate Python FastAPI service on port 8100.
