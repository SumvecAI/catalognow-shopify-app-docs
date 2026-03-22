# Batch Operations

## Overview

Process multiple products at once with AI enrichment.

## Available Batch Actions

- **Full AI Enrichment** — runs all agents on selected products
- **Auto-Categorize** — classifies selected products
- **Generate Descriptions** — creates descriptions for selected products

## How to Use

1. Go to **Batch Operations**
2. Select products using checkboxes (or "Select All")
3. Choose a batch action from the dropdown
4. Click **Run**

## Safety Features

Before any batch operation:
1. **Consent is logged** — records what you're doing and why
2. **Automatic backup** — snapshots all products before changes
3. **AI suggestions go to queue** — nothing is applied automatically
4. **You review and accept/reject** in the Enrichment Queue

## Error Handling

If an error occurs during batch operations, a friendly error message is displayed instead of a blank page. The error includes:
- A description of what went wrong
- A suggestion to refresh the page
- If the issue persists, check that the app is properly configured (API key, AI Core connection)

Products that were successfully processed before the error retain their suggestions in the Enrichment Queue.

## Limits

- Maximum 100 products per batch
- Each product is processed sequentially
- Timeout: 2 minutes per product
