Here is the updated README file with the added `docker-compose` configuration and structured integration for better clarity:

---

# Poste.io - README.md

## Overview

This project sets up a self-hosted **Poste.io** mail server using Docker Compose.

---

## Features

- **Mail server**:
    - Fully featured with SMTP, IMAP, and a web interface.
    - Custom SSL certificates support.

---

## Prerequisites

- **Docker** and **Docker Compose** installed on your server.

---

## Docker Compose Configuration

Below is the `docker-compose.yml` file configuration for running Poste.io:

```yaml
version: '3.8'

services:
  mailserver:
    image: analogic/poste.io
    container_name: mailserver
    hostname: ${POSTEO_HOST_NAME}
    environment:
      - TZ=${TZ}
      - POSTEO_DOMAIN=${POSTEO_DOMAIN}
      - POSTEO_ADMIN_PASSWORD=${POSTEO_ADMIN_PASSWORD}
      - POSTEO_POSTMASTER_PASSWORD=${POSTEO_POSTMASTER_PASSWORD}
      - POSTEO_SMTP_PORT=25
      - POSTEO_IMAP_PORT=143
      - POSTEO_WEBSERVER_PORT=80
      - POSTEO_HTTP_PORT=80
    volumes:
      - ./data:/data
      - ./mail:/mail
      - ./certs:/etc/ssl/certs  # Add this for SSL certificates
    ports:
      - "25:25"      # SMTP
      - "143:143"    # IMAP
      - "88:80"      # Web interface (non-SSL)
      - "8443:443"   # Web interface (SSL, optional)
    restart: unless-stopped
```

---


## Environment Variables

Create a `.env` file in the project directory and populate it with the following variables:

```env
# Mail server settings
TZ=Africa/Lagos
POSTEO_DOMAIN=your-domain.com
POSTEO_HOST_NAME=mail.your-domain.com
POSTEO_ADMIN_PASSWORD=your-admin-password
POSTEO_POSTMASTER_PASSWORD=your-postmaster-password
POSTEO_HOST_NAME=your-hostname
```

---

## Setup

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Tesimune/poste.io.git
   cd poste.io
   ```

2. **Create `.env` File**:
   Add your environment variables as described above.

3. **Start the Service**:
   Run the following command to start the mail server:
   ```bash
   docker-compose up -d
   ```

4. **Verify the Setup**:
    - Access the mail server web interface at `http://<your-server-ip>:88` (non-SSL).

---

## Ports

- `25`: SMTP
- `143`: IMAP
- `88`: Web interface (non-SSL)
- `8443`: Web interface (SSL, optional)

Ensure these ports are open in your server firewall.

---

## Customizing SSL Certificates

To use custom SSL certificates:

1. Place your certificate files in the `./certs` directory.
2. The `docker-compose.yml` file already maps the `./certs` directory to `/etc/ssl/certs`.

---

## Logs

To monitor service logs:

- **Mailserver**:
  ```bash
  docker logs -f mailserver
  ```

---

## Troubleshooting

- **Port conflicts**: If port `88` or `8443` is already in use, update the `docker-compose.yml` file to use different ports.

---

## Backup Options

To enable backups for your Poste.io server, you can follow these guides:

1. [Poste.io with AWS S3 Backup](https://github.com/Tesimune/poste.io/tree/master/backup/S3)
2. [Poste.io with R2 Backup](https://github.com/Tesimune/poste.io/tree/master/backup/R2)

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

Let me know if you need further customization!