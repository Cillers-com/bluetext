# Documentation For The Curity Identity Server Blueprint

This Blueprint specifies how to add and update a Curity Identity Server module for providing identity and access management services with authentication, authorization, and user management capabilities.

Parent module: `polytope/curity`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes.

## Add The Module To `polytope.yml`

```yaml
  - id: curity
    info: "Runs the Curity Identity Server (blueprint: curity-identity-provider)"
    module: polytope/curity
    args:
      image: curity.azurecr.io/curity/idsvr:10.4.0
      # log-level: DEBUG  # NOTE: uncomment when developing against curity
      config-file: { type: host, path: ./modules/curity/config.xml }
      restart: { policy: always }
      password: password
      env:
        - { name: ADMIN, value: true }
        - { name: CURITY_BASE_URL, value: pt.value curity-base-url }
        - { name: WEB_APP_URL, value: pt.value web-app-url }
        - { name: AGENT_BASE_URL, value: pt.value curity-agent-base-url}
      mounts:
        - { path: /opt/idsvr/usr/bin/post-commit-cli-scripts, source: { type: host, path: ./modules/curity/post-commit-cli-scripts }}
        - { path: /opt/idsvr/etc/init/curity-users.xml, source: { type: host, path: ./modules/curity/users.xml }}
```

## Generate The Module Directory

Run the following command to generate the module directory: 

`pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-curity-identity-provider, target-path: module-root-path}"`

Where `module-root-path` should be replaced by the relative path to where you should place the module directory, `modules/curity` by default. 


## Values And Secrets

The following values should be defined: `curity-base-url`, `web-app-url`, `curity-agent-base-url`. Do not hardcode these values in `polytope.yml`. Where:
* `curity-base-url` is the URL to the Curity Identity Provider
* `web-app-url` is the URL for accessing the web app through the API Gateway. 
* `curity-agent-base-url` is the URL for accessing the Curity Identity Provider Token Handler through the API Gateway. 

```yaml
# Curity configuration
pt values set curity-base-url http://localhost:8443
pt values set web-app-url http://localhost:5000
pt values set curity-agent-base-url http://localhost:5000
```