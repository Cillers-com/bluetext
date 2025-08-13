# Documentation For The Phantom Token Handler Access Control Blueprint

This Blueprint specifies how to add and update modules for implementing phantom token handler access control pattern. This pattern provides secure API access by exchanging opaque tokens for JWT tokens at the API gateway level.

Currently, this blueprint includes instructions for updating the Kong API Gateway module. Instructions for updating Curity, web-app, and API modules will be added in future versions.

## Prerequisites

The following blueprint-based modules must be installed in the `polytope.yml` file: `kong-api-gateway`, `curity`, `postgres`, `api`, `web-app`. 

All network access to the `api` module must be configured to go through the `kong` module.

If one of these prerequisites is missing, you must ask the user wheather to proceed or not. 

## Postgres Module Updates

Add the following attribute to the `postgres` module in `polytope.yml`

```yaml
      scripts:
        - { type: host, path: ./modules/curity/db.sql }
```

## Kong Module Updates

When implementing phantom token handler access control, the Kong module requires specific environment variables and configuration updates.


Add the following attributes to your Kong module:

```yaml
env:
  - { name: KONG_NGINX_HTTP_LUA_SHARED_DICT, value: 'phantom-token 10m' }
  - { name: KONG_PLUGINS, value: 'bundled,oauth-proxy,phantom-token' }
plugins:
  - { name: phantom-token, package: kong-phantom-token, version: 2.0.0 }
```

**Important:** If the `KONG_PLUGINS` environment variable already exists with a value, append the new plugins to the existing value as a comma-separated list. For example, if the current value is `bundled,rate-limiting`, it should become `bundled,rate-limiting,oauth-proxy,phantom-token`.

### Example of Updated Polytope Kong Module

```yaml
  - id: kong
    info: "Runs the Kong API Gateway with phantom token handler (blueprint: phantom-token-handler-access-control)"
    module: polytope/kong!simple
    args:
      image: gcr.io/arched-inkwell-420116/kong-oauth-proxy:latest
      config-file: { type: host, path: ./modules/kong/kong.yml }
      autoreload: true
      env:
        - { name: KONG_ADMIN_LISTEN, value: "0.0.0.0:8001" }
        - { name: KONG_PROXY_LISTEN, value: "0.0.0.0:8000" }
        - { name: KONG_NGINX_HTTP_LUA_SHARED_DICT, value: 'phantom-token 10m' }
        - { name: KONG_PLUGINS, value: 'bundled,oauth-proxy,phantom-token' }
      plugins:
        - { name: phantom-token, package: kong-phantom-token, version: 2.0.0 }
      services:
      - { id: kong, ports: [{ port: 8000, protocol: http, expose-as: pt.value kong-port }] }
      - { id: kong-admin, ports: [{ port: 8001, protocol: http, expose-as: pt.value kong-admin-port }] }
```

## Kong Configuration Updates

The Kong configuration file (typically `modules/kong/kong.yml`) must be updated to include the token handler service and updated API service plugins.

### Token Handler Service

Add the following service to your Kong configuration:

```yaml
- name: token-handler
  url: http://curity:8443/apps/token-handler
  routes:
  - name: token-handler-route
    paths:
    - /apps
  plugins:
  - name: cors
    config:
      origins:
      - http://localhost:5000
      methods:
      - GET
      - POST
      - OPTIONS
      headers:
      - Accept
      - Authorization
      - Content-Type
      - x-curity-csrf
      exposed_headers:
      - Authorization
      credentials: true
      preflight_continue: false
```

### Updated API Service Plugins

Update your existing API service to include the following plugins:

```yaml
- name: api
  url: http://api:4000
  routes:
  - name: api-route
    strip_path: false
    paths:
    - /api
  plugins:
    - name: oauth-proxy
      config:
        cookie_prefix: th-
        cookie_key: -----BEGIN ENCRYPTED PRIVATE KEY-----\nMIH0MF8GCSqGSIb3DQEFDTBSMDEGCSqGSIb3DQEFDDAkBBC9QYX294HIS9+2QD+I\n417zAgIIADAMBggqhkiG9w0CCQUAMB0GCWCGSAFlAwQBKgQQUxjBCsV/uznFjiOA\ny0cbTwSBkLznX6LXEjyWSpWpvgc13SK9fiELuMFgCcIjuDJeIc62K+PixqODqgfY\ngsnYLuYKWBx+UY+DNn9xaganJ42c1VfY+HtZFdWagE+jB6xJ0pnp2IQAK9yixv9e\nT0EEIJQ+kRoMM16B0PimJPHd1bEyivUpNfkP7fMHEpmkbbdaO8fwke4oUkk3uciF\nVBdAwnupLA==\n-----END ENCRYPTED PRIVATE KEY-----
        cookie_key_pass: password

    - name: phantom-token
      config:
        introspection_endpoint: http://curity:8443/oauth/v2/oauth-introspect
        client_id: kong-introspection 
        client_secret: Password1
        token_cache_seconds: 900
        verify_ssl: false

    - name: cors
      config:
        origins:
        - http://localhost:5000
        methods:
        - GET
        - POST
        - OPTIONS
        headers:
        - Accept
        - Authorization
        - Content-Type
        - x-curity-csrf
        exposed_headers:
        - Authorization
        credentials: true
        preflight_continue: false
```

### Complete Configuration Example

Here's a complete example of a Kong configuration file with phantom token handler access control:

```yaml
_format_version: '2.1'
_transform: true

services:
- name: token-handler
  url: http://curity:8443/apps/token-handler
  routes:
  - name: token-handler-route
    paths:
    - /apps
  plugins:
  - name: cors
    config:
      origins:
      - http://localhost:5000
      methods:
      - GET
      - POST
      - OPTIONS
      headers:
      - Accept
      - Authorization
      - Content-Type
      - x-curity-csrf
      exposed_headers:
      - Authorization
      credentials: true
      preflight_continue: false

- name: api
  url: http://api:4000
  routes:
  - name: api-route
    strip_path: false
    paths:
    - /api
  plugins:
    - name: oauth-proxy
      config:
        cookie_prefix: th-
        cookie_key: -----BEGIN ENCRYPTED PRIVATE KEY-----\nMIH0MF8GCSqGSIb3DQEFDTBSMDEGCSqGSIb3DQEFDDAkBBC9QYX294HIS9+2QD+I\n417zAgIIADAMBggqhkiG9w0CCQUAMB0GCWCGSAFlAwQBKgQQUxjBCsV/uznFjiOA\ny0cbTwSBkLznX6LXEjyWSpWpvgc13SK9fiELuMFgCcIjuDJeIc62K+PixqODqgfY\ngsnYLuYKWBx+UY+DNn9xaganJ42c1VfY+HtZFdWagE+jB6xJ0pnp2IQAK9yixv9e\nT0EEIJQ+kRoMM16B0PimJPHd1bEyivUpNfkP7fMHEpmkbbdaO8fwke4oUkk3uciF\nVBdAwnupLA==\n-----END ENCRYPTED PRIVATE KEY-----
        cookie_key_pass: password

    - name: phantom-token
      config:
        introspection_endpoint: http://curity:8443/oauth/v2/oauth-introspect
        client_id: kong-introspection 
        client_secret: Password1
        token_cache_seconds: 900
        verify_ssl: false

    - name: cors
      config:
        origins:
        - http://localhost:5000
        methods:
        - GET
        - POST
        - OPTIONS
        headers:
        - Accept
        - Authorization
        - Content-Type
        - x-curity-csrf
        exposed_headers:
        - Authorization
        credentials: true
        preflight_continue: false
```

## Security Notes

**Private Key Certificate:** The configuration currently includes a private key certificate directly in the Kong configuration file. This is acceptable for initial setup, but the private key should be moved to a Polytope secret and interpolated into the configuration file in production environments.

## Values And Secrets

The following values should be defined: `kong-host`, `kong-host-pt`, `kong-port` and `kong-admin-port`. Do not hardcode these values in `polytope.yml`.

