# MCP Config File

MCPHub.nvim like other MCP clients uses a JSON configuration file to manage MCP servers. This `config` file is located at `~/.config/mcphub/servers.json` by default and supports real-time updates across all Neovim instances. You can set `config` option to a custom location. 

> [!NOTE]
> You can use a single config file for any MCP client like VSCode, Cursor, Cline, Zed etc as long as the config file follows the below structure. With MCPHub.nvim, `config` file can be safely added to source control as it allows some special placeholder values in the `env` and `headers` fields on MCP Servers.

## Manage Servers

Adding, editing, deleting and securing MCP servers in easy and intuitive with MCP Hub. You don't need to edit the `servers.json` file directly. Everything can be done right from the UI.

### From Marketplace

#### Browse, sort, filter , search from available MCP servers. 

![Image](https://github.com/user-attachments/assets/f5c8adfa-601e-4d03-8745-75180a9d3648)

#### One click AI install with Avante and CodeCompanion
![Image](https://github.com/user-attachments/assets/2d0a0d8b-18ca-4ac8-a207-4758d09d359d)

#### Or Simple copy paste `mcpServers` json block in the README

![Image](https://github.com/user-attachments/assets/359bc81e-d6fe-47bb-a25b-572bf280851e)
<!-- ![Image](https://github.com/user-attachments/assets/f58fcba3-8670-4b4e-998b-cd70b9e6c7ec) -->


### From Hub View

![Image](https://github.com/user-attachments/assets/1cb950da-2f7f-46e9-a623-4cc4b00cc3d0)

Add (`<A>`), edit (`<e>`), delete (`<d>`) MCP servers from the (`H`) Hub view.

## Basic Schema

The `config` file should have a `mcpServers` key. This contains `stdio` and `remote` MCP servers. There is also another top level MCPHub specific field `nativeMCPServers` to store any disabled tools, custom instructions etc that the plugin updates internally. See [Lua MCP Servers](/mcp/native/index) for more about Lua native MCP servers

```json
{
    "mcpServers": {
        // Add stdio and remote MCP servers here
    },
    "nativeMCPServers": { // MCPHub specific
        // To store disabled tools, custom instructions etc
    }
}
```

## Server Types

### Local (stdio) Servers

```json
{
    "mcpServers": {
        "local-server": {
            "command": "uvx",
            "args": ["mcp-server-fetch"]
        }
    }
}
```

##### Required fields:
- `command`: The executable to start the server

##### Optional fields: 
- `args`: Array of command arguments
- `env`: Optional environment variables
- `dev`: Development mode configuration for auto-restart on file changes

##### `env` Special Values

The `env` field supports several special values. Given `API_KEY=secret` in the environment:

| Example | Becomes | Description |
|-------|---------|-------------|
| `"API_KEY": ""` | `"API_KEY": "secret"` | Empty string falls back to `process.env.API_KEY` |
| `"API_KEY": null` | `"SERVER_URL": "secret"` | `null` falls back to `process.env.API_KEY` |
| `"AUTH": "Bearer ${API_KEY}"` | `"AUTH": "Bearer secret"` | `${}` Placeholder values are also replaced | 
| `"TOKEN": "$: cmd:op read op://example/token"`  | `"TOKEN": "secret"` | Values starting with `$: ` will be executed as shell command | 
| `"HOME": "/home/ubuntu"` | `"HOME": "/home/ubuntu"` | Used as-is | 

##### `dev` Development Mode

The `dev` field enables automatic server restarts when files change during development:

```json
{
    "mcpServers": {
        "my-dev-server": {
            "command": "node",
            "args": ["src/index.js"],
            "dev": {
                "enabled": true,
                "cwd": "/absolute/path/to/server/directory"
                "watch": ["src/**/*.js", "src/**/*.ts","**/*.py", "package.json"],
            }
        }
    }
}
```

###### Dev Configuration Options:

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `cwd` | **Yes** | - | Absolute path to server's working directory |
| `watch` | No | `["**/*.js", "**/*.ts", "**/*.py","**/*.json"]` | Array of glob patterns to watch |
| `enabled` | No | `true` | Enable/disable dev mode |

When enabled, the server will automatically restart whenever files matching the watch patterns change in the specified directory. The system uses a 500ms debounce to prevent rapid restarts and ignores common directories like `node_modules`, `build`, `.git`, etc.

**Example use cases:**
- TypeScript/JavaScript MCP servers during development
- Python servers with source code changes
- Configuration file updates that require restarts

### Remote Servers

MCPHub supports both `streamable-http` and `sse` remote servers.

```json
{
    "mcpServers": {
        "remote-server": {
            "url": "https://api.example.com/mcp",
            "headers": {
                "Authorization": "Bearer ${API_KEY}"
            }
        }
    }
}
```

##### Required fields:
- `url`: Remote server endpoint

##### Optional fields:
- `headers`: Optional authentication headers

##### `headers` Special Values

The `headers` field supports `${...}` Placeholder values. Given `API_KEY=secret` in the environment:

| Example | Becomes | Description |
|-------|-------------|---------|
| `"Authorization": "Bearer ${API_KEY}"` |`"AUTH": "Bearer secret"` | `${}` Placeholder values are replaced | 

## MCPHub Specific Fields

MCPHub adds several extra keys for each server automatically from the UI:

```json
{
    "mcpServers": {
        "example": {
            "disabled": false,
            "disabled_tools": ["expensive-tool"],
            "disabled_resources": ["resource://large-data"],
            "disabled_resourceTemplates": ["resource://{type}/{id}"],
            "custom_instructions": {
                "disabled": false,
                "text": "Custom instructions for this server"
            }
        }
    }
}
```



