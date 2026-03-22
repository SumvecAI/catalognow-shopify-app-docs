# Security

## API Key Storage

- OpenRouter API keys are encrypted with AES-256-GCM before storage
- Each encrypted value includes a unique IV and authentication tag
- The encryption key is stored as an environment variable, never in the database
- Keys are decrypted only in server-side code, never exposed to the client

## Shopify OAuth

- Authentication handled by `@shopify/shopify-app-remix`
- Session tokens validated on every request
- Webhook HMAC validation provided by the Shopify library

## Multi-Tenancy

- All database queries are scoped by `shopId`
- Products, settings, and logs are isolated per shop
- No cross-tenant data access is possible through the app

## GDPR Compliance

- `customers/data_request` — CatalogNow stores no customer PII
- `customers/redact` — No customer data to redact
- `shop/redact` — Deletes ALL shop data (products, settings, backups, traces, logs)

## Data Safety

- Automatic backups before destructive operations
- Consent logging for all AI and batch operations
- Product versioning with rollback capability
- All mutations logged in the audit trail

## Best Practices

- Never commit `.env` files with real credentials
- Rotate the `ENCRYPTION_KEY` by re-encrypting all stored keys
- Monitor the audit trail for unexpected activity
- Keep dependencies updated
