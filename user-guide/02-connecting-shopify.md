# Connecting to Shopify

## Automatic Connection

CatalogNow connects to your Shopify store automatically during installation via OAuth. No manual configuration is needed.

## Permissions

CatalogNow requests the following Shopify permissions:
- **read_products** — to sync your product catalog
- **write_products** — to push AI-enriched content back to Shopify
- **read_metaobjects** — to resolve taxonomy category names and metafield display values

## Product Sync

### Manual Sync
1. Go to **Products**
2. Click **Sync from Shopify**
3. All products are imported/updated

### Auto-Sync
Enable auto-sync in **Settings** to automatically import new products when they're created in Shopify.

### Webhook Updates
When products are updated directly in Shopify, CatalogNow marks them as "Dirty" (out of sync) so you know which products need attention.

## Sync Status

Each product has a sync status:
- **SYNCED** — matches Shopify data
- **DIRTY** — changed in CatalogNow but not yet pushed to Shopify, or changed in Shopify but not yet pulled
- **ERROR** — sync failed
