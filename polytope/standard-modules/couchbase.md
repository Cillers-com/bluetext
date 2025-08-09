# polytope/couchbase

Runs a Couchbase Server container with sensible defaults and full configurability.

## Module Specification

```yml
info: Runs a Couchbase container.
id: couchbase
default?: true
params:
- id: image
  info: The container image to use.
  name: Image
  type: [default, str, 'public.ecr.aws/docker/library/couchbase:community-7.2.4']
- id: id
  info: The ID of the container to spawn.
  name: ID
  type: [default, str, couchbase]
- id: cmd
  info: The command to run in the container. If unspecified, runs the Couchbase server.
  name: Command
  type:
  - maybe
  - - either
    - str
    - - [maybe, str]
- id: data-volume
  name: Data Volume
  info: The volume (if any) to mount for data.
  type: [maybe, mount-source]
- id: env
  info: Environment variables for the container.
  name: Environment variables
  type:
  - maybe
  - - [maybe, env-var]
- id: services
  info: Ports to expose as services.
  name: Services
  type:
  - maybe
  - [service-spec]
- id: mounts
  info: Code or files to mount into the container.
  name: Mounts
  type:
  - maybe
  - - {source: mount-source, path: absolute-path}
- id: restart
  info: What policy to apply on restarting containers that fail.
  name: Restart policy
  type:
  - maybe
  - policy: [enum, always, on-failure]
    max-restarts: [maybe, int]
module: polytope/container
args:
  image: '#pt-clj (:image params)'
  id: '#pt-clj (:id params)'
  services: |-
    #pt-clj (or
     (:services params)
     [{:id    :couchbase
      :ports [{:internal true
               :label    :epmd
               :port     4369
               :protocol :tcp}
              {:label :http, :port 8091, :protocol :http}
              {:label :capi, :port 8092, :protocol :http}
              {:label :query, :port 8093, :protocol :http}
              {:label :fts, :port 8094, :protocol :http}
              {:label :cbas, :port 8095, :protocol :http}
              {:label :eventing, :port 8096, :protocol :http}
              {:label :backup, :port 8097, :protocol :http}
              {:internal true
               :label    :indexer
               :protocol :tcp
               :range    "9100-9105"}
              {:internal true
               :label    :analytics
               :protocol :tcp
               :range    "9110-9122"}
              {:label :prometheus, :port 9123, :protocol :tcp}
              {:internal true
               :label    :backup-grpc
               :port     9124
               :protocol :tcp}
              {:internal true
               :label    :fts-grpc
               :port     9130
               :protocol :tcp}
              {:internal true
               :label    :eventing-debug
               :port     9140
               :protocol :tcp}
              {:internal true
               :label    :indexer
               :port     9999
               :protocol :tcp}
              {:label    :memcached-ssl
               :port     11207
               :protocol :tcp}
              {:label    :memcached
               :protocol :tcp
               :range    "11209-11210"}
              {:label    :memcached-prometheus
               :port     11280
               :protocol :tcp}
              {:internal true
               :label    :cluster-management
               :port     21100
               :protocol :tcp}
              {:internal true
               :label    :cluster-management
               :port     21150
               :protocol :tcp}
              {:label :http-ssl, :port 18091, :protocol :http}
              {:label :capi-ssl, :port 18092, :protocol :http}
              {:label :query-ssl, :port 18093, :protocol :http}
              {:label :fts-ssl, :port 18094, :protocol :http}
              {:label :cbas-ssl, :port 18095, :protocol :http}
              {:label    :eventing-ssl
               :port     18096
               :protocol :http}
              {:label    :backup-ssl
               :port     18097
               :protocol :http}]}])
  cmd: |-
    #pt-clj (when-let [c (:cmd params)]
      (if (string? c)
        c
        (remove nil? c)))
  env: '#pt-clj (:env params)'
  mounts: |-
    #pt-clj (concat
     (when-let [v (:data-volume params)]
      [{:path "/opt/couchbase/var", :source v}])
     (:mounts params))
  restart: '#pt-clj (:restart params)'
```

## Description

This module provisions a Couchbase Server container suitable for development and testing. It exposes the full set of standard Couchbase service ports and supports persistent storage via a mounted data volume.

## Parameters

- image: Docker image to use (default: public.ecr.aws/docker/library/couchbase:community-7.2.4)
- id: Container ID (default: couchbase)
- cmd: Optional command to run; by default runs Couchbase Server
- data-volume: Optional volume to persist data at /opt/couchbase/var
- env: Optional environment variables for container configuration
- services: Optional service/port specification; defaults to a comprehensive Couchbase service mapping
- mounts: Additional mounts to inject files/code into the container
- restart: Optional restart policy ({policy: always|on-failure, max-restarts?: int})

## Services

By default, the module exposes the following ports:

- External HTTP/UI and APIs:
  - http: 8091 (Cluster UI/REST)
  - capi: 8092
  - query: 8093 (N1QL)
  - fts: 8094
  - cbas: 8095 (Analytics)
  - eventing: 8096
  - backup: 8097
  - http-ssl: 18091, capi-ssl: 18092, query-ssl: 18093, fts-ssl: 18094, cbas-ssl: 18095, eventing-ssl: 18096, backup-ssl: 18097
- Memcached/data:
  - memcached: 11209-11210 (TCP range)
  - memcached-ssl: 11207
  - memcached-prometheus: 11280
- Internal (cluster, indexing, analytics, etc.):
  - epmd: 4369 (internal)
  - indexer: 9100-9105 (range, internal) and 9999
  - analytics: 9110-9122 (range, internal)
  - backup-grpc: 9124 (internal)
  - fts-grpc: 9130 (internal)
  - eventing-debug: 9140 (internal)
  - cluster-management: 21100, 21150 (internal)
  - prometheus: 9123

You can override the default service list via the services parameter.

## Usage Example

```yaml
modules:
  - id: couchbase
    module: polytope/couchbase
    args:
      # Persist data across runs
      data-volume:
        type: volume
        scope: project
        id: couchbase-data
```

## Notes

- Data is stored at /opt/couchbase/var inside the container; use data-volume for persistence.
- Consult the official Couchbase container documentation for supported environment variables and initialization options.
