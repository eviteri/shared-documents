---
description: Fetches Strapi content-type schemas and saves them locally for the Strapi agent to reference.
agent: "agent"
tools: ["read", "edit", "web"]
---

Fetch content-type schemas from a running Strapi instance and save them locally for the Strapi agent to reference.

## Steps

1. **Read config** — Load `.github/strapi-config.json` to get all instances
   - If the file is missing, ask the user for their Strapi URL(s)

2. **Fetch schemas for each instance** — For every instance in the config, make a GET request to:

   ```
   {instance.baseUrl}/api/content-type-builder/content-types
   ```

   Include `Authorization: Bearer {instance.token}` if a token is configured.

   > **Note:** This endpoint requires an admin token. If the request fails with 401/403, inform the user they need to provide an admin API token for that instance in `.github/strapi-config.json`.

3. **Filter schemas** — From each response, keep only `api::` content types (exclude `admin::`, `plugin::`, and other internal types). For each content type, extract:
   - Collection name
   - Display name
   - Attributes (field name, type, required, relation info)

4. **Save to file** — Write the filtered schemas to `.github/strapi-schemas.json` in this format:

   ```json
   {
     "fetchedAt": "2025-01-01T00:00:00.000Z",
     "instances": {
       "main": {
         "source": "http://localhost:1337",
         "contentTypes": [
           {
             "uid": "api::article.article",
             "name": "Article",
             "pluralName": "articles",
             "endpoint": "/api/articles",
             "attributes": {
               "title": { "type": "string", "required": true },
               "slug": { "type": "uid", "targetField": "title" },
               "content": { "type": "richtext" },
               "category": { "type": "relation", "relation": "manyToOne", "target": "api::category.category" }
             }
           }
         ]
       },
       "cms": {
         "source": "http://localhost:1338",
         "contentTypes": [...]
       }
     }
   }
   ```

5. **Report** — Summarize what was found per instance:
   - Instance name and URL
   - Number of content types
   - List each content type with its endpoint and field count
   - Note any instances that failed (unreachable, auth error)
   - Confirm the file was saved

## Rules

- **ONLY perform GET requests**
- Only save `api::` content types — skip all internal Strapi types
- Fetch schemas from ALL configured instances, not just the default
- If one instance fails, still save schemas from the others and report the failure
- If a Strapi server is unreachable, tell the user to verify the URL and that Strapi is running
