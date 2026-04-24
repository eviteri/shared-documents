---
name: strapi-v4
description: Strapi v4 REST API patterns using nested data.attributes format. Use this when working with Strapi v4 projects or APIs.
---

# Strapi v4 Skill

## Response Format

Strapi v4 uses a **nested response format** with `data.attributes` wrapping all fields.

### Single Entry Response

```json
{
  "data": {
    "id": 1,
    "attributes": {
      "title": "My Article",
      "slug": "my-article",
      "createdAt": "2025-01-01T00:00:00.000Z",
      "updatedAt": "2025-01-01T00:00:00.000Z",
      "publishedAt": "2025-01-01T00:00:00.000Z"
    }
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
      "attributes": {
        "title": "My Article",
        "slug": "my-article"
      }
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

- **Use numeric `id`** for all CRUD operations
- Fields are **nested inside `data.attributes`** — never access them directly on `data`
- Relations are also nested: `data.attributes.category.data.attributes.name`

## REST API Patterns

### Fetch by ID

```
GET /api/articles/{id}
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
PUT /api/articles/{id}
Content-Type: application/json

{
  "data": {
    "title": "Updated Title"
  }
}
```

### Delete

```
DELETE /api/articles/{id}
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

## Accessing Nested Data

When working with v4 responses, always unwrap `attributes`:

```js
// Single entry
const title = response.data.attributes.title;

// Collection
const titles = response.data.map(item => item.attributes.title);

// Nested relation
const categoryName = response.data.attributes.category.data.attributes.name;
```