# Reverse Proxy Server

This project implements a reverse proxy server that routes requests to different services based on the URL prefix. The server listens for requests and proxies them to the appropriate target service defined in the configuration. It includes automatic TLS certificate generation and renewal through Let's Encrypt for secure HTTPS connections.

## Features

- Dynamic request routing based on domain + prefix
- Automatic SSL/TLS certificate management via Let's Encrypt
- Zero-downtime certificate renewal
- Support for multiple domains
- Configurable routing rules
- Global rate limiting with cooldown

## Getting Started

### Prerequisites

- [Go](https://golang.org/dl/) installed on your machine
- Domain names pointed to your server's IP address (for SSL certificates)
- Port 80 and 443 available for HTTP/HTTPS traffic

### Configuration

Configurations are stored on the root's `config.yaml`.

#### Domain Configuration

The domains configuration specifies which domains the proxy will handle. Example:

```yaml
domains:
  - "domain.com"
  - "api.domain.com"
  - "sub.domain.com"
```

#### Routes Configuration

Routes define how incoming requests are routed different services. Example:

```yaml
routes:
  "domain.com": "http://localhost:4000"
  "domain.com/api": "http://localhost:4001"
```

#### Rate Limiting

Rate limiting defines how many request a client can do in a specified timeframe. Example:

```yaml
rate_limit:
  burst: 50
  rate: 10
  cooldown: 60000
```

`burst` Maximum requests allowed in a short period. This enables handling sudden traffic spikes.

`rate` Requests per second at which tokens are replenished.

`cooldown` Time (in milliseconds) a client must wait after exhausting the burst limit.

#### How it Works

- `burst`: Client can make up to burst requests (50 in the example) without hitting the limit.
- `rate`: After the burst, requests are limited to rate tokens per second (10 in this example).
- `cooldown`: After exceeding the burst, the client must wait for the cooldown period (60 seconds) before making more requests.

### SSL Certificates

The server automatically manages SSL certificates through Let's Encrypt using [certmagic](https://github.com/caddyserver/certmagic):
- Certificates are obtained when the server starts
- Automatic renewal before expiration
- Certificates are cached locally for reuse

### Running the Server

1. Set up your configuration file
2. You have two options to run the server:

#### Option 1: Direct Run
```bash
go run cmd/server/main.go
```

#### Option 2: Systemd Deployment
There's a `build-deploy.sh` script that automates the deployment process and sets up the server as a systemd service:

1. Run the deployment script:
```bash
chmod +x ./build-deploy.sh
./build-deploy.sh
```

This script will:
- Build the Go binary
- Install and enable the service
- Start the server

You can configure the `proxy.service` as needed, but you need to keep the:
```
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
```
as the reverse proxy server needs to bind on privileged ports, `80` and `443`.

To check the service status:
```bash
systemctl status proxy
```

To view logs:
```bash
journalctl -u proxy -f
```
