# Backup & Restore

## Overview

CatalogNow protects your product data with automatic and manual backups.

## Automatic Backups

Created automatically before:
- Batch enrichment operations
- Batch accept of AI suggestions
- Backup restore operations

Automatic backups are labeled with the operation that triggered them.

## Manual Backups

1. Go to **Settings** → **Backup & Restore**
2. Click **Create Backup**
3. Choose type (Full or Products Only)
4. Optionally add a description
5. Click **Create**

## Restoring a Backup

1. Go to **Settings** → **Backup & Restore**
2. Find the backup you want to restore
3. Click **Restore**
4. Confirm in the dialog

A pre-restore backup is created automatically before restoring.

## Backup Types

- **FULL** — all products with images and labels
- **PRODUCTS_ONLY** — product data only
- **PRE_OPERATION** — automatic backup before destructive operations

## Product Version Rollback

In addition to shop-level backups, each product has its own version history. When you accept an AI suggestion, a version snapshot is created automatically.

**Rolling back a product:**
1. Go to the product detail page
2. Click **Rollback** on a previous version
3. The product is restored to that version's state

**What rollback does:**
- Restores all product fields (title, description, SEO, tags, category, vendor, etc.)
- Resets the AI content status to **NONE**
- Dismisses all pending AI suggestions for that product
- Marks the product as **DIRTY** so changes sync back to Shopify

This ensures a clean slate — no stale AI suggestions remain after rolling back.

## What's Included in a Backup

- Product titles, descriptions, categories
- SEO fields
- AI-generated content
- Product labels
- Image references (URLs, not files)

## Limitations

- Backups are stored in the database as JSON
- Very large catalogs (10,000+ products) may have large backup sizes
- Image files are not backed up, only URLs
