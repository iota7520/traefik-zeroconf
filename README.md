# Traefik Zero-Conf

## Introduction

[Traefik](https://doc.traefik.io/traefik/) is a well-known open-source reverse proxy. I use this standard configuration in my homelab as well as in production environments.

This setup avoids using any [configuration file](https://doc.traefik.io/traefik/reference/install-configuration/providers/others/file/)  (neither **_traefik.toml_** nor **_traefik_dynamic.toml_**). All the magic happens directly within the Traefik container.

### Prerequisites

To get started, ensure you have the following:

- A Linux-based server with a **sudo-enabled non-root user**.
- [**Docker**](https://docs.docker.com/engine/install/) and [**Compose**](https://docs.docker.com/compose/install/) installed on your server.
- **"A records"** in your domain's DNS pointing to the server's IP address (e.g., _traefik.example.com_).

---

### What Does It Do?

This setup provides the following features:

- Launches a Docker container running the latest [Traefik image](https://hub.docker.com/_/traefik).
- Configures [entrypoints](https://doc.traefik.io/traefik/routing/entrypoints/) for HTTP (port 80) and HTTPS (port 443).
- Sets up a certificate resolver using [Let's Encrypt](https://doc.traefik.io/traefik/https/acme/).
- Enables the [Dashboard](https://doc.traefik.io/traefik/operations/dashboard/).
- Creates a [BasicAuth](https://doc.traefik.io/traefik/middlewares/http/basicauth/) middleware for securing the dashboard.
- Listens to the [Docker socket](https://doc.traefik.io/traefik/providers/docker/#docker-api-access).
- Configures itself via [labels](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/#routing-configuration-with-labels) to:
  - Route requests to _traefik.example.com_ to the dashboard.
  - Use HTTPS exclusively and generate valid certificates through Let's Encrypt.
  - Forward relevant HTTP headers to the container.
  - Enforce HTTP Strict Transport Security (HSTS).

Once set up, you can reuse these labels to manage your own containers.

---

## Usage

1. Clone this repository.
2. Customize the `.env` file (see below).
3. Create an `acme.json` file to store certificates.
4. Update the permissions of `acme.json`.
5. Launch and enjoy!

---

### The `.env` File

Use the provided `.env.sample` file as a template to create your own configuration:

| Variable                     | Default Value                              | Notes                                    |
| ---------------------------- | ------------------------------------------ | ---------------------------------------- |
| `ADMIN_MAIL`                 | `admin@example.com`                        | Email used for certificate renewal notifications. |
| `DOMAIN`                     | `traefik.example.com`                      | Replace with your own domain.            |
| `AUTH_MIDDLEWARE_USER`       | `admin`                                    | Avoid using `admin` in production.       |
| `AUTH_MIDDLEWARE_HASHEDPASS` | `$$apr1$$jgzyu8ha$$xgWAUvjRb1AcfWxkGrJMN1` | Password must be hashed. This example's password is `§v9B7jv+QT2r0&CK`. |

**Note**: In the `docker-compose.yml` file, all `$` characters in the hashed password must be escaped with double `$$`.

---

### Commands

Run the following shell commands to deploy Traefik:

```shell
git clone https://github.com/iota7520/traefik-zeroconf
cd traefik-zeroconf
touch acme.json
sudo chmod 600 acme.json
docker network create traefik
docker compose up
