# Documentation for boilerplate module for Generating code from Git repositories

The boilerplate module supports creating the initial code by downloading boilerplate code from any Git repository.

Check if the boilerplate module is added to the polytope.yml file. If not, add it. If you try to run this module before adding it, you will fail. 

## Polytope Module Spec

```yaml
  - id: boilerplate
    module: polytope/container
    params:
      - id: target-path
        info: Path relative to the repo root where the module's code will be created
        type: str
      - id: source-path
        info: Git repository URL to clone boilerplate code from
        type: str
    args:
      image: alpine:latest
      cmd:
        - sh
        - -c
        - |
          set -eu
          echo "Installing git..."
          apk add --no-cache git
          
          echo "Creating boilerplate code from $SOURCE_PATH at $TARGET_PATH..."
          
          # Check target path
          if [ -d "$TARGET_PATH" ]; then
            if [ "$(ls -A "$TARGET_PATH")" ]; then
              echo "ERROR: The target path '$TARGET_PATH' already exists and is not empty. Please choose a different path."
              exit 1
            fi
          else
            mkdir -p "$TARGET_PATH"
          fi
          
          # Clone repository to temporary location
          echo "Cloning boilerplate from $SOURCE_PATH..."
          TEMP_DIR="/tmp/boilerplate-$$"
          git clone --depth 1 "$SOURCE_PATH" "$TEMP_DIR"
          
          # Copy all contents first
          echo "Copying boilerplate files to $TARGET_PATH..."
          cp -r "$TEMP_DIR"/* "$TARGET_PATH/" 2>/dev/null || true
          cp -r "$TEMP_DIR"/.[!.]* "$TARGET_PATH/" 2>/dev/null || true
          
          # Remove .git directory and all git-related files
          echo "Removing .git directory and git-related files..."
          rm -rf "$TARGET_PATH"/.git
          rm -f "$TARGET_PATH"/.gitignore
          rm -f "$TARGET_PATH"/.gitattributes
          rm -f "$TARGET_PATH"/.gitmodules
          rm -f "$TARGET_PATH"/.gitkeep
          
          # Remove README and LICENSE files from root directory if they exist
          echo "Removing README and LICENSE files from root directory..."
          rm -f "$TARGET_PATH"/README*
          rm -f "$TARGET_PATH"/LICENSE*
          rm -f "$TARGET_PATH"/readme*
          rm -f "$TARGET_PATH"/license*
          
          # Clean up
          rm -rf "$TEMP_DIR"
          
          echo "Done! Boilerplate code created at $TARGET_PATH"
      workdir: /repo
      mounts:
        - path: /repo
          source:
            type: host
            path: ''
      env:
        - name: TARGET_PATH
          value: pt.param target-path
        - name: SOURCE_PATH
          value: pt.param source-path       
```

## Usage

```bash
pt run --non-interactive "boilerplate{source-path: <git-repository-url>, target-path: <target-directory>}"
```

## Examples

| Module Type | Repository | Command |
|-------------|------------|---------|
| React app   | https://github.com/bluetext-io/boilerplate-react-web-app | `pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-react-web-app, target-path: modules/web-app}"` |
| Python API  | https://github.com/bluetext-io/boilerplate-python-api | `pt run --non-interactive "boilerplate{source-path: https://github.com/bluetext-io/boilerplate-python-api, target-path: modules/api}"` |

## Parameters

- `source-path`: Git repository URL to clone boilerplate code from (required)
- `target-path`: Path relative to the repo root where the module's code will be created (required)

## How it works

The boilerplate module:
1. Downloads the specified Git repository using `git clone --depth 1`
2. Copies all files (including hidden files) to the target path
3. Explicitly removes the `.git` directory and all git-related files (`.gitignore`, `.gitattributes`, `.gitmodules`, `.gitkeep`)
4. Removes README and LICENSE files from the root directory
5. Cleans up temporary files

## Prerequisites

- Internet connection to access the remote repositories
- Git is automatically installed in the container during execution

## Flexibility

This module can clone any publicly accessible Git repository, not just the predefined boilerplate repositories. You can use it with:
- GitHub repositories
- GitLab repositories  
- Any other Git-compatible repository hosting service
- Public repositories (authentication for private repositories is not currently supported)

Before running the boilerplate module, use the MCP server to fetch the appropriate blueprint resource for Polytope-specific documentation when using the standard blueprints. Use the `access_mcp_resource` tool with the server name `bluetext` and the URI `bluetext://blueprints/<blueprint-id>` where `<blueprint-id>` is the specific blueprint you need.
