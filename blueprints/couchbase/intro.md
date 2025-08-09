# Documentation For The Couchbase Blueprint

This Blueprint specifies how to add and update a Couchbase module for providing a Couchbase cluster as well as how to access the Couchbase cluster.

Parent module: `polytope/couchbase`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes.

## Example of a Polytope Couchbase Module

```yaml
  - id: couchbase
    info: "Runs the Couchbase cluster (blueprint: couchbase)"
    module: polytope/redpanda
    args:
      image: couchbase:enterprise-7.6.6
      data-volume: { id: couchbase-data, type: volume, scope: project }
```

**IMPORTANT: Always create a wrapper module for Redpanda with a persistent volume!** Without a volume, all data will be lost when the container restarts.

## Accessing Couchbase

### Initialization
It can take up to 10 minutes for Couchbase to be initialized on the first run depending on the machine it is running on. Code that depends on Couchbase being up and running therefore needs to have long connection retry periods.

### How To Test that the connection is working
Python: bucket.ping()

## Cache the connection
Cache the connection to Couchbase so you don't need to reestablish it for each action. 

No need to cache the scope or collection. 

## How to store data in Couchbase
Document id: the document id is stored as the key to the document. No need to include it in the document itself.

Document type: this is determined by which collection the document is stored in. No need to include it in the document itself.

Documents are stored in collections. Ensure that you are storing each document in the correct collection. 

## Dependencies

### Values And Secrets

The following values should be defined: `couchbase-host` and `couchbase-tls`. The following secrets should be defined: `couchbase-username` and `couchbase-password`.  Do not hardcode these values in `polytope.yml`.

### Init Module
Read the blueprint documentation resource for the `init` module. The `init` module must run as part of the same stack as the `couchbase` module to ensure that the Couchbase cluster is initialized. Also, use the `init` module for adding and updating buckets, scopes and collections.

If you try to add or configure the `init` module without first reading its blueprint, you will fail.

The `COUCHBASE_HOST`, `COUCHBASE_TLS`, `COUCHBASE_USERNAME`, `COUCHBASE_PASSWORD` values/secrets need to be added to the `init` modules `env` `args`. E.g.
```yaml
        - { name: COUCHBASE_HOST, value: pt.value couchbase-host }
        - { name: COUCHBASE_TLS, value: pt.value couchbase-tls }
        - { name: COUCHBASE_USERNAME, value: pt.secret couchbase-username }
        - { name: COUCHBASE_PASSWORD, value: pt.secret couchbase-password }
```

The `init` id for Couchbase is 'couchbase'. Make sure it is included in the `INIT_SERVICES` `env` `args`.

#### Configuration File

Buckets, Scopes and Collections should be configured with environment-specific settings in the `./conf/init/couchbase.yaml` file.

Configuration Specification:
- `bucket_defaults`: Global settings applied to all buckets
- `collection_defaults`: Global settings applied to all collections
- `buckets`: Define specific buckets with their scopes and collections
- `defaults`: Per-bucket/collection defaults across all environments
- `env_settings`: Environment-specific overrides (dev, test, staging, prod)
- Empty objects `{}` inherit from global defaults


E.g. 

```yaml
bucket_defaults:  # For all buckets
  ram_quota_mb: 100
  bucket_type: couchbase
  num_replicas: 0
  flush_enabled: false
  conflict_resolution_type: sequence_number
  eviction_policy: value_only
  compression_mode: off

collection_defaults:  # For all collections
  max_ttl: 0  # seconds

buckets:
  main:
    defaults:  # Per-bucket defaults across envs
      flush_enabled: true
      compression_mode: passive
    env_settings:
      dev:
        ram_quota_mb: 100
        num_replicas: 0
      test:
        ram_quota_mb: 150
        num_replicas: 1
      staging:
        ram_quota_mb: 500
        num_replicas: 2
        flush_enabled: false
        eviction_policy: full_eviction
        compression_mode: active
      prod:
        ram_quota_mb: 2048
        num_replicas: 3
        flush_enabled: false
        eviction_policy: full_eviction
        compression_mode: active
    scopes:
      _default:
        collections:
          users:
            defaults:  # Per-collection defaults across envs
              max_ttl: 3600
            env_settings:
              staging:
                max_ttl: 7200
              prod:
                max_ttl: 7200
          sessions: {}  # No settings; uses collection_defaults

```














