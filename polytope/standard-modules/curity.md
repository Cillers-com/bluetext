# polytope/curity

Runs a Curity ID server container.


## Generate The Module Files Using The Curity Identity Server Boilerplate
Fetch the documentation for the boilerplate module from the bluetext MCP server.

Run the following command to generate the initial code for the React Web App module: 

`pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-react-web-app, target-path: my-module-code-root-path}"`

Where `my-module-code-root-path` should be replaced by the relative path to where you should place the module code root directory, `modules/web-app` by default. 


## Example Of A Curity Polytope Module

```yaml
  - id: curity
    module: polytope/curity
    args:
      image: curity.azurecr.io/curity/idsvr:10.4.0
      # log-level: DEBUG  # NOTE: uncomment when developing against curity
      config-file: { type: host, path: ./modules/curity/config.xml }
      restart: { policy: always }
      password: "password"
      env:
        - { name: ADMIN, value: true }
        - { name: CURITY_BASE_URL, value: pt.value curity-base-url }
        - { name: WEB_APP_URL, value: pt.value web-app-url }
        - { name: AGENT_BASE_URL, value: pt.value curity-agent-base-url }
      mounts:
        - { path: /opt/idsvr/usr/bin/post-commit-cli-scripts, source: { type: host, path: ./modules/curity/post-commit-cli-scripts }}
        - { path: /opt/idsvr/etc/init/curity-users.xml, source: { type: host, path: ./modules/curity/users.xml }}

```

## Parent Module

This module extends `polytope/curity` and provides Curity-specific configuration and defaults.

