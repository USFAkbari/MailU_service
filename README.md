# Mailu Deployment Guide

This repository contains the configuration to deploy Mailu, a full-featured mail server, as a Docker-based service.

## Prerequisites

- Docker
- Docker Compose

## Quick Start (Local)

1.  **Review Configuration**:
    The `mailu.env` file is pre-configured for local testing with `localhost`.
    
    > **Note**: For production, change `DOMAIN`, `HOSTNAMES`, and `TLS_FLAVOR`.

2.  **Generate Secret Key**:
    You **must** generate a secure secret key for the `SECRET_KEY` variable in `mailu.env`.
    Run the following command in your terminal:
    ```bash
    openssl rand -hex 16
    ```
    Copy the output and replace `change_me_to_a_long_random_string` in `mailu.env`.

3.  **Start the Service**:
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

## Production Setup

For production deployment:

1.  Update `DOMAIN` and `HOSTNAMES` in `mailu.env` to your real domain.
2.  Set `TLS_FLAVOR=letsencrypt` for automatic SSL certificates.
3.  Ensure your DNS records (A, MX, SPF, DKIM, DMARC) are correctly configured.
4.  Update `INITIAL_ADMIN_DOMAIN` to match your production domain.

## Troubleshooting

- **Check Logs**: `docker compose logs -f`
- **Check Status**: `docker compose ps`
