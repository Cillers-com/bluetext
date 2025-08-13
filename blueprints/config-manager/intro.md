# Documentation for how to use Bluetext Config Manager in Polytope

# Bluetext Config Manager Module Documentation

The Bluetext Config Manager module is a system service config manager that handles initialization, data structures and other configurations of system services such as Couchbase and Redpanda. 

## Module Spec

## Example
```yaml
  - id: config-manager
    info: Manages Redpanda topics and Couchbase buckets/scopes/collections
    module: polytope/container
    args:
      image: gcr.io/arched-inkwell-420116/config-manager:latest
      id: config-manager
      restart: { policy: on-failure, max-restarts: 3 }
      env:
        - { name: ENVIRONMENT, value: pt.value environment }
        - { name: SERVICES, value: pt.value config-manager-services }
        # Service-specific environment variables (see service documentation)
      mounts:
        - { path: /config-manager/conf, source: { type: host, path: ./modules/config-manager } }
        - { path: /root/.cache/, source: { type: volume, scope: project, id: dependency-cache } }
```

## The config-manager Docker image

The config-manager image handles:
- Couchbase cluster initialization
- Bucket, scope, and collection management
- Redpanda topic creation and configuration
- Environment-specific configuration management
- Connection handling and retry logic

The published Docker image is available at:
```
gcr.io/arched-inkwell-420116/config-manager:latest
```

## Configuration

## Generate the following environment config file at "./modules/config-manager/config.yaml"
```yaml
environments:
  - dev
  - test
  - staging
  - prod

config_paths:
  couchbase: "couchbase.yaml"
  redpanda: "redpanda.yaml"
```

### Service-Specific Configuration Files
The config-manager image expects the configuration files for the SERVICES at the following paths within the container. So they need to be mounted in to the container.

- `./modules/config-manager/couchbase.yaml`: Bucket, scope, and collection definitions (see couchbase.md)
- `./modules/config-manager/redpanda.yaml`: Topic definitions and configurations (see redpanda.md)

#### Configuration Structure

The config-manager module uses a hierarchical configuration structure for resources for each resource type that is similar across all service types:

1. **Global Defaults**: Applied to all resources within this service
2. **Resource Defaults**: Per-resource defaults across all environments
3. **Environment Settings**: Environment-specific overrides

This allows for:
- Consistent base configurations
- Environment-specific scaling (dev → staging → prod)
- Minimal configuration duplication
- Easy maintenance and updates

## Environment Variables

### Required
- `ENVIRONMENT`: The target environment (must be defined in env.yaml)
- `SERVICES`: Comma-separated list of services to initialize (e.g., "couchbase", "redpanda", "couchbase,redpanda")

### Service-Specific
See individual service documentation for additional required environment variables:
- Couchbase: COUCHBASE_HOST, COUCHBASE_USERNAME, COUCHBASE_PASSWORD, COUCHBASE_TLS
- Redpanda: REDPANDA_HOST, REDPANDA_PORT

## Values And Secrets

The following values should be defined: `config-manager-services`, `environment`. Do not hardcode these values in `polytope.yml`.

The `config-manager-services` value should be specified as a comma-separated list of services to manage.

Example:
```bash
pt values set config-manager-services couchbase,redpanda
pt values set environment dev
