# Blotato API Workflows (n8n)

This document catalogs the n8n workflows in this repo that interact with the Blotato API, the endpoints they use, required headers, and environment configuration.

- Service base URL (local): `http://localhost:5679` (n8n-enhanced)
- API base (Blotato): `https://backend.blotato.com/v2`
- Auth header: `blotato-api-key`

## Workflows that call Blotato

- `workflows/recovered-workflows/drive_blotato_instagram_n8n.json`
  - Nodes:
    - `Upload to Blotato` → `POST /media`
      - Body: `{ "url": "{{ $json.ucLink }}" }`
      - Headers: `{ "blotato-api-key": "{{ $env.BLOTATO_API_KEY }}" }`
    - `Post to Instagram` → `POST /posts`
      - Body:
        ```json
        {
          "post": {
            "accountId": "{{ $env.INSTAGRAM_ACCOUNT_ID }}",
            "content": {
              "text": "{{ $json.caption }}",
              "mediaUrls": ["{{ $json.url }}"],
              "platform": "instagram"
            },
            "target": { "targetType": "instagram" }
          }
        }
        ```
      - Headers: `{ "blotato-api-key": "{{ $env.BLOTATO_API_KEY }}" }`
  - Flow highlights:
    - Reads Google Sheet (`$env.SHEETS_ID`) → Prepares Drive file share link → Uploads to Blotato → Waits for CDN → Posts to Instagram → Updates sheet status.

- `workflows/recovered-workflows/My_workflow (4).json`
- `workflows/recovered-workflows/My_workflow (5).json`
  - Multiple platform-specific publish nodes using Blotato:
    - `POST /media` (upload)
    - `POST /posts` (publish) for LinkedIn, TikTok, Pinterest, YouTube, Threads, Twitter, Bluesky, Instagram, etc.
  - Header source examples:
    - `={{ $json.blotato_api_key }}`
    - `={{ $('Setup Social Accounts').item.json.blotato_api_key }}`

- `workflows/recovered-workflows/ai-social-media-system-v1.json`
  - Several HTTP mappers performing `POST /posts` with header:
    - `{"name": "blotato-api-key", "value": "{{14.blotato_api_key}}"}`

## Required environment/credentials

- `BLOTATO_API_KEY` (Blotato API key)
- Platform account IDs as needed, e.g. `INSTAGRAM_ACCOUNT_ID`
- Google APIs (if using Drive/Sheets):
  - `SHEETS_ID` (spreadsheet with content rows)
  - OAuth2 credentials for Google Sheets and Drive (configured in n8n Credentials)

Tips:
- Prefer storing secrets in n8n Credentials when possible. If using env vars in nodes, ensure the same variables are defined in the n8n container/service.

## Blotato endpoints used

- `POST /media` — Upload media by public URL
  - Body: `{ "url": "<public URL>" }`
- `POST /posts` — Publish a post to a platform
  - Body (example for Instagram): see above
- Required header for both: `blotato-api-key: <key>`

## How to import/run workflows

- Import via n8n UI: `+` → Import from file → select the JSON.
- Or via script (`music_system/import_workflow.py`): posts to `http://localhost:5679/api/v1/workflows`.
- After importing, open the workflow at `http://localhost:5679/workflow/<workflowId>`.

## Troubleshooting

- 401/403 from Blotato → verify `BLOTATO_API_KEY` is set/valid.
- 404 from Blotato `/media` or `/posts` → check body shape and that media URLs are public/accessible.
- n8n 404 on deep-link → verify the workflow exists in the instance and use the correct UI route `.../workflow/<id>` (no extra segments).
- If Google uploads fail → ensure Drive file is shared (`Make Public` step) before posting.
