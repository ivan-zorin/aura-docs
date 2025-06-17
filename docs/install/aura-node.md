---
sidebar_position: 4
title: Aura Node
---

# Aura Node

Aura Node is a lightweight container with included Xray-core.

:::note

Aura Panel is not contains Xray-core inside, so you need to install Aura Node on a separate server in order to fully use Aura.
:::

## Step 1 - Creating project directory

```bash title="Creating project directory"
mkdir /opt/auranode && cd /opt/auranode
```

## Step 2 - Configure the .env file

```bash title="Creating .env file"
nano .env
```

:::tip
`SSL_CERT` can be found in the main panel under the Nodes tab, Management page, after clicking the **Create new node** button. `APP_PORT` can be customized, make sure it's not being used by other services.
:::

```bash title=".env file content"
APP_PORT=2222

SSL_CERT=CERT_FROM_MAIN_PANEL
```

## Step 4 - Create docker-compose.yml file

```bash title="Creating docker-compose.yml file"
nano docker-compose.yml
```

Paste the following content into the file:

```yaml title="docker-compose.yml file content"
services:
    auranode:
        container_name: auranode
        hostname: auranode
        image: localzet/aura-node:latest
        restart: always
        network_mode: host
        env_file:
            - .env
```

## Step 5 - Start the containers

Start the containers by running the following command:

```bash title="Start the containers"
docker compose up -d && docker compose logs -f -t
```

## Advanced usage

### GeoSite files

You can mount additional geosite files into the `/usr/local/share/xray/` directory in the container.

:::caution
Do not mount the entire folder. Otherwise, you will overwrite the default Xray geosite files. Mount each file individually.
:::

Add the following to the `docker-compose.yml` file:

```yaml
services:
    auranode:
        container_name: auranode
        hostname: auranode
        image: localzet/aura-node:latest
        restart: always
        network_mode: host
        env_file:
            - .env
        // highlight-next-line-green
        volumes:
            // highlight-next-line-green
            - './zapret.dat:/usr/local/share/xray/zapret.dat'
```

Usage in xray config:

```json
  "routing": {
    "rules": [
       // Other rules
      {
        "type": "field",
        "domain": [
          "ext:zapret.dat:zapret"
        ],
        "inboundTag": [ // Optional
          "VLESS_TCP_REALITY"
        ],
        "outboundTag": "NOT_RU_OUTBOUND"
      }
      // Other rules
    ]
  }
```

### Log from Node

You can access logs from the node by mounting them to your host's file system.

:::caution
You **must** set up log rotation, otherwise the logs will fill up your disk!
:::

Add the following to the `docker-compose.yml` file:

```yaml
services:
    auranode:
        container_name: auranode
        hostname: auranode
        image: localzet/aura-node:latest
        restart: always
        network_mode: host
        env_file:
            - .env
        // highlight-next-line-green
        volumes:
            // highlight-next-line-green
            - '/var/log/auranode:/var/log/auranode'
```

Usage in xray config:

```json
  "log": {
      "error": "/var/log/auranode/error.log",
      "access": "/var/log/auranode/access.log",
      "loglevel": "warning"
  }
```

On the server where the node is hosted, create the folder `/var/log/auranode`:

```bash
mkdir -p /var/log/auranode
```

Install logrotate (if not already installed):

```bash
sudo apt update && sudo apt install logrotate
```

Create a logrotate configuration file:

```bash
nano /etc/logrotate.d/auranode
```

Paste the following logrotate configuration for RemnaNode:

```bash
/var/log/auranode/*.log {
      size 50M
      rotate 5
      compress
      missingok
      notifempty
      copytruncate
  }
```

Run logrotate manually to test:

```bash
logrotate -vf /etc/logrotate.d/auranode
```

### XRay SSL cert for Node

If you’re using certificates for your XRay configuration, you need to mount them into the panel.

:::info
Mount the folder via Docker volumes, and in the config refer to the internal path.
Inside the container there’s a dedicated (empty) folder for certs:
/var/lib/remnawave/configs/xray/ssl/
:::

Add the following to the `docker-compose.yml` file:

```yaml
aura-backend:
  image: localzet/aura-backend:latest
  container_name: 'remnawave'
  hostname: remnawave
  restart: always
  ports:
    - '127.0.0.1:3000:3000'
  env_file:
    - .env
  networks:
    - aura-network
  // highlight-next-line-green
  volumes:
      // highlight-next-line-green
      - '/opt/aura/nginx:/var/lib/remnawave/configs/xray/ssl'
  depends_on:
    aura-db:
      condition: service_healthy
    aura-redis:
      condition: service_healthy
```

:::info
When the panel pushes the config to the node, it will automatically read the mounted files and send the certs to the node.
:::

Usage in XRay config:

```json
  "certificates": [
    {
    "keyFile": "/var/lib/remnawave/configs/xray/ssl/privkey.key",
    "certificateFile": "/var/lib/remnawave/configs/xray/ssl/fullchain.pem"
    // Other fields
    }
  ]
```

:::caution
Pay attention to the **.key** and **.pem** extensions.
:::
