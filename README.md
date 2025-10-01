# r2-webdav

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/abersheeran/r2-webdav)

Use Cloudflare Workers to provide a WebDav interface for Cloudflare R2.

## Features

- **WebDAV Interface**: Full WebDAV support for R2 storage
- **User Isolation**: Optional user-specific folder isolation when using user-service authentication
- **Flexible Authentication**: Supports both basic authentication and external user-service authentication

## Usage

Change wrangler.toml to your own.

```toml
[[r2_buckets]]
binding = 'bucket' # <~ valid JavaScript variable name, don't change this
bucket_name = 'webdav'
```

### Basic Authentication

For simple username/password authentication:

```bash
wrangler deploy

wrangler secret put USERNAME
wrangler secret put PASSWORD
```

### User-Service Authentication

For integration with an external user authentication service:

```bash
wrangler deploy

# Set the user-service URL
wrangler secret put USER_SERVICE_URL
```

The user-service should accept POST requests with JSON body:

```json
{
	"username": "user",
	"password": "pass"
}
```

And respond with:

```json
{
	"success": true,
	"username": "verified_username"
}
```

When using user-service authentication, each user will only have access to their own folder (prefixed with their username) in the R2 bucket, providing automatic user isolation.

## Development

With `wrangler`, you can build, test, and deploy your Worker with the following commands:

```sh
# run your Worker in an ideal development workflow (with a local server, file watcher & more)
$ npm run dev

# deploy your Worker globally to the Cloudflare network (update your wrangler.toml file for configuration)
$ npm run deploy
```

## Test

Use [litmus](https://github.com/notroj/litmus) to test.
