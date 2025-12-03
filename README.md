# Mailu Deployment Guide

This repository contains the configuration to deploy Mailu, a full-featured mail server, as a Docker-based service.

## Prerequisites

- Docker
- Docker Compose

## Quick Start (Local)

1.  **Review Configuration**:
    The `mailu.env` file is pre-configured for local testing with `localhost`.
    
    > **Note**: For production, change `DOMAIN`, `HOSTNAMES`, and `TLS_FLAVOR`.
    > **Important**: Ensure the following variables are set in `mailu.env`:
    > - `WEB_WEBMAIL=/webmail` (or `/`)
    > - `ADMIN=true`
    > - `WEB_ADMIN=/admin`
    > - `WEBROOT_REDIRECT=/webmail` (optional, to redirect root to webmail)

2.  **Generate Secret Key**:
    You **must** generate a secure secret key for the `SECRET_KEY` variable in `mailu.env`.
    Run the following command in your terminal:
    ```bash
    openssl rand -hex 16
    ```
    Copy the output and replace `change_me_to_a_long_random_string` in `mailu.env`.

3.  **Generate Self-Signed Certificates** (Localhost only):
    For local testing with `TLS_FLAVOR=cert`, you must generate self-signed certificates.
    Run:
    ```bash
    openssl req -x509 -newkey rsa:4096 -keyout mailu/certs/key.pem -out mailu/certs/cert.pem -days 365 -nodes -subj "/CN=localhost"
    ```

4.  **Start the Service**:
    Run the following command to start all services:
    ```bash
    docker compose up -d
    ```

4.  **Access the Admin Interface**:
    Once the containers are running, access the admin interface at:
    [https://localhost/admin](https://localhost/admin)

    **Default Credentials**:
    - **Email**: `admin@localhost`
    - **Password**: `password`

    > **Warning**: Change the default password immediately after logging in!

## Directory Structure

- `docker-compose.yml`: Defines the Mailu services (front, admin, imap, smtp, etc.).
- `mailu.env`: Environment variables for configuration.
- `mailu/`: Directory where persistent data (mail, certs, db) will be stored.

## Architecture & Services

Mailu is composed of several Docker containers, each performing a specific role:

-   **front**: The main Nginx reverse proxy and authentication server. It handles all incoming connections and routes them to the appropriate backend services.
-   **admin**: The administration interface for managing domains, users, and aliases. It also handles DKIM key generation.
-   **imap**: Dovecot IMAP server for mail storage and retrieval.
-   **smtp**: Postfix SMTP server for mail delivery and routing.
-   **antispam**: Rspamd for spam filtering and DKIM signing.
-   **antivirus**: ClamAV for scanning emails for viruses.
-   **webmail**: A web-based email client (RainLoop or SnappyMail) for users to access their mail.
-   **redis**: Key-value store used by various services for caching and session management.
-   **resolver**: Unbound DNS resolver to ensure proper DNS lookups for anti-spam checks.

## Network & Ports

The following ports are exposed by the `front` service to handle various email and web protocols.

| Port | Protocol | Service | Description |
| :--- | :--- | :--- | :--- |
| **80** | TCP | HTTP | Webmail and Admin interface (redirects to HTTPS usually) |
| **443** | TCP | HTTPS | Secure Webmail and Admin interface access |
| **25** | TCP | SMTP | Standard SMTP port for incoming mail from other servers |
| **465** | TCP | SMTPS | Secure SMTP for mail submission (SSL/TLS) |
| **587** | TCP | SMTP | Secure SMTP for mail submission (STARTTLS) |
| **110** | TCP | POP3 | Standard POP3 for mail retrieval |
| **995** | TCP | POP3S | Secure POP3 for mail retrieval (SSL/TLS) |
| **143** | TCP | IMAP | Standard IMAP for mail retrieval |
| **993** | TCP | IMAPS | Secure IMAP for mail retrieval (SSL/TLS) |

## Production Setup

For production deployment:

1.  Update `DOMAIN` and `HOSTNAMES` in `mailu.env` to your real domain.
2.  Set `TLS_FLAVOR=letsencrypt` for automatic SSL certificates.
3.  Ensure your DNS records (A, MX, SPF, DKIM, DMARC) are correctly configured.
4.  Update `INITIAL_ADMIN_DOMAIN` to match your production domain.

## Troubleshooting

- **Check Logs**: `docker compose logs -f`
- **Check Status**: `docker compose ps`
