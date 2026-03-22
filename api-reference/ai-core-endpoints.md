# AI Core API Reference

The AI Core is a Python FastAPI service that provides AI-powered product enrichment.

## Base URL

```
http://localhost:8100
```

## Authentication

All requests must include the `X-OpenRouter-Key` header with a valid OpenRouter API key.

Optional: `X-OpenRouter-Model` header to specify a preferred model.

## Endpoints

### Health Check

```
GET /health
```

Returns `{"status": "ok"}`.

### Generate Description

```
POST /api/enrich/description
```

**Request Body:**
```json
{
  "product_id": "product-uuid",
  "title": "Product Title",
  "description": "Current HTML description",
  "image_url": "https://example.com/image.jpg",
  "context": {
    "vendor": "Brand Name",
    "tags": "tag1, tag2",
    "metafields": {"color": "Blue", "material": "Cotton"},
    "price_range": "$19.99 - $29.99",
    "variant_info": "3 variants: S, M, L",
    "existing_seo_title": "Current SEO Title",
    "existing_seo_description": "Current meta description"
  }
}
```

**Response:**
```json
{
  "suggestions": {
    "shortDescription": "AI-generated short description",
    "bodyHtml": "<p>AI-generated HTML description</p>",
    "bulletPoints": ["Feature 1", "Feature 2"]
  },
  "agent": "description_agent",
  "reasoning": "Based on the product title and image...",
  "confidence": 0.85
}
```

### Analyze Image

```
POST /api/vision/analyze
```

**Request Body:**
```json
{
  "image_url": "https://example.com/image.jpg"
}
```

**Response:**
```json
{
  "suggestions": {
    "color": "Blue",
    "material": "Cotton",
    "style": "Casual",
    "category": "Apparel"
  },
  "agent": "vision_agent",
  "confidence": 0.78
}
```

### Categorize Product

```
POST /api/taxonomy/categorize
```

**Request Body:**
```json
{
  "title": "Product Title",
  "description": "Product description",
  "image_url": "https://example.com/image.jpg"
}
```

**Response:**
```json
{
  "suggestions": {
    "category": "Clothing",
    "subcategory": "T-Shirts",
    "gender": "Unisex",
    "ageGroup": "Adult"
  },
  "agent": "taxonomy_agent",
  "confidence": 0.92
}
```

### Quality Score

```
POST /api/quality/score
```

**Request Body:**
```json
{
  "title": "Product Title",
  "description": "Product description",
  "short_description": "Short desc",
  "image_url": "https://example.com/image.jpg",
  "category": "Clothing"
}
```

**Response:**
```json
{
  "score": 72,
  "details": {
    "title": 90,
    "description": 65,
    "images": 80,
    "seo": 45,
    "categorization": 80
  },
  "agent": "content_quality_agent"
}
```

### Full Enrichment (Supervisor)

```
POST /api/supervisor/enrich
```

Runs the complete enrichment pipeline: Taxonomy → SEO → Description → Vision (if image available).

**Request Body:**
```json
{
  "products": [{
    "id": "product-uuid",
    "title": "Product Title",
    "description": "Product description",
    "image_url": "https://example.com/image.jpg",
    "category": "Existing Category"
  }],
  "context": {
    "vendor": "Brand Name",
    "tags": "tag1, tag2",
    "metafields": {"color": "Blue"},
    "price_range": "$19.99",
    "variant_info": "2 variants"
  }
}
```

**Response:**
```json
{
  "results": {
    "category_path": "Apparel > T-Shirts",
    "seo_title": "Product Title — Brand Name | Shop Now",
    "seo_description": "High-quality product description for SEO...",
    "description": "<p>AI-generated product description</p>",
    "tags": ["tag1", "tag2", "tag3"]
  },
  "agent": "supervisor"
}
```
