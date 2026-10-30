# AGENTS.md

This file contains guidelines and commands for agentic coding agents working in the docker-dnsmasq repository.

## Project Overview

This is a Docker container for dnsmasq with a web UI, forked from jpillora/dnsmasq. The project is minimal with only a few key files:
- `Dockerfile` - Multi-stage Alpine-based container build
- `dnsmasq.conf` - Default dnsmasq configuration
- `README.md` - Usage documentation
- `.github/workflows/` - CI/CD pipelines for Docker builds and security scanning

## Build and Development Commands

### Docker Commands
```bash
# Build the Docker image
docker build -t leto1210/docker-dnsmasq:latest .

# Run the container for development
docker run --name dnsmasq-dev -d \
  -p 53:53/udp \
  -p 5380:8080 \
  -v $(pwd)/dnsmasq.conf:/etc/dnsmasq.conf \
  -e "HTTP_USER=admin" \
  -e "HTTP_PASS=admin" \
  leto1210/docker-dnsmasq:latest

# Test DNS resolution
host myhost.company localhost

# Reload dnsmasq config without container restart
docker exec dnsmasq-dev pkill -HUP dnsmasq

# View container logs
docker logs dnsmasq-dev

# Access web UI
# Open http://localhost:5380 in browser
```

### Testing Commands
```bash
# No automated test suite exists. Manual testing required:

# 1. Test basic DNS functionality
docker run --rm -p 53:53/udp leto1210/docker-dnsmasq:latest &
sleep 5
dig @localhost google.com

# 2. Test custom host resolution
dig @localhost myhost.company

# 3. Test web UI accessibility
curl -I http://localhost:5380

# 4. Test configuration reload
docker exec <container_name> pkill -HUP dnsmasq
```

### Security and Linting
```bash
# Security scanning (requires Trivy)
trivy image leto1210/docker-dnsmasq:latest

# Dockerfile linting (hadolint)
hadolint Dockerfile

# Check for Alpine package updates
docker run --rm alpine:3.23.2 apk list --upgradable
```

## Code Style Guidelines

### Dockerfile Conventions
- Use Alpine Linux base images for minimal size
- Pin specific versions (e.g., `alpine:3.23.2`)
- Combine RUN commands with && to reduce layers
- Use multi-line strings for complex commands with \ continuation
- Order instructions: FROM → RUN → COPY → EXPOSE → ENTRYPOINT
- Use `--no-cache` flag for apk installs
- Clean up package caches in same RUN layer
- Set maintainer and schema labels

### Configuration File Style
- Use `dnsmasq.conf` with INI-style syntax
- Include descriptive comments explaining each directive
- Group related configurations together
- Use consistent spacing around = signs
- End file with newline

### Shell Script Style (if added)
- Use `#!/bin/bash` shebang
- Quote variables: `"$VAR"`
- Use `set -euo pipefail` for error handling
- Prefer functions over one-liners
- Use 2-space indentation for shell scripts

## File Structure and Organization

```
docker-dnsmasq/
├── Dockerfile              # Main container definition
├── dnsmasq.conf           # Default dnsmasq configuration
├── README.md              # User documentation and examples
├── LICENSE                # MIT license
├── .gitignore             # macOS and build artifacts
├── .github/
│   ├── workflows/         # CI/CD pipelines
│   │   ├── docker-image.yml    # Docker build and push
│   │   ├── trivyV2.yml         # Security scanning
│   │   └── dependency-review.yml
│   └── dependabot.yml     # Dependency updates
└── .devcontainer/
    └── devcontainer.json  # VS Code dev container config
```

## Development Workflow

### Making Changes
1. Always test Docker build after changes: `docker build .`
2. Test container functionality: `docker run --rm -it <image>`
3. Update README.md if adding new features or changing usage
4. Update version numbers in Dockerfile ENV variables if needed
5. Test configuration reload functionality

### Security Considerations
- Run containers as non-root user when possible
- Use specific image tags, not `latest`
- Regularly update Alpine packages
- Scan images for vulnerabilities before release
- Minimize exposed ports (only 53/udp and 8080 needed)

### Release Process
1. Update version in Dockerfile ENV variables
2. Test build and functionality thoroughly
3. Push to Docker Hub with version tag
4. Update GitHub releases with changelog
5. CI/CD will automatically handle security scanning

## Common Tasks

### Adding New dnsmasq Features
1. Update `dnsmasq.conf` with new directives
2. Document in README.md with examples
3. Test functionality in container
4. Consider adding environment variable support

### Updating Base Image
1. Change FROM line in Dockerfile
2. Test all functionality
3. Update documentation if needed
4. Run security scans

### Debugging Container Issues
```bash
# Interactive shell for debugging
docker run --rm -it --entrypoint /bin/sh leto1210/docker-dnsmasq:latest

# Check dnsmasq process
docker exec <container> ps aux | grep dnsmasq

# Check configuration
docker exec <container> cat /etc/dnsmasq.conf

# Test DNS from inside container
docker exec <container> nslookup google.com
```

## Environment Variables

The container supports these environment variables:
- `HTTP_USER` - Web UI username (default: none)
- `HTTP_PASS` - Web UI password (default: none)

## Important Notes

- This project has no traditional code tests - testing is done via container functionality
- The web UI is provided by webproc, not custom code
- dnsmasq configuration is the primary customization point
- Docker image size should be kept minimal (Alpine-based)
- Security scanning is automated via GitHub Actions
- No package.json, Makefile, or traditional build system exists
