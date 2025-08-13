# Documentation For The Kong API Gateway Blueprint

This Blueprint specifies how to add and update a Kong API Gateway module for providing an API gateway service with routing, authentication, and other middleware capabilities.

Parent module: `polytope/kong!simple`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes.

## Example of a Polytope Kong API Gateway Module

```yaml
  - id: kong
    info: "Runs the Kong API Gateway (blueprint: kong-api-gateway)"
    module: polytope/kong!simple
    args:
      image: gcr.io/arched-inkwell-420116/kong-oauth-proxy:latest
      config-file: { type: host, path: ./modules/kong/kong.yml }
      autoreload: true
      env:
        - { name: KONG_ADMIN_LISTEN, value: "0.0.0.0:8001" }
        - { name: KONG_PROXY_LISTEN, value: "0.0.0.0:8000" }
      services:
      - { id: kong, ports: [{ port: 8000, protocol: http, expose-as: pt.value kong-port }] }
      - { id: kong-admin, ports: [{ port: 8001, protocol: http, expose-as: pt.value kong-admin-port }] }
```

## Kong Configuration

Kong uses declarative configuration files in YAML format. The configuration defines services, routes, and plugins. Create a file `modules/kong/kong.yml` with the appropriate configruation.

### Example Configuration

```yaml
_format_version: '2.1'
_transform: true

services:
- name: api
  url: http://api:4000
  routes:
  - name: api-route
    strip_path: false
    paths:
    - /api
  plugins:
    - name: cors
      config:
        origins:
        - http://localhost:3000
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

## Values And Secrets

The following values should be defined: `kong-host`, `kong-host-pt`, `kong-port` and `kong-admin-port`. Do not hardcode these values in `polytope.yml`.


