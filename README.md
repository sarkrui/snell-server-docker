# Snell-Server-Docker

This repository packages [Snell Server](https://manual.nssurge.com/others/snell.html) into a Docker image for easy deployment and integration with ShadowTLS. It also supports Shadowsocks + ShadowTLS deployment.

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Basic understanding of proxy servers
- Server with open ports (recommended: 443)

## Deployment Options

### Option 1: Snell + ShadowTLS

```bash
# Create directory and navigate to it
mkdir shadowtls-snell
cd shadowtls-snell

# Download compose file
wget -O compose.yaml https://raw.githubusercontent.com/missuo/snell-server-docker/refs/heads/master/compose-snell.yaml

# Generate random password for enhanced security
PASSWORD=$(openssl rand -base64 32)
echo "Generated password: $PASSWORD"

# Edit the compose file with your configuration
nano compose.yaml

# Start the containers
docker compose up -d

# Check logs
docker compose logs
```

### Option 2: Shadowsocks + ShadowTLS

```bash
# Create directory and navigate to it
mkdir shadowtls-shadowsocks
cd shadowtls-shadowsocks

# Download compose file
wget -O compose.yaml https://raw.githubusercontent.com/missuo/snell-server-docker/refs/heads/master/compose-shadowsocks.yaml

# Generate random password and automatically update compose.yaml
PASSWORD=$(openssl rand -base64 32)
echo "Generated password: $PASSWORD"
ESCAPED_PASSWORD=$(echo "$PASSWORD" | sed 's/[\/&]/\\&/g')
sed -i "s/PASSWORD=CHANGE_ME/PASSWORD=$ESCAPED_PASSWORD/" compose.yaml
if [ $? -eq 0 ]; then
    echo "Password successfully updated in compose.yaml"
else
    perl -i -pe "s/PASSWORD=CHANGE_ME/PASSWORD=$PASSWORD/g" compose.yaml
fi

# Start the containers
docker compose up -d

# Check logs
docker compose logs
```

## Client Configuration

### Surge for iOS/macOS

Add the following to your Surge configuration:

#### For Snell + ShadowTLS:

```ini
[Proxy]
my-snell = snell, your-server-ip, 8888, psk=your-snell-password, version=4, tfo=true, reuse=true, shadow-tls-password=shadowtls-pass, shadow-tls-version=3, shadow-tls-sni=weather-data.apple.com
```

#### For Shadowsocks + ShadowTLS:

> Surge User
```ini
[Proxy]
my-shadowsocks = ss, your-server-ip, 8443, encrypt-method=chacha20-ietf-poly1305, password=shadowsocks-pass, reuse=true, shadow-tls-password=shadowtls-pass, shadow-tls-version=3, shadow-tls-sni=weather-data.apple.com
```

> Clash Meta (Minoho core) User
```yaml
proxies:
- name: "your-proxy-name"
  type: ss    
  server: your-server-ip
  port: 8443
  cipher: chacha20-ietf-poly1305
  password: your-shadowsocks-password
  udp: true
  udp-over-tcp: false
  udp-over-tcp-version: 2
  ip-version: ipv4
  smux:
    enabled: false
  plugin: shadow-tls
  client-fingerprint: chrome
  plugin-opts:
    host: "weather-data.apple.com"
    password: your-shadow-tls-password
    version: 3
```

### Shadowrocket

Shadowrocket supports Shadowsocks with ShadowTLS plugin. You can either manually fill in the configuration fields, or import the Clash Meta YAML configuration file - Shadowrocket will automatically parse it into its native format.