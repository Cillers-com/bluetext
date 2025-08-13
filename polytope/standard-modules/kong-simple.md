# polytope/kong!simple

Runs a Kong Gateway container with pre-specified plugins and config file.

## Module Specification

```yml
info: Runs a Kong Gateway container with pre-specified plugins and config file.
id: simple
params:
- id: image
  info: The container image to run.
  name: Image
  type: [default, str, 'public.ecr.aws/docker/library/kong:3.6.1']
- id: port
  info: The port to serve Kong on.
  name: Port
  type: [default, int, 3000]
- id: plugins
  info: List of Kong plugins to install.
  name: Plugins
  type:
  - maybe
  - - {name: str, version: str, package: str}
- {id: config-file, info: The declarative YAML config file to use., name: Config file, type: mount-source}
- id: autoreload
  info: If true, watches for changes to the config file and reloads Kong when it changes.
  name: Autoreload
  type: [default, bool, true]
- id: log-level
  info: Kong's log level.
  name: Log level
  type: [default, str, notice]
- id: env
  info: Environment variables to pass to Kong.
  name: Environment variables
  type:
  - maybe
  - [env-var]
- id: restart
  info: What policy to apply on restarting containers that fail.
  name: Restart policy
  type:
  - maybe
  - policy: [enum, always, on-failure]
    max-restarts: [maybe, int]
- id: services
  info: The services to expose.
  name: Services
  type:
  - default
  - [service-spec]
  - - id: kong
      ports:
      - {port: 3000, protocol: http}
module: kong
args:
  cmd: /opt/kong-scripts/watch
  env: |-
    #pt-clj (concat
     [{:name "KONG_DATABASE", :value "off"}
     {:name "CONF_IN", :value "/tmp/kong-in.yml"}
     {:name "CONF_OUT", :value "/tmp/kong.yml"}
     {:name  "KONG_LOG_LEVEL"
      :value (:log-level params)}
     {:name  "AUTORELOAD"
      :value (str (:autoreload params))}
     {:name  "INSTALL_PLUGINS"
      :value (->>
              (:plugins params)
              (map
              (fn
              [{p :package, v :version}]
              (str p " " v)))
              (clojure.string/join \,))}
     {:name  "KONG_PLUGINS"
      :value (->>
              (into
              ["bundled"]
              (map :name (:plugins params)))
              (clojure.string/join \,))}
     {:name  "KONG_PROXY_LISTEN"
      :value (str "0.0.0.0:" (:port params))}]
     (:env params))
  image: '#pt-clj (:image params)'
  mounts:
  - source: {type: repo, repo: '#pt-clj pt/module-project-ref', path: /scripts}
    path: /opt/kong-scripts
  - {source: '#pt-clj (:config-file params)', path: /tmp/kong-in.yml}
  port: '#pt-clj (:port params)'
  restart: '#pt-clj (:restart params)'
  services: '#pt-clj (:services params)'
  user: |-
    #pt-clj (when (not-empty (:plugins params))
      0)
```

## Description

This module provides a Kong API Gateway with declarative configuration support, automatic plugin installation, and configuration file watching. Kong runs in database-free mode using declarative configuration files.

## Parameters

- **image**: The Kong Docker image to use (default: `public.ecr.aws/docker/library/kong:3.6.1`)
- **port**: The port Kong will listen on (default: `3000`)
- **plugins**: List of Kong plugins to install with their versions and packages
- **config-file**: Required declarative YAML configuration file for Kong
- **autoreload**: Whether to watch config file changes and reload Kong (default: `true`)
- **log-level**: Kong's logging level (default: `notice`)
- **env**: Additional environment variables for Kong
- **restart**: Container restart policy configuration
- **services**: Service exposure configuration (default: Kong service on specified port)

## Configuration File

Kong requires a declarative configuration file in YAML format. This file defines services, routes, plugins, and other Kong entities.

### Basic Configuration Structure

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
```

## Plugin Installation

The module supports automatic installation of Kong plugins. Specify plugins with their name, version, and package:

```yaml
plugins:
  - name: rate-limiting
    version: "2.1.0"
    package: kong-plugin-rate-limiting
```

## Features

- **Database-Free Mode**: Runs Kong without a database using declarative configuration
- **Automatic Plugin Installation**: Installs specified plugins during container startup
- **Configuration Watching**: Automatically reloads Kong when configuration files change
- **Flexible Service Exposure**: Configurable port exposure for proxy and admin APIs
- **Environment Configuration**: Full environment variable support for Kong settings

## Default Services

By default, the module exposes Kong on the specified port with HTTP protocol. You can override this with the `services` parameter to expose additional ports (like the admin API).

## Log Levels

Available log levels: `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, `emerg`

## Environment Variables

The module automatically sets essential Kong environment variables:
- `KONG_DATABASE=off` (database-free mode)
- `KONG_PROXY_LISTEN` (based on port parameter)
- `KONG_LOG_LEVEL` (based on log-level parameter)
- Plugin-related variables for installation and activation

Additional environment variables can be provided via the `env` parameter.
