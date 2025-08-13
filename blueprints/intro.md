# Documentation for blueprints

When adding a module or template to polytope.yml, always check if there is a blueprint for that type of module or template. 

## Here are tables with a description of the blueprints that exist. 

| Module/Template Type | Blueprint ID | Module/Template | Boilerplate Repository |
|----------------------|--------------|-----------------|------------------------|
| Python API | python-api | Module | https://github.com/bluetext-io/boilerplate-python-api |
| React app | react-web-app | Module | https://github.com/bluetext-io/boilerplate-react-web-app |
| Kong API Gateway | kong-api-gateway | Module | N/A |
| Phantom Token Handler Access Control | phantom-token-handler-access-control | Module | N/A |
| Curity Identity Server | curity-identity-provider | Module | N/A |
| PostgreSQL | postgres | Module | N/A |
| Redpanda | redpanda | Module | N/A |
| Redpanda Console | redpanda-console | Module | N/A |
| Couchbase | couchbase | Module | N/A |
| Bluetext Config Manager | config-manager | Module | N/A |

The blueprints are located at `blueprints/<Blueprint ID>`

## Context
Always start by using the MCP server to fetch the appropriate blueprint resource for information about how to add the module or template. Use the `access_mcp_resource` tool with the server name `bluetext` and the URI `bluetext://blueprints/<Blueprint ID>` where `<Blueprint ID>` is the specific blueprint you need (e.g., `bluetext://blueprints/python-api` for the Python API blueprint).

Available blueprint resources:
- `bluetext://blueprints/couchbase` - Couchbase blueprint documentation
- `bluetext://blueprints/curity-identity-provider` - Curity Identity Server blueprint documentation
- `bluetext://blueprints/config-manager` - Bluetext Config Manager blueprint documentation
- `bluetext://blueprints/kong-api-gateway` - Kong API Gateway blueprint documentation
- `bluetext://blueprints/phantom-token-handler-access-control` - Phantom Token Handler Access Control blueprint documentation
- `bluetext://blueprints/postgres` - PostgreSQL blueprint documentation
- `bluetext://blueprints/python-api` - Python API blueprint documentation
- `bluetext://blueprints/redpanda` - Redpanda blueprint documentation
- `bluetext://blueprints/redpanda-console` - Redpanda Console blueprint documentation
- `bluetext://blueprints/react-web-app` - Web app blueprint documentation

## Source code boilerplate
For blueprints that have boilerplate repositories (Python API and React app), use the `boilerplate` module to generate the initial code for the module. The boilerplate code is downloaded from remote Git repositories.

**Python API:**
`pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-python-api, target-path: my-module-code-root-path}"`

**React app:**
`pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-react-web-app, target-path: my-module-code-root-path}"`

The boilerplate module will download the specified repository and copy the contents to your specified target path. You can use any publicly accessible Git repository as the source-path.
