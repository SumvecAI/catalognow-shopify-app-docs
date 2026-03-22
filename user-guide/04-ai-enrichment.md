# AI Enrichment

## Overview

CatalogNow uses AI agents to enrich your product catalog. Each agent specializes in a different task. Results are powered by OpenRouter, routing to free or paid LLM models depending on your configuration.

## Available AI Actions

### Generate Description
Creates a compelling product description from the title and existing data.
- Input: Product title, existing description, category
- Output: HTML-formatted product description in a professional tone

### Generate SEO Content
Creates SEO-optimized metadata for better search engine visibility.
- Input: Product title, description, category
- Output: SEO title (50-60 chars), meta description (140-155 chars), focus keywords

### Analyze Image
Uses vision AI to extract product attributes from images.
- Input: Product image URL
- Output: Detected colors, materials, style, category suggestions, image-product match verification

### Auto-Categorize
Classifies products into a hierarchical taxonomy and generates SEO metadata.
- Input: Product title, description
- Output: Category path with confidence score, SEO title, SEO description, and up to 10 tags

### Quality Score
Assesses the completeness and quality of product content.
- Input: All product fields
- Output: Score (0-100) with breakdown by category

### Full AI Enrichment
Runs the full enrichment pipeline which orchestrates all agents sequentially.
- Pipeline: Taxonomy → SEO → Description → Vision (if image available)
- Input: All available product data
- Output: Combined results — category, tags, SEO fields, description, vision attributes

## Using AI Enrichment

### Single Product
1. Go to **Products** → click a product
2. In the **AI Enrichment** panel (right side), click an action button
3. Wait for the AI to process (may take 10-60 seconds)
4. Results auto-populate the form fields and appear as **Pending Suggestions**

> **Note:** If a product is missing its description, category, and image, a warning banner will appear in the AI Enrichment panel. While enrichment will still work, results are more accurate when the product has existing data for the AI to work with. Consider adding basic product information before running enrichment.

### Batch Operations
1. Go to **Batch Operations**
2. Select products
3. Choose an action
4. Click **Run**

A backup is automatically created before any batch operation.

## Reviewing Suggestions

AI suggestions appear in the **Pending Suggestions** section on each product page. Each suggestion shows:
- **Field badge** — which field the suggestion applies to (e.g., seoTitle, category, tags)
- **Agent name** — which AI action generated it (e.g., generate_seo, categorize)
- **Suggested value** — truncated preview of the AI-generated content

For each suggestion you can:
- **Accept** — applies the suggestion to the product, creates a version snapshot, syncs to Shopify, and marks the optimization as accepted
- **Dismiss** — rejects the suggestion without applying it

Suggestions are also available in the **Enrichment Queue** page for bulk review:
- **Accept All** — applies all pending suggestions (creates backup first)

## AI Models

CatalogNow uses OpenRouter to access AI models:
- Free models (NVIDIA Nemotron, StepFun) work immediately with no credits
- Paid models (Claude, GPT-4) require OpenRouter credits
- Configure your preferred model in **Settings**
- Each store uses its own OpenRouter API key (encrypted at rest with AES-256-GCM)

The default model pool uses automatic fallback — if the first model fails or is rate-limited, the next model in the pool is tried automatically.

## How Results Are Processed

All AI results are structured and parsed automatically:
- **Auto-Categorize** returns a category path, confidence score, SEO title (max 70 chars), SEO description (max 160 chars), and up to 10 tags
- **Generate SEO Content** returns an SEO title, meta description, and focus keywords
- **Full Enrichment** combines taxonomy, SEO, description, and vision results in a single operation
- Results appear as Pending Suggestions on the product page for your review before being applied
- Accepted suggestions create a version history entry and sync changes to Shopify automatically
