<h1 align="center">gitea-gh-mirror</h1>

<p align="center">
    <img src="./assets/logo.png" alt="gitea-gh-mirror logo" width="50%">
</p>

`gitea-gh-mirror` provides a ready-to-use Docker Compose setup for running a self-hosted [Gitea](https://about.gitea.com/) instance alongside the [gitea-mirror](https://github.com/RayLabsHQ/gitea-mirror) tool, so you can mirror GitHub repositories to your own server.

It is designed for homelab users, self-hosters, and developers who want a simple way to keep an independent copy of their GitHub repositories.

## Why This Exists

gitea-gh-mirror was created to make mirroring GitHub repositories to a self-hosted Gitea instance simple and hassle free with a very easy and developer friendly setup.
With the rise of supply chain attacks and security incidents on GitHub, having a reliable backup of your work matters more than ever. Whether GitHub goes down, an account gets flagged, or something else entirely goes wrong, your code stays safe and accessible on infrastructure you control.

## Prerequisites

Before running this project, you need:

- Docker
- Docker Compose
- Git
- A GitHub account
- A GitHub access token with access to the repositories you want to mirror

## Quick Start

Clone the repository:

```sh
git clone https://github.com/homelabshq/gitea-gh-mirror
cd gitea-gh-mirror
```

Start the services:

```sh
docker compose up -d
```

Then open:

- Gitea: `http://<your-server-ip>:8085`
- gitea-mirror: `http://<your-server-ip>:4321`
- Gitea SSH: `<your-server-ip>:2222`

If your system still uses the legacy Docker Compose binary, use `docker-compose` instead of `docker compose`.

## First-Time Setup

After the containers are running:

1. Open Gitea at `http://<your-server-ip>:8085`.
2. Create or configure your Gitea user/admin account.
3. Create a token of your Gitea account which is used by gitea-mirror to access your Gitea instance.
4. Open gitea-mirror at `http://<your-server-ip>:4321`.
5. Add your GitHub Account and access token.
6. Add your Gitea Account and access token.
7. Select the repositories you want to mirror.
8. Start the mirror process.

## Configuration

Most runtime configuration is currently managed in `docker-compose.yml`.

Before exposing this setup beyond your local network, update the default passwords and secrets in `docker-compose.yml`, especially:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_PASSWORD`
- `GITEA__database__PASSWD`
- `BETTER_AUTH_SECRET`

## Data and Backups

All persistent data is stored under:

```text
./data/
```

Back up the entire `./data` directory to preserve your Gitea instance, repository mirrors, database, and mirror configuration.

### Follow the 3-2-1 Backup Strategy

For important repositories and self-hosted services, you should always follow the **3-2-1 backup strategy**:

- Keep **3 copies** of your data
- Store them on **2 different types of storage/media**
- Keep **1 copy off-site**

This project already helps with the first part by creating a self-hosted mirror of your GitHub repositories. However, you should still back up the `./data` directory separately.

For example:

- One copy on GitHub
- One copy on your self-hosted Gitea server
- One additional backup on external storage, NAS, or cloud storage

In the future, this project may include a built-in or documented way to back up the `./data` directory to cloud storage as part of a proper 3-2-1 backup strategy.

## Common Commands

Start services:

```sh
docker compose up -d
```

Stop services:

```sh
docker compose down
```

View logs:

```sh
docker compose logs -f
```

View logs for a specific service:

```sh
docker compose logs -f server
docker compose logs -f db
docker compose logs -f gitea-mirror
```

Restart services:

```sh
docker compose restart
```

Stop and remove containers while keeping data:

```sh
docker compose down
```

Stop and remove containers, networks, and anonymous volumes:

```sh
docker compose down --volumes
```

> Be careful with volume/data removal commands. Always back up `./data` before destructive maintenance.

## Updating

Pull the latest changes and restart the containers:

```sh
git pull --ff-only
docker compose up -d --build
docker image prune -f
```

> Always make sure you have no uncommitted local changes before updating. Run `git status` to check.

## Troubleshooting

### Permission Denied on Linux

This setup runs services as UID/GID `1000:1000`. On Linux, bind-mounted directories preserve host filesystem ownership, so incorrect ownership can cause errors such as:

```text
/var/lib/gitea/git is not writable
cannot create /app/data/.encryption_secret: Permission denied
Errcode: 13 "Permission denied"
```

The Compose file includes an `init-permissions` service that prepares the `./data` directory automatically. If you still see permission errors, run:

```sh
sudo mkdir -p data/gitea/data data/gitea/config data/db
sudo chown -R 1000:1000 data
docker compose up -d
```

### Ports Already in Use

By default, this project uses:

- `8085` for Gitea HTTP
- `2222` for Gitea SSH
- `4321` for gitea-mirror

If any of these ports are already in use, update the port mappings in `docker-compose.yml`.

Example:

```yml
ports:
  - "8086:8085"
```

This would expose Gitea on host port `8086` while keeping container port `8085` unchanged.

### Containers Keep Restarting

Check logs with:

```sh
docker compose logs -f
```

Common causes include:

- Incorrect file permissions under `./data`
- Database startup issues
- Port conflicts
- Invalid environment variable values
- Changed or missing secrets

## Security Notes

Before using this setup publicly or on an untrusted network:

- Change all default passwords and secrets.
- Use strong, unique values for database passwords and auth secrets.
- Back up `./data` regularly.

## License

This project is open source and available under the [MIT License](./LICENSE).
