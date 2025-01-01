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

## Environment Variables

Create a `.env` file in the project directory and populate it with the following variables:

```env
# Mail server settings
TZ=Africa/Lagos
POSTEO_DOMAIN=your-domain.com
POSTEO_ADMIN_PASSWORD=your-admin-password
POSTEO_POSTMASTER_PASSWORD=your-postmaster-password
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
2. Map the directory in the `volumes` section of the `docker-compose.yml` file:
   ```yaml
   - ./certs:/etc/ssl/certs
   ```

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