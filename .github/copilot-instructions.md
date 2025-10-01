# Copilot Instructions for snapgoated-r2-webdav

## Project Overview

This is a Cloudflare Workers project that provides a WebDAV interface for Cloudflare R2 storage. It allows users to access and manage their R2 buckets through WebDAV clients like file explorers, command-line tools, or any WebDAV-compatible application.

## Technology Stack

- **Runtime**: Cloudflare Workers
- **Language**: TypeScript (ES2021)
- **Storage**: Cloudflare R2 (object storage)
- **Protocol**: WebDAV (RFC 4918)
- **Build Tool**: Wrangler (Cloudflare Workers CLI)

## Code Style and Conventions

### Formatting

- Use **tabs** for indentation (not spaces)
- Tab width: 2 spaces
- Line width: 120 characters
- Use single quotes for strings
- Always use semicolons
- Run `npm run format` to auto-format code with Prettier

### TypeScript

- Target: ES2021
- Module: ES2022
- Strict mode enabled
- Use `@cloudflare/workers-types` for type definitions
- Prefer explicit types over `any`

### Code Organization

- Main entry point: `src/index.ts`
- All handler functions are async
- Use generator functions for pagination (see `listAll`)

## Architecture Patterns

### WebDAV Methods

The application implements WebDAV protocol methods:

- `OPTIONS`: Return supported methods
- `PROPFIND`: List directory contents and properties
- `PROPPATCH`: Modify resource properties
- `MKCOL`: Create collections (directories)
- `GET`: Retrieve files
- `HEAD`: Get file metadata
- `PUT`: Upload/update files
- `DELETE`: Remove files
- `COPY`: Copy files
- `MOVE`: Move/rename files

### Authentication

- Basic HTTP authentication
- Credentials stored in environment variables: `USERNAME` and `PASSWORD`
- Set via `wrangler secret put`

### R2 Storage Patterns

- Use `customMetadata.resourcetype` to distinguish directories from files
- Directory marker: `<collection />`
- Store empty Uint8Array for directory objects
- Always include `httpMetadata` and `customMetadata` when listing

### Path Handling

- Use `make_resource_path()` to extract and normalize paths from requests
- Handle trailing slashes for directory paths
- URL decode paths properly

## Development Workflow

### Local Development

```bash
npm run dev          # Start local development server
npm run format       # Format code with Prettier
npm run format:check # Check formatting without modifying files
```

### Deployment

```bash
wrangler deploy              # Deploy to Cloudflare Workers
wrangler secret put USERNAME # Set authentication username
wrangler secret put PASSWORD # Set authentication password
```

### Configuration

- Edit `wrangler.toml` for Workers configuration
- Set R2 bucket binding name and bucket name
- Compatibility date: 2023-10-16
- Node.js compatibility enabled

## Testing

Use [litmus](https://github.com/notroj/litmus) for WebDAV protocol compliance testing.

## Important Implementation Details

### Windows Explorer Compatibility

- Windows Explorer may send request bodies for MKCOL requests
- Don't strictly validate empty bodies for MKCOL (see comments in code)

### CORS Headers

- Set appropriate CORS headers for web-based WebDAV clients
- Allow all origins or match request origin
- Expose necessary WebDAV headers

### Error Responses

- Use proper HTTP status codes:
  - 401: Unauthorized
  - 404: Not Found
  - 405: Method Not Allowed
  - 409: Conflict (e.g., parent directory doesn't exist)
  - 415: Unsupported Media Type
  - 201: Created
  - 204: No Content

### XML Response Generation

- WebDAV responses use XML format
- Use string templates for XML generation
- Properly escape XML content in responses

## Key Functions to Understand

- `listAll()`: Generator function for paginating through R2 objects
- `fromR2Object()`: Convert R2Object to WebDAV properties
- `generate_propfind_response()`: Create XML responses for PROPFIND
- `make_resource_path()`: Extract resource path from request URL
- `is_authorized()`: Verify Basic authentication credentials

## When Making Changes

1. Maintain WebDAV protocol compliance
2. Test with multiple WebDAV clients (Windows Explorer, macOS Finder, etc.)
3. Consider R2 storage costs and API limits
4. Handle edge cases (empty directories, special characters in names)
5. Preserve existing authentication and CORS behavior
6. Format code with Prettier before committing
7. Update comments for any Windows Explorer or client-specific workarounds
