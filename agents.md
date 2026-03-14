# Profistranka AI Agent Guide

This document describes how an AI agent can upload a complete website to Profistranka CMS.

## Overview

Profistranka is a multi‑tenant CMS that hosts AI‑generated websites. Each website gets:
- A unique subdomain: `{internalName}.profistranka.cz`
- Automatic HTTPS (Let’s Encrypt)
- An admin dashboard to edit content and settings
- API access for programmatic updates

## Step‑by‑step workflow for AI agents

### 1. Create a new website

**Endpoint:** `POST https://admin.profistranka.cz/api/v1/websites`

**Headers:**
- `Content-Type: application/json`

**Body:**
```json
{
  "name": "Website Name",
  "internalName": "unique-slug"
}
```

**Response:**
```json
{
  "id": "website-uuid",
  "name": "Website Name",
  "internalName": "unique-slug",
  "apiKey": {
    "apiKey": "hex-api-key",
    "secret": "hex-secret"
  },
  "domain": {
    "domainName": "unique-slug.profistranka.cz",
    "sslEnabled": true
  }
}
```

**Important:** The `secret` is shown **only once** on creation. Store it securely.

### 2. Upload website files

**Endpoint:** `POST https://admin.profistranka.cz/api/v1/ai/upload`

**Headers:**
- `Content-Type: multipart/form-data`
- `X-API-Key: your-api-key`
- `X-API-Secret: your-secret`

**Form fields:**
- `websiteId` (string) – the website UUID from step 1
- `files` (file[]) – one or more HTML/CSS/JS/image files

**Notes:**
- The endpoint accepts multiple files in a single request.
- Each file will be stored under its original filename (relative paths are preserved if you send `path/to/file.html`).
- Maximum file size: 10 MB per file.
- Supported MIME types: `text/html`, `text/css`, `application/javascript`, `image/*`.

**Response:**
```json
{
  "websiteId": "website-uuid",
  "uploaded": [
    {
      "filePath": "index.html",
      "mimeType": "text/html",
      "size": 1234
    }
  ]
}
```

### 3. Verify the website is live

**URL:** `https://{internalName}.profistranka.cz`

The website will be available immediately after upload. SSL certificate is issued automatically (may take up to 60 seconds).

### 4. Update website settings (optional)

**Endpoint:** `PATCH https://admin.profistranka.cz/api/v1/websites/{websiteId}/settings`

**Headers:**
- `Content-Type: application/json`
- `X-API-Key`, `X-API-Secret`

**Body:**
```json
{
  "metaTitle": "Page Title",
  "metaDescription": "Page description for SEO",
  "favicon": "data:image/svg+xml;base64,...",
  "analyticsCode": "<!-- Google Analytics -->"
}
```

### 5. Edit files later

**Get file list:** `GET https://admin.profistranka.cz/api/v1/files?websiteId={websiteId}`

**Get file content:** `GET https://admin.profistranka.cz/api/v1/files/websites/{websiteId}/files/{filePath}`

**Update file:** `PUT https://admin.profistranka.cz/api/v1/files/websites/{websiteId}/files/{filePath}`

**Headers for update:**
- `Content-Type: application/json`
- `X-API-Key`, `X-API-Secret`

**Body:**
```json
{
  "content": "data:text/html;base64,PGh0bWw+...",
  "mimeType": "text/html"
}
```

## Example: Full website upload (Python‑like pseudocode)

```python
import requests

# 1. Create website
create_resp = requests.post(
    "https://admin.profistranka.cz/api/v1/websites",
    json={"name": "My AI Site", "internalName": "my-ai-site"}
)
website = create_resp.json()
website_id = website["id"]
api_key = website["apiKey"]["apiKey"]
secret = website["apiKey"]["secret"]

# 2. Prepare files
files = [
    ("files", ("index.html", open("index.html", "rb"), "text/html")),
    ("files", ("style.css", open("style.css", "rb"), "text/css")),
    ("files", ("img/logo.png", open("logo.png", "rb"), "image/png"))
]

# 3. Upload
upload_resp = requests.post(
    "https://admin.profistranka.cz/api/v1/ai/upload",
    headers={"X-API-Key": api_key, "X-API-Secret": secret},
    data={"websiteId": website_id},
    files=files
)

# 4. Verify
live_url = f"https://{website['domain']['domainName']}"
print(f"Website live at: {live_url}")
```

## Error handling

| HTTP Code | Meaning | Action |
|-----------|---------|--------|
| 400 | Bad request (missing fields, invalid data) | Check request body |
| 401 | Invalid API key/secret | Verify credentials |
| 404 | Website or file not found | Check IDs |
| 409 | Duplicate internalName | Choose a different slug |
| 429 | Rate limit exceeded | Wait before retrying |
| 500 | Server error | Retry later |

## Best practices for AI agents

1. **Generate unique internalName** – use a combination of topic and hash (e.g., `pet‑clinic‑a3f8e`).
2. **Bundle all assets** – upload HTML, CSS, JS, images in one batch.
3. **Use relative paths** – inside HTML, reference `style.css`, not `/style.css`.
4. **Set meaningful meta tags** – improve SEO and social sharing.
5. **Store credentials securely** – the secret cannot be retrieved again.
6. **Handle SSL delay** – certificate issuance may take a minute; you can poll `https://subdomain.profistranka.cz` until it returns 200.

## Advanced: Custom domains

To attach a custom domain (e.g., `www.client‑site.cz`):

1. **Add domain:** `POST https://admin.profistranka.cz/api/v1/domains`
   ```json
   {"websiteId": "...", "domainName": "www.client‑site.cz"}
   ```
2. **Set DNS A record** for `www.client‑site.cz` to `185.199.108.153` (Profistranka server).
3. **SSL will be issued automatically** within a few minutes.

## Support

- **Documentation:** https://github.com/Matkalcz/profistranka
- **Issues:** GitHub Issues in the repository
- **API status:** `https://admin.profistranka.cz/health`

---

*Last updated: 2026‑03‑14*  
*Profistranka version: 1.0*