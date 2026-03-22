# Product Sync

## How Sync Works

CatalogNow maintains a local copy of your products in its database. This allows AI processing without affecting your live store until you're ready.

## Syncing Products

1. Navigate to **Products**
2. Click **Sync from Shopify**
3. CatalogNow imports all products with:
   - Title, description, vendor, product type
   - All product images
   - Product variants (title, SKU, price, inventory)
   - Product metafields (namespace, key, value, type)
   - Current status

## What Gets Synced

| Field | Synced |
|-------|--------|
| Title | Yes |
| Description (HTML) | Yes |
| Vendor | Yes |
| Product Type | Yes |
| Status | Yes |
| Images | Yes |
| Variants | Yes |
| Metafields | Yes |

## Syncing Changes Back to Shopify

When you edit a product (manually or via AI enrichment), CatalogNow automatically pushes the changes back to your Shopify store. This happens when you:

- **Save** product edits on the product detail page
- **Accept** an AI enrichment suggestion
- **Accept All** pending suggestions in the enrichment queue
- **Rollback** to a previous product version

The sync status badge on each product shows whether the local copy is in sync with Shopify (`SYNCED`), has unpushed changes (`DIRTY`), or encountered an error (`ERROR`).

## Notes

- Images are referenced by URL, not downloaded
- All products are synced regardless of store size (pagination is handled automatically)
