---
sidebar_position: 2
title: Aura Panel
---

# Aura Panel

Aura Panel is the main component of Aura. It will be used to manage your users, nodes, subscriptions and more.

## Step 1 - Download vital files

```bash title="Creating project directory"
mkdir /opt/aura && cd /opt/aura
```

Download [`docker-compose.yml`][compose-file] and [`.env.sample`][env-file] by running the following commands:

```bash title="Get docker-compose.yml file"
curl -o docker-compose.yml https://raw.githubusercontent.com/localzet/aura-backend/refs/heads/main/docker-compose-prod.yml
```

```bash title="Get .env file"
curl -o .env https://raw.githubusercontent.com/localzet/aura-backend/refs/heads/main/.env.sample
```

## Step 2 - Configure the .env file

`JWT_AUTH_SECRET` and `JWT_API_TOKENS_SECRET` are used for authentication or other purposes.

Generate secret key by running the following command:

```bash title="Generating secure keys"
sed -i "s/^JWT_AUTH_SECRET=.*/JWT_AUTH_SECRET=$(openssl rand -hex 64)/" .env && sed -i "s/^JWT_API_TOKENS_SECRET=.*/JWT_API_TOKENS_SECRET=$(openssl rand -hex 64)/" .env
```

```bash title="Generating passwords"
sed -i "s/^METRICS_PASS=.*/METRICS_PASS=$(openssl rand -hex 64)/" .env && sed -i "s/^WEBHOOK_SECRET_HEADER=.*/WEBHOOK_SECRET_HEADER=$(openssl rand -hex 64)/" .env
```

Also, it is better to change the default Postgres password.

```bash title="Changing Postgres password"
pw=$(openssl rand -hex 24) && sed -i "s/^POSTGRES_PASSWORD=.*/POSTGRES_PASSWORD=$pw/" .env && sed -i "s|^\(DATABASE_URL=\"postgresql://postgres:\)[^\@]*\(@.*\)|\1$pw\2|" .env
```

Now, open the `.env` file, you need to change the following variables:

- `FRONT_END_DOMAIN`
- `SUB_PUBLIC_DOMAIN`

`FRONT_END_DOMAIN` is the domain name under which the panel will be accessible. Just use your domain name here.
Example: `panel.yourdomain.com`.

`SUB_PUBLIC_DOMAIN` – for now just enter your panel domain and add `/api/sub` to the end of it.  
Example: `panel.yourdomain.com/api/sub`.

:::tip Changing environment variables
You can find more information about the environment variables in the [Environment Variables](/docs/install/environment-variables.md) page.
:::

## Step 3 - Start the containers

Start the containers by running the following command:

```bash title="Start the containers"
docker compose up -d && docker compose logs -f -t
```

Wait a few seconds and you will see the following output

![Aura Panel](/install/panel_up.webp)

## Next Steps

:::danger
Reverse proxy is required to properly run Aura Panel.

Do not expose the services to the public internet; use only `127.0.0.1` for Aura services.
:::

Now we are ready to move on to Reverse Proxy installation.

<Button label="Reverse Proxy Installation" link="/docs/install/reverse-proxies/" variant="secondary" size="md" outline style={{ marginBottom: '1rem' }} />

[compose-file]: https://raw.githubusercontent.com/localzet/aura-backend/refs/heads/main/docker-compose-prod.yml
[env-file]: https://raw.githubusercontent.com/localzet/aura-backend/refs/heads/main/.env.sample
