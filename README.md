# Mailu Deployment Guide

This repository contains the configuration to deploy Mailu, a full-featured mail server, as a Docker-based service.

## Prerequisites

- Docker
- Docker Compose

## Quick Start

> **⚠️ Important**: Mailu does **NOT** work with `localhost` as a domain. You must use a real domain name (e.g., `example.com`, `mail.example.com`). The email validation in the login interface will reject `localhost` and other invalid domains.

1.  **Review Configuration**:
    The `mailu.env` file needs to be configured with your domain.
    
    > **Note**: You must set a real domain - `localhost` will not work for email addresses.
    > **Important**: Ensure the following variables are set in `mailu.env`:
    > - `DOMAIN` - Your actual domain (e.g., `example.com`)
    > - `HOSTNAMES` - Your mail server hostnames (e.g., `mail.example.com,example.com`)
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
    Copy the output and replace the `SECRET_KEY` value in `mailu.env`.

3.  **Choose TLS Configuration**:
    - **For Production**: Set `TLS_FLAVOR=letsencrypt` (automatic SSL certificates)
      - Requires valid DNS records pointing to your server
      - Ports 80 and 443 must be accessible from the internet
    - **For Testing with Self-Signed Certificates**: Set `TLS_FLAVOR=cert`
      - Generate certificates manually:
        ```bash
        openssl req -x509 -newkey rsa:4096 -keyout mailu/certs/key.pem -out mailu/certs/cert.pem -days 365 -nodes -subj "/CN=yourdomain.com"
        ```

4.  **Start the Service**:
    Run the following command to start all services:
    ```bash
    docker compose up -d
    ```

5.  **Access the Admin Interface**:
    Once the containers are running, access the admin interface at:
    `https://yourdomain.com/admin` or `https://mail.yourdomain.com/admin`

    **Default Credentials** (from `mailu.env`):
    - **Email**: `admin@yourdomain.com` (matches `INITIAL_ADMIN_ACCOUNT` and `INITIAL_ADMIN_DOMAIN`)
    - **Password**: The value set in `INITIAL_ADMIN_PW` (default: `password`)

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

### Required DNS Records

For your domain to work properly, configure these DNS records (replace `example.com` with your actual domain):

1. **A Record** (for mail server):
   ```
   mail.example.com  A  <your-server-ip>
   ```

2. **A Record** (for web access, if using root domain):
   ```
   example.com  A  <your-server-ip>
   ```

3. **MX Record** (for receiving email):
   ```
   example.com  MX  10  mail.example.com
   ```

4. **SPF Record** (TXT):
   ```
   example.com  TXT  "v=spf1 mx a:mail.example.com ~all"
   ```

5. **DKIM Record** (TXT):
   - After starting Mailu, get the DKIM key from the admin interface
   - Add as: `mail._domainkey.example.com  TXT  "<dkim-key>"`

6. **DMARC Record** (TXT):
   ```
   _dmarc.example.com  TXT  "v=DMARC1; p=quarantine; rua=mailto:admin@example.com"
   ```

### Firewall Requirements

Ensure these ports are open on your server:
- **80/tcp** - HTTP (for Let's Encrypt validation)
- **443/tcp** - HTTPS (webmail and admin)
- **25/tcp** - SMTP (incoming mail)
- **587/tcp** - SMTP submission (STARTTLS)
- **465/tcp** - SMTPS (SSL/TLS)
- **993/tcp** - IMAPS (secure IMAP)
- **995/tcp** - POP3S (secure POP3)

## Troubleshooting

### Login Issues

If you cannot log in to the admin interface:

1. **Domain Validation Error**:
   - **Important**: Mailu does NOT work with `localhost` as a domain
   - You must use a real domain name (e.g., `example.com`)
   - The login interface will show "Invalid email address" if you use `localhost` or invalid domains

2. **Clear Browser Data**:
   - Clear cookies and cache for your domain
   - Or use an incognito/private browsing window
   - Make sure you're accessing `https://yourdomain.com/admin` (not `/webmail`)

3. **Verify Credentials**:
   - **Email**: Use the full email address (e.g., `admin@yourdomain.com`, not just "admin")
   - **Password**: The value set in `INITIAL_ADMIN_PW` in `mailu.env`
   - Make sure there are no extra spaces when typing

4. **Reset Admin Password** (if needed):
   ```bash
   docker compose exec admin flask mailu admin admin yourdomain.com yourpassword --mode update
   ```
   Replace `yourdomain.com` with your actual domain and `yourpassword` with your desired password.

4. **Check Service Status**:
   ```bash
   docker compose ps
   ```
   All services should show as `(healthy)`

5. **Check Logs for Errors**:
   ```bash
   docker compose logs admin | grep -i error
   docker compose logs front | grep -i error
   ```

### Other Issues

- **Check Logs**: `docker compose logs -f`
- **Check Status**: `docker compose ps`
- **Restart Services**: `docker compose restart`
