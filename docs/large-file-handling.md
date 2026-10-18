# Handling Large File Transfers in n8n

This document explains how our n8n setup is configured to handle large file transfers (up to ~1GB) and provides guidance on workflow patterns for optimal performance.

## Configuration Overview

We've implemented the following configurations to enable large file transfers:

### 1. Environment Variables

```bash
# Binary data storage mode - uses filesystem instead of memory
N8N_DEFAULT_BINARY_DATA_MODE=filesystem

# Maximum file size for form-data uploads (4GB)
N8N_FORMDATA_FILE_SIZE_MAX=4096

# Maximum payload size for JSON bodies (1GB)
N8N_PAYLOAD_SIZE_MAX=1024
```

### 2. Resource Allocation

The Docker container is configured with:

- Memory limit: 3GB
- Memory reservation: 1GB

This provides sufficient headroom for handling files up to ~1GB in size.

## Why These Settings Matter

| Setting | Default | Our Setting | Why It Matters |
|---------|---------|-------------|----------------|
| Binary data mode | `default` (in-memory) | `filesystem` | Prevents out-of-memory errors by storing binary data on disk instead of RAM |
| Form-data file size | 200 MiB | 4096 MiB (4GB) | Allows HTTP Request nodes to send larger files |
| Payload size | 16 MiB | 1024 MiB (1GB) | Increases the maximum size for JSON payloads |
| Container memory | varies | 3GB | Provides sufficient RAM for Node.js operations during file transfers |

## Recommended Workflow Pattern

For handling large files in your n8n workflows:

1. **Read/Ingest**
   - Use the *Read Binary File* node or a Webhook with **Binary Output** enabled

2. **Prepare Headers**
   - Use a *Set* node to add appropriate headers like `Content-Type: video/mp4`

3. **HTTP Request**
   - Method: `PUT` or `POST`
   - **Send Binary Data:** `true`
   - **Binary Property:** `data` (or your chosen property name)
   - **Timeout:** Set a large value (or `0` for no timeout)

## Limitations and Best Practices

- **Node.js Limit:** Files approaching 2GB may still cause issues due to Node.js limitations
- **Worker Blocking:** Large file transfers block the n8n worker thread; consider using presigned URLs for very large files
- **Alternative Approach:** For files >1GB, consider:
  - Having n8n generate a presigned URL
  - Using an *Execute Command* node with `curl` or `aws s3 cp` to handle the actual upload
  - Using chunked/multipart uploads when supported by the destination service

## Storage Location

With `N8N_DEFAULT_BINARY_DATA_MODE=filesystem`, temporary binary files are stored in:

- Container path: `/home/node/.n8n/storage`
- Host path: `./data/storage` (via volume mapping)

## Troubleshooting

If you encounter issues with large file transfers:

1. **413 Errors:** If using a reverse proxy (Nginx/Traefik), ensure `client_max_body_size` (Nginx) or `server.maxRequestBodyBytes` (Traefik) are increased
2. **Memory Errors:** Check Docker logs for OOM (Out of Memory) errors; increase container memory if needed
3. **Timeouts:** Increase timeout settings in both n8n nodes and any proxy servers

## References

For more information, refer to:

- [n8n Documentation on Binary Data Scaling](https://docs.n8n.io/hosting/scaling/binary-data/)
- [n8n Documentation on Environment Variables](https://docs.n8n.io/hosting/configuration/environment-variables/endpoints/)
