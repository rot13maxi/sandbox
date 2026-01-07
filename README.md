# Docker Sandbox for AI Agents

A shell script that replicates Docker Desktop's sandbox functionality for plain Docker on Linux. Run AI coding agents (Claude Code, OpenCode, Aider, etc.) in isolated containers with your workspace and configs mounted.

## Features

- **One container per workspace**: Automatically reuses existing sandboxes for the same workspace/agent
- **Workspace mounting**: Your project directory is mounted at the same path inside the container
- **Config mounting**: Agent configs (`~/.claude`, `~/.anthropic`, etc.) automatically mounted
- **Git integration**: Git user name/email injected for proper commit attribution
- **SSH keys**: `~/.ssh` mounted read-only for git operations
- **Persistent state**: Agent credentials stored in Docker volumes
- **Docker socket**: Optional Docker socket access for agents that need it
- **Custom templates**: Use your own container images

## Installation

```bash
# Clone or download the script
curl -o sandbox https://raw.githubusercontent.com/your-repo/sandbox
chmod +x sandbox

# Move to your PATH
sudo mv sandbox /usr/local/bin/

# Or add to your PATH
export PATH="$PATH:/path/to/sandbox/directory"
```

## Quick Start

```bash
# Navigate to your project
cd ~/projects/myapp

# Run Claude Code in a sandbox
sandbox run claude

# Or run OpenCode
sandbox run opencode
```

## Usage

### Run an agent

```bash
sandbox run [options] <agent> [agent-args...]

Options:
  -e, --env KEY=VALUE       Set environment variable
  -v, --volume SRC:DST      Mount additional volume
  -w, --workspace PATH      Use specified workspace (default: current directory)
  -d, --detach              Run in background
  --mount-docker-socket     Mount Docker socket into container
  --template IMAGE          Use custom container image
  --force-recreate          Remove existing sandbox and create new one
```

### List sandboxes

```bash
sandbox ls          # Show all sandboxes
sandbox ls -q       # Show only IDs
```

### Manage sandboxes

```bash
sandbox stop <id>       # Stop a running sandbox
sandbox start <id>      # Start and attach to a sandbox
sandbox start -d <id>   # Start in background
sandbox rm <id>         # Remove a sandbox
sandbox rm -f <id>      # Force remove
sandbox inspect <id>    # Show detailed info (JSON)
sandbox exec <id> bash  # Execute command in sandbox
sandbox logs <id>       # View container logs
```

## Supported Agents

| Agent    | Image                                    |
|----------|------------------------------------------|
| claude   | docker/sandbox-templates:claude-code     |
| opencode | ghcr.io/anomalyco/opencode:latest        |
| aider    | paulgauthier/aider                       |

## Examples

### Basic usage

```bash
# Run Claude in current directory
cd ~/myproject
sandbox run claude
```

### With environment variables

```bash
sandbox run -e NODE_ENV=development -e DEBUG=true claude
```

### Mount additional volumes

```bash
# Mount datasets read-only and models read-write
sandbox run \
  -v ~/datasets:/data:ro \
  -v ~/models:/models \
  claude
```

### Docker socket access

```bash
# Give agent access to build/run Docker containers
sandbox run --mount-docker-socket claude
```

### Custom template image

Create a Dockerfile with your tools pre-installed:

```dockerfile
FROM docker/sandbox-templates:claude-code
RUN apt-get update && apt-get install -y nodejs npm
```

Build and use it:

```bash
docker build -t my-dev-env .
sandbox run --template my-dev-env claude
```

### Different workspace

```bash
sandbox run -w ~/other-project claude
```

### Force recreate sandbox

```bash
# Useful when you want to change config (env vars, volumes, etc.)
sandbox run --force-recreate -e NEW_VAR=value claude
```

## How It Works

1. **Container creation**: Creates a container from the agent's template image
2. **Workspace mounting**: Mounts your current directory at the same absolute path
3. **Config mounting**: Mounts agent config directories from your home directory
4. **Git injection**: Reads `git config` and injects name/email as environment variables
5. **One per workspace**: Uses labels to track workspace/agent combinations
6. **Reuse**: Subsequent runs in the same directory attach to the existing container

## What Gets Mounted

| Host Path         | Container Path     | Mode       |
|-------------------|-------------------|------------|
| `$PWD`            | `$PWD`            | read-write |
| `~/.claude`       | `/root/.claude`   | read-write |
| `~/.anthropic`    | `/root/.anthropic`| read-write |
| `~/.opencode`     | `/root/.opencode` | read-write |
| `~/.ssh`          | `/root/.ssh`      | read-only  |
| `~/.gitconfig`    | `/root/.gitconfig`| read-only  |

## Differences from Docker Desktop Sandboxes

| Feature                     | Docker Desktop | This Script     |
|-----------------------------|----------------|-----------------|
| Requires Docker Desktop     | Yes            | No              |
| Works on headless Linux     | No             | Yes             |
| GUI integration             | Yes            | No (CLI only)   |
| `docker sandbox` command    | Yes            | `sandbox` script|
| Authentication flow         | Integrated     | Manual          |

## Troubleshooting

### Agent binary not found

If you get "binary not found" errors, you may need to use a different image:

```bash
# Check what's in the image
docker run --rm -it docker/sandbox-templates:claude-code which claude

# Or use a custom template with the binary installed
sandbox run --template my-image-with-agent claude
```

### Permission denied on mounted volumes

Ensure the files/directories exist and are readable:

```bash
ls -la ~/.claude
```

### Container keeps restarting

Check logs:

```bash
sandbox logs <id>
```

### Want to change sandbox config

Remove and recreate:

```bash
sandbox rm <id>
sandbox run -e NEW_CONFIG=value claude
```

Or use `--force-recreate`:

```bash
sandbox run --force-recreate -e NEW_CONFIG=value claude
```

## License

MIT
