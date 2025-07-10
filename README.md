# iobnode - ioBroker Development Server

A Docker-based development environment for ioBroker developers with SSH access, Git integration, and the official ioBroker Dev-Server.

## Overview

**iobnode** provides a complete development environment for:

- ioBroker adapter developers
- Contributors to ioBroker core
- Teams requiring consistent development environments
- Developers using remote development via SSH

## Features

### Included Software

- **Node.js**: Version 20.19.1
- **npm**: Bundled with Node.js
- **Yarn**: Version 1.22.19
- **@iobroker/dev-server**: Globally installed
- **Development Tools**: gcc, g++, make, git, vim, curl, sudo
- **SSH Server**: For remote access

### System Setup

- **Base OS**: Debian Bookworm Slim
- **Main User**: `node` (UID: 1000, GID: 1000)
- **Dev User**: `iobdev` (for development work)
- **Architecture**: Multi-arch support (amd64, arm64, armhf, etc.)

### Exposed Ports

- **22**: SSH access
- **8081**: ioBroker Dev-Server

## Quick Start

### Docker Run

```bash
docker run -d \
  --name iobroker-dev \
  -p 2222:22 \
  -p 8081:8081 \
  -e GIT_USERNAME="Your Name" \
  -e GIT_EMAIL="your.email@example.com" \
  -v $(pwd)/workspace:/workspace \
  -v ~/.ssh/id_rsa.pub:/root/.ssh/authorized_keys:ro \
  marc-berg/iobnode:latest
```

### Docker Compose

```yaml
version: '3.8'
services:
  iobroker-dev:
    image: marc-berg/iobnode:latest
    container_name: iobroker-dev
    ports:
      - "2222:22"
      - "8081:8081"
    environment:
      - GIT_USERNAME=Your Name
      - GIT_EMAIL=your.email@example.com
    volumes:
      - ./workspace:/workspace
      - ~/.ssh/id_rsa.pub:/root/.ssh/authorized_keys:ro
    restart: unless-stopped
```

### Build from Source

```bash
git clone https://github.com/Marc-Berg/iobnode.git
cd iobnode
docker build -t marc-berg/iobnode .
```

## Configuration

### Environment Variables

|Variable      |Description             |Required|
|--------------|------------------------|--------|
|`GIT_USERNAME`|Git username for commits|Yes     |
|`GIT_EMAIL`   |Git email for commits   |Yes     |

### SSH Access Setup

1. **Generate SSH key** (if you don’t have one):
   
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
   ```
1. **Mount public key**:
   
   ```bash
   -v ~/.ssh/id_rsa.pub:/root/.ssh/authorized_keys:ro
   ```
1. **Connect via SSH**:
   
   ```bash
   ssh -p 2222 root@localhost
   ```

## Development Workflow

### 1. Start Container

```bash
docker run -d --name iobroker-dev \
  -p 2222:22 -p 8081:8081 \
  -e GIT_USERNAME="Developer Name" \
  -e GIT_EMAIL="dev@example.com" \
  -v $(pwd)/projects:/workspace \
  marc-berg/iobnode
```

### 2. Connect via SSH

```bash
ssh -p 2222 root@localhost
```

### 3. Start Development

```bash
# Inside the container
cd /workspace
mkdir my-adapter
cd my-adapter

# Start ioBroker Dev-Server
iobroker-dev
```

### 4. Access Dev Interface

Open `http://localhost:8081` in your browser to access the ioBroker development interface.

## ioBroker Dev-Server Commands

The installed `@iobroker/dev-server` provides:

```bash
# Start dev server
iobroker-dev start

# Start with specific port
iobroker-dev start --port 8081

# Start in debug mode
iobroker-dev start --debug

# Install adapter
iobroker-dev add adapter-name

# View logs
iobroker-dev logs
```

## Volume Management

### Recommended Volume Structure

```
project-root/
├── workspace/          # Main development folder
│   ├── adapters/      # Adapter projects
│   ├── scripts/       # Development scripts
│   └── configs/       # Configuration files
├── ssh-keys/          # SSH keys
└── docker-compose.yml
```

### Volume Mounting

```yaml
volumes:
  - ./workspace:/workspace                    # Development folder
  - ./ssh-keys/authorized_keys:/root/.ssh/authorized_keys:ro
  - ./configs:/etc/iobroker:ro               # Optional configurations
```

## IDE Integration

### Visual Studio Code

#### Remote-SSH Extension

1. Install “Remote - SSH” extension
1. Add SSH configuration:
   
   ```
   Host iobroker-dev
     HostName localhost
     Port 2222
     User root
   ```
1. Connect to container

#### Docker Extension

- Manage containers directly from VS Code
- Access logs and terminal
- File system browser

### JetBrains IDEs

- Use SSH remote development
- Docker plugin for container management

## Troubleshooting

### Common Issues

#### SSH Connection Fails

```bash
# Check SSH logs
docker logs iobroker-dev

# Check SSH service in container
docker exec iobroker-dev ps aux | grep ssh
```

#### Git Configuration Missing

```bash
# Set manually
docker exec iobroker-dev git config --global user.name "Name"
docker exec iobroker-dev git config --global user.email "email@example.com"
```

#### Node.js/npm Issues

```bash
# Check Node.js version
docker exec iobroker-dev node --version

# Clear npm cache
docker exec iobroker-dev npm cache clean --force
```

#### Permission Problems

```bash
# Start container as specific user
docker run --user 1000:1000 ...

# Fix permissions
docker exec iobroker-dev chown -R node:node /workspace
```

### Debug Commands

```bash
# View container logs
docker logs -f iobroker-dev

# Enter container
docker exec -it iobroker-dev /bin/bash

# Check processes
docker exec iobroker-dev ps aux

# Network status
docker exec iobroker-dev netstat -tlnp
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: ioBroker Adapter Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      iobroker-dev:
        image: marc-berg/iobnode:latest
        ports:
          - 8081:8081
        env:
          GIT_USERNAME: CI
          GIT_EMAIL: ci@example.com

    steps:
      - uses: actions/checkout@v2
      - name: Test Adapter
        run: |
          npm test
```

## Maintenance

### Update Container

```bash
# Pull new image
docker pull marc-berg/iobnode:latest

# Recreate container
docker stop iobroker-dev
docker rm iobroker-dev
docker run -d --name iobroker-dev ... # with same parameters
```

### Backup Development Data

```bash
# Backup workspace
tar -czf workspace-backup.tar.gz workspace/

# Backup container volumes
docker run --rm -v iobroker-dev-data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/container-backup.tar.gz /data
```

## Development Best Practices

### Code Organization

```
workspace/
├── my-adapter/
│   ├── src/
│   ├── test/
│   ├── package.json
│   └── README.md
├── shared-libs/
└── docs/
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/new-feature

# Regular commits
git add .
git commit -m "feat: add new feature"

# Push to remote
git push origin feature/new-feature
```

### Testing

```bash
# Unit tests
npm test

# Integration tests with dev server
iobroker-dev test

# Linting
npm run lint
```

## Dockerfile Structure

The Dockerfile implements:

1. **Base Setup**: Debian Bookworm Slim
1. **Node.js Installation**: Secure verification with GPG
1. **Yarn Installation**: Package manager for better dependency management
1. **Development Tools**: Compilers and Git tools
1. **ioBroker Dev-Server**: Global installation
1. **SSH Configuration**: Remote access setup
1. **Entrypoint Script**: Automated configuration

## Community and Support

### Official Resources

- **ioBroker Forum**: [forum.iobroker.net](https://forum.iobroker.net)
- **ioBroker Discord**: Developer community
- **GitHub**: [github.com/ioBroker](https://github.com/ioBroker)

### Development Documentation

- **Adapter Development**: ioBroker Developer Portal
- **API Reference**: ioBroker Core Documentation
- **Best Practices**: Community Guidelines

### Contributing

- Pull requests for improvements
- Issue reports for bugs
- Documentation updates
- Testing and feedback

## License

This project follows open-source principles. Contributions are welcome via pull requests, issue reports, documentation improvements, and testing feedback.

-----

*This Docker image enables a professional ioBroker development environment with all necessary tools for modern adapter development.*
