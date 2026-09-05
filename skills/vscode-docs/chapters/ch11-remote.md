# Chapter 11: Remote Development

## Core Idea
VS Code enables remote development through SSH, containers, WSL, and tunnels, providing a full IDE experience on remote machines.

## Frameworks Introduced
- **Remote - SSH**: Develop on remote machines via SSH connection
- **Remote - WSL**: Develop in Windows Subsystem for Linux
- **Remote - Containers**: Develop inside Docker containers
- **Remote - Tunnels**: Connect to remote machines without SSH configuration
- **VS Code Server**: Backend component that runs on remote machines

## Key Concepts
- **Remote Connection**: Connect to remote machine while running VS Code locally
- **Port Forwarding**: Automatically forward ports from remote to local
- **Remote Extensions**: Extensions run on remote machine, not locally
- **Workspace Trust**: Security model for remote folders
- **Tunnel**: Secure connection to remote machine without SSH setup

## Mental Models
- Use **Remote - SSH** for Linux server development
- Use **Remote - WSL** for Linux development on Windows
- Use **Remote - Containers** for reproducible development environments
- Use **Tunnels** for quick access to remote machines

## Anti-patterns
- **Storing credentials in plain text**: Use SSH keys and credential managers
- **Skipping workspace trust**: Always verify remote folder trust before opening
- **Ignoring network latency**: Use appropriate timeouts and keep connections alive

## Code Examples
```json
// .vscode/settings.json (remote development)
{
  "remote.SSH.defaultExtensions": [
    "ms-python.python",
    "dbaeumer.vscode-eslint"
  ],
  "remote.SSH.connectTimeout": 30,
  "remote.SSH.serverKeepAliveIntervalMinutes": 5
}
```

## Key Takeaways
1. Remote extensions run on the remote machine — install them there
2. Port forwarding happens automatically; check Ports panel for status
3. SSH config in `~/.ssh/config` is respected by Remote - SSH
4. Tunnels provide quick access without SSH key setup
5. Use remote indicator (bottom-left) to switch between local and remote

## Connects To
- **Ch 10**: Containers are a common remote development target
- **Ch 12**: Debugging remotely
- **Ch 1**: Local setup vs. remote development
