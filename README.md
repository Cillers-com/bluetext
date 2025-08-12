# Bluetext MCP Server

Agentic coding assistant for systems built on Polytope, optimized for use with Cline (https://cline.bot).

The Bluetext MCP server provides documentation on our framework for how to build enterprise-grade systems, code generation tools, and blueprints on how to structure specific types of functionality, services and apps. 

Bluetext systems run on Polytope (https://polytope.com) which is a container orchestration platform that runs all software components in containers, e.g. Docker on your laptop and Kubernetes in the cloud. 

.
## Prerequisite: Polytope

You need Polytope installed on your computer to work with Bluetext. Here are instructions on how to install it: https://docs.cillers.com/polytope

Polytope depends on Docker in development mode, so also make sure to have Docker installed. We recommend OrbStack (https://orbstack.dev) for an optimized experience on Mac. 

## Installation

To use with Cline, clone this repo to your local machine and add use the Cline MCP manager to add it.

Blueprint should also work well with other good MCP-powered agentic coding tools. 

## Features

### Resources
The server provides access to the following documentation resources:

- **`bluetext://intro`** - Main overview and quick start guide for Bluetext framework (`intro.md`)
- **`bluetext://polytope-docs`** - Comprehensive Polytope platform documentation (`polytope/intro.md`)
- **`bluetext://code-gen-modules/<module-id>`** - Code generation module documentation (`code-gen-modules/<module-id>.md`)
  - Available modules: `add-package-npm`, `add-package-python`, `boilerplate`
- **`bluetext://blueprints`** - Introduction to Bluetext blueprints (`blueprints/intro.md`)
- **`bluetext://blueprints/<blueprint-id>`** - Individual blueprint documentation (`blueprints/<blueprint-id>/intro.md`)
  - Available blueprints: `couchbase`, `init`, `python-api`, `redpanda`, `redpanda-console`, `web-app`
- **`bluetext://polytope/standard-modules/<module-id>`** - Built-in Polytope module documentation (`polytope/standard-modules/<module-id>.md`)
  - Available modules: `container`, `node`, `python`, `python!simple`, `postgres`, `postgres!simple`, `redpanda`, `redpanda!console`, `redpanda!connect`

All resources are served with `text/markdown` MIME type and include comprehensive documentation about Polytope concepts, modules, and blueprints.

## Development

Install dependencies:
```bash
npm install
```

Build the server:
```bash
npm run build
```

For development with auto-rebuild:
```bash
npm run watch
```

## Sample Prompts

### Getting Started
```
Get the main Bluetext documentation from the bluetext mcp server to understand the framework and available resources.
```

### React Web App
```
Get the polytope documentation and a list of all available blueprints from the bluetext mcp server.

Generate a Polytope React app that runs on port 3000 by default.
```


### Python API Server

```
Get the polytope documentation and a list of all available blueprints from the bluetext mcp server.

Generate a Polytope Python api that runs on port 4000 by default.
```

### Using Standard Modules

```
Get the polytope documentation and standard modules documentation from the bluetext mcp server.

Create a Polytope configuration that uses the polytope/python module to run a Flask API server with a PostgreSQL database using the polytope/postgres module.
```

### Custom Container Setup

```
Get the polytope/container and polytope/node standard module documentation from the bluetext mcp server.

Create a Polytope setup that runs a custom Node.js application with specific environment variables and port configuration using the standard modules.
```
