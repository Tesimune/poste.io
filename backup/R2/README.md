# Poste.io with Cloudflare R2 Backup - README.md

## Overview

This project sets up a self-hosted **Poste.io** mail server using Docker Compose. It includes an automated backup service that archives mail server data and uploads it to **Cloudflare R2** daily.

---

## Features

- **Mail server**:
  - Fully featured with SMTP, IMAP, and a web interface.
  - Custom SSL certificates support.

- **Automated backup**:
  - Archives mail data and uploads it to Cloudflare R2.
  - Backup runs daily at 24-hour intervals.

---

## Prerequisites

- **Docker** and **Docker Compose** installed on your server.
- A **Cloudflare R2 bucket** for storing backups.
- R2 API credentials with write access to the bucket.

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
      - "8443:443"    # Web interface (SSL, optional)
    restart: unless-stopped

  backup:
    image: amazon/aws-cli:2.13.17
    container_name: mailserver_backup
    depends_on:
      - mailserver
    environment:
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AWS_DEFAULT_REGION: ${AWS_DEFAULT_REGION}
      R2_ACCOUNT_ID: ${R2_ACCOUNT_ID}
      R2_ACCESS_KEY_ID: ${R2_ACCESS_KEY_ID}
      R2_SECRET_ACCESS_KEY: ${R2_SECRET_ACCESS_KEY}
      R2_BUCKET: ${R2_BUCKET}
      R2_ENDPOINT: ${R2_ENDPOINT}
      BACKUP_PROVIDER: ${BACKUP_PROVIDER} # Set to 'aws' or 'r2'
    volumes:
      - ./data:/data
      - ./mail:/mail
    entrypoint: >
      /bin/sh -c "
      while true; do
        TIMESTAMP=$(date +%Y%m%d-%H%M%S);
        BACKUP_FILE=/tmp/posteio-backup-$TIMESTAMP.tar.gz;
        tar -czvf $BACKUP_FILE /data /mail;
        if [ "$BACKUP_PROVIDER" = "aws" ]; then
          aws s3 cp $BACKUP_FILE s3://$S3_BUCKET/$TIMESTAMP.tar.gz;
        elif [ "$BACKUP_PROVIDER" = "r2" ]; then
          aws s3 cp $BACKUP_FILE s3://$R2_BUCKET/$TIMESTAMP.tar.gz --endpoint-url=$R2_ENDPOINT;
        fi;
        rm -f $BACKUP_FILE;
        sleep 86400; # Run backup every 24 hours
      done"

```

---

## Environment Variables

Create a `.env` file in the project directory and populate it with the following variables:

```env
# Mail server settings
TZ=UTC
POSTEO_DOMAIN=your-domain.com
POSTEO_HOST_NAME=mail.your-domain.com
POSTEO_ADMIN_PASSWORD=your-admin-password
POSTEO_POSTMASTER_PASSWORD=your-postmaster-password

# Cloudflare R2 backup settings
R2_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key-id
R2_SECRET_ACCESS_KEY=your-secret-access-key
R2_BUCKET=your-bucket-name
R2_ENDPOINT=https://<your-r2-endpoint>
```
[Poste.io Time zone](https://github.com/Tesimune/poste.io/tree/main/timezone)
---

## Setup

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-repo/posteio-r2-backup.git
   cd posteio-r2-backup
   ```

2. **Create `.env` File**:
   Add your environment variables as described above.

3. **Start the Services**:
   Run the following command to start both the mail server and the backup service:
   ```bash
   docker-compose up -d
   ```

4. **Verify the Setup**:
   - Access the mail server web interface at `http://<your-server-ip>:88` (non-SSL).
   - Check the Cloudflare R2 bucket for backup files.

---

## Ports

- `25`: SMTP
- `143`: IMAP
- `88`: Web interface (non-SSL)
- `8443`: Web interface (SSL, optional)

Ensure these ports are open in your server firewall.

Update your domain’s DNS records for proper mail delivery:
- **MX Record**: Points to your server’s IP or hostname.
- **SPF, DKIM, DMARC**: For better email deliverability.
  https://poste.io/doc/configuring-dns
---

## Backup Process

- Archives the `/data` and `/mail` directories from the mail server.
- Creates a tarball with a timestamp in the format `posteio-backup-YYYYMMDD-HHMMSS.tar.gz`.
- Uploads the tarball to the specified Cloudflare R2 bucket.
- Runs daily at 24-hour intervals.

---

## Restoring from Backup

To restore data from a backup:

1. **Download the Backup**:
   ```bash
   aws s3 cp s3://your-r2-bucket-name/posteio-backup-YYYYMMDD-HHMMSS.tar.gz ./backup.tar.gz --endpoint-url=https://<your-r2-endpoint>
   ```

2. **Extract the Backup**:
   ```bash
   tar -xzvf backup.tar.gz -C ./data
   tar -xzvf backup.tar.gz -C ./mail
   ```

3. **Restart the Containers**:
   ```bash
   docker-compose up -d
   ```

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

- **Backup Service**:
  ```bash
  docker logs -f mailserver_backup
  ```

---

## Troubleshooting

- **Port conflicts**: If port `88` or `8443` is already in use, update the `docker-compose.yml` file to use different ports.
- **Backup not running**: Ensure your R2 credentials in the `.env` file are correct and have sufficient permissions.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---