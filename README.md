# Full URL Rewrite Traefik Plugin

A Traefik plugin that allows using regex to match and rewrite the full request URL, including both the host and path. This enables dynamic request forwarding to different backends without client-side redirects.
**Note:** This plugin is a fork of [e-flux-platform/full-url-rewrite-traefik-plugin](https://github.com/e-flux-platform/full-url-rewrite-traefik-plugin) with bug fixes.

## Features

- ✅ Full URL rewriting (host + path) using regex patterns
- ✅ Preserves request body and headers during rewrite
- ✅ No client-side redirects (browser URL stays unchanged)
- ✅ Supports complex regex patterns with capture groups
- ✅ Validated configuration with helpful error messages

## Use Cases

This plugin is particularly useful when you need to:

- Dynamically forward requests to different backends based on URL patterns
- Route requests to internal services without exposing internal hostnames
- Transform URLs transparently without client-side redirects
- Implement multi-tenant routing based on subdomain or path patterns

## Example

Transform requests like:

```
https://example.com/service-42/api/callback
```

into:

```
https://service-42.internal.local/api/callback
```

without performing a client redirect. This preserves the browser URL, avoids unnecessary round-trips, and keeps your internal routing logic hidden.

---

## Configuration

## Installation

### Static Configuration

Enable the plugin in Traefik's static configuration:

```yaml
experimental:
  plugins:
    fullUrlRewrite:
      moduleName: github.com/abramad-labs/url-rewrite-traefik
      version: v1.0.0
```

---

### Dynamic Configuration

Configure the plugin as a **middleware** in your dynamic configuration.

#### Basic Example

```yaml
http:
  routers:
    service-proxy:
      rule: Host(`example.com`) && PathPrefix(`/service-`)
      entryPoints:
        - web
      middlewares:
        - rewrite-service-url
      service: backend-service

  services:
    backend-service:
      loadBalancer:
        servers:
          - url: http://127.0.0.1:8080
        passHostHeader: true

  middlewares:
    rewrite-service-url:
      plugin:
        fullUrlRewrite:
          regex: ^/service-([0-9]+)(/.*)
          replacement: //service-$1.internal.local$2
```

## How It Works

1. **Pattern Matching**: The plugin matches the `regex` pattern against either:
   - The request URL (default), or
   - A header value (if `sourceStringFromHeader` is specified)

2. **Replacement**: When a match is found, it applies the `replacement` string, which can use capture groups from the regex (e.g., `$1`, `$2`)

3. **URL Rewriting**: The matched portion is replaced, and the request URL is updated accordingly

4. **Transparent Forwarding**: No client-side redirect occurs — the browser URL stays unchanged while the request is forwarded to the new destination

## Important Notes

### URL Format

Both `regex` and `replacement` typically start with `//` because request URLs in Traefik's reverse proxy context are non-absolute and do not include the scheme (http/https).

### Host Resolution

Ensure that Traefik can resolve rewritten hostnames (via Docker network, internal DNS, etc.) so the requests reach the correct backend.

### Host Header

By default, the rewritten host is passed downstream in the `Host` HTTP header. This behavior can be disabled with the [`passHostHeader`](https://doc.traefik.io/traefik/routing/services/#pass-host-header) setting if needed.