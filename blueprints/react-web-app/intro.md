## Documentation For The React Web App Blueprint

This Blueprint specifies how to add and update a Polytope module for a React Web App.

Parent module: `polytope/node`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes.

## Example of a Polytope React Web App module

```yaml
  - id: web-app
    params:
      - id: cmd
        type: [default, str, ./bin/run] # default to the run script
    module: polytope/node
    args:
      id: web-app
      image: oven/bun:slim # specify an image with bun
      code: { type: host, path: ./modules/web-app } # mount the `modules/web-app` directory (assuming the code is there)
      cmd: pt.param cmd
      env:
        - { name: PORT, value: pt.value web-app-port }
        - { name: API_HOST, value: }
      restart:
        policy: always # always restart on failure
      services:
        - id: web-app
          ports: [{protocol: http, port: pt.value web-app-port, expose-as: pt.value web-app-port}]
      mounts:
        # Use volumes to cache dependencies:
        - { path: /root/.cache/, source: { type: volume, id: dependency-cache }}
        - { path: /root/.bun/, source: { type: volume, id: bun-cache }}
        - { path: /app/node_modules/, source: { type: volume, id: node-modules }}
```

## Values And Secrets

The following values should be defined: `web-app-port`. Do not hardcode these values in `polytope.yml`.

## Instructions For How To Add npm Packages

Fetch the documentation for the add-package-npm module from the bluetext MCP server. Run the add-package-npm module in accordance with this documenation.

## Accessing An API Server

Do not route API requests through the web app node server. Send the requests directly to the API server. 

## Adding The Module

Here are the steps you should take to add this module.

### 1. Start by running the React Web App boilerplate
Fetch the documentation for the boilerplate module from the bluetext MCP server. 

Run the following command to generate the initial code for the React Web App module: 

`pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-react-web-app, target-path: my-module-code-root-path}"`

Where `my-module-code-root-path` should be replaced by the relative path to where you should place the module code root directory, `modules/web-app` by default. 

### 2. Make any necessary modifications
Check the generated boilerplate code and see if any changes need to be made to fulfill the user's request. 

### 3. Add needed packages using the instructions provided above.

## API Access

If there is an API module that is part of the same template as this Web App module, you should by default access that API directly without proxying through the Web App backend. 