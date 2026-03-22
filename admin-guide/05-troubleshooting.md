# Troubleshooting

## Common Issues

### "ENCRYPTION_KEY must be set"
Generate a key with `openssl rand -hex 32` and add it to your `.env` file.

### Database connection failed
1. Ensure PostgreSQL is running: `docker compose -f docker-compose.dev.yml ps`
2. Check `DATABASE_URL` in `.env`
3. Run migrations: `npx prisma migrate dev`

### AI enrichment returns errors
1. Verify your OpenRouter API key is valid in Settings
2. Check the AI Core is running: `curl http://localhost:8100/health`
3. Check AI Core logs: `docker compose -f docker-compose.dev.yml logs ai-core`

### Products not syncing
1. Ensure the app has `write_products` and `read_products` permissions
2. Try re-authenticating by visiting `/auth/login`
3. Check for GraphQL errors in the server logs

### Webhook not firing
1. Webhooks are registered by `shopify app dev` automatically
2. Check `shopify.app.toml` for correct webhook URIs
3. Verify with `shopify app webhooks list`

### "Shop not found" errors
The Shop record is created on first visit to Settings. Navigate to Settings first.

## Resetting Local Development

```bash
# Reset database
npx prisma migrate reset

# Restart services
docker compose -f docker-compose.dev.yml down
docker compose -f docker-compose.dev.yml up -d

# Re-run dev
shopify app dev
```

## Previously Fixed Issues

These issues have been resolved but are documented for reference:

### Pagination resets to page 1 when searching
**Symptom:** Clicking Next/Previous on the Products page would reset to page 1 after the debounce timer fired.
**Root cause:** The debounced search `useEffect` depended on `handleSearch` which was recreated on every `searchParams` change.
**Fix:** Added a `userTypingRef` guard so the debounce only fires on actual user keyboard input.

### Products show stale data after rollback
**Symptom:** After rolling back a product, AI content status remained "ENRICHED" and pending suggestions were still visible.
**Root cause:** Rollback only restored field values but didn't reset the AI status or dismiss pending suggestions.
**Fix:** Rollback now resets `aiContentStatus` to NONE, dismisses all pending suggestions, and uses explicit null coalescing for nullable fields.

### Batch operations show blank page on error
**Symptom:** If a batch operation fails, a blank "Application Error" page appears.
**Fix:** Added an `ErrorBoundary` component that shows a descriptive error message.

## Production Debugging

### SSH into the VPS
```bash
ssh deploy@<VPS_IP>
```

### Check Docker container logs
```bash
cd /var/www/catalognow-store-app
docker compose logs --tail=100 app
docker compose logs --tail=100 ai-core
docker compose logs --tail=100 db
```

### Check Jenkins build status
Review the Jenkins dashboard for recent build and deployment logs.

## Getting Help

- Check the audit trail for detailed error context
- File issues at https://github.com/sourabhnow/catalognow-store-app/issues
