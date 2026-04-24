---
name: strapi-v5
description: Strapi v5 REST API patterns using flattened documentId format. Use this when working with Strapi v5 projects or APIs.
---

# Strapi v5 Skill

## Response Format

Strapi v5 uses a **flattened response format** with `documentId` as the primary identifier.

### Single Entry Response

```json
{
  "data": {
    "id": 1,
    "documentId": "abc123def456",
    "title": "My Article",
    "slug": "my-article",
    "createdAt": "2025-01-01T00:00:00.000Z",
    "updatedAt": "2025-01-01T00:00:00.000Z",
    "publishedAt": "2025-01-01T00:00:00.000Z"
  },
  "meta": {}
}
```

### Collection Response

```json
{
  "data": [
    {
      "id": 1,
      "documentId": "abc123def456",
      "title": "My Article",
      "slug": "my-article"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 25,
      "pageCount": 1,
      "total": 1
    }
  }
}
```

## Key Rules

- **Use `documentId`** (string) for all CRUD operations — not numeric `id`
- Fields are **flat on `data`** — no nested `attributes` wrapper
- Relations are embedded directly when populated

## REST API Patterns

### Fetch by documentId

```
GET /api/articles/{documentId}
```

### Filter

```
GET /api/articles?filters[title][$eq]=My Article
```

### Populate Relations

```
GET /api/articles?populate=category
GET /api/articles?populate[category][fields][0]=name
```

### Create

```
POST /api/articles
Content-Type: application/json

{
  "data": {
    "title": "New Article",
    "slug": "new-article"
  }
}
```

### Update

```
PUT /api/articles/{documentId}
Content-Type: application/json

{
  "data": {
    "title": "Updated Title"
  }
}
```

### Delete

```
DELETE /api/articles/{documentId}
```

## Common Filters

| Operator | Syntax | Description |
|----------|--------|-------------|
| `$eq` | `filters[field][$eq]=value` | Equal |
| `$ne` | `filters[field][$ne]=value` | Not equal |
| `$in` | `filters[field][$in][0]=a&filters[field][$in][1]=b` | In array |
| `$contains` | `filters[field][$contains]=text` | Contains (case-sensitive) |
| `$containsi` | `filters[field][$containsi]=text` | Contains (case-insensitive) |
| `$gt` / `$lt` | `filters[field][$gt]=10` | Greater / less than |

## Sort and Pagination

```
GET /api/articles?sort[0]=title:asc&pagination[page]=1&pagination[pageSize]=10
```