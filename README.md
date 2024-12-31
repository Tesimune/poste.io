# Poste.io with AWS S3 Backup - README.md

## Overview

This project sets up a self-hosted **Poste.io** mail server using Docker Compose. It also includes an automated backup service that archives mail server data and uploads it to an **AWS S3 bucket** daily.

---

## Features

- **Mail server**:
  - Fully featured with SMTP, IMAP, and a web interface.
  - Custom SSL certificates support.

- **Automated backup**:
  - Archives mail data and uploads it to an AWS S3 bucket.
  - Backup runs daily at 24-hour intervals.

---

## Prerequisites

- **Docker** and **Docker Compose** installed on your server.
- An **AWS S3 bucket** for storing backups.
- AWS CLI credentials with write access to the S3 bucket.

---

## Environment Variables

Create a `.env` file in the project directory and populate it with the following variables:

```env
# Mail server settings
TZ=Africa/Lagos
POSTEO_DOMAIN=your-domain.com
POSTEO_ADMIN_PASSWORD=your-admin-password
POSTEO_POSTMASTER_PASSWORD=your-postmaster-password

# AWS backup settings
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_DEFAULT_REGION=your-region
S3_BUCKET=your-bucket-name
```

---

## Setup

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-repo/posteio-aws-backup.git
   cd posteio-aws-backup
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
   - Check the AWS S3 bucket for backup files.

---

## Ports

- `25`: SMTP
- `143`: IMAP
- `88`: Web interface (non-SSL)
- `8443`: Web interface (SSL, optional)

Ensure these ports are open in your server firewall.

---

## Backup Process

- Archives the `/data` and `/mail` directories from the mail server.
- Creates a tarball with a timestamp in the format `posteio-backup-YYYYMMDD-HHMMSS.tar.gz`.
- Uploads the tarball to the specified AWS S3 bucket.
- Runs daily at 24-hour intervals.

---

## Restoring from Backup

To restore data from a backup:

1. **Download the Backup**:
   ```bash
   aws s3 cp s3://your-bucket-name/posteio-backup-YYYYMMDD-HHMMSS.tar.gz ./backup.tar.gz
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
- **Backup not running**: Ensure your AWS credentials in the `.env` file are correct and have sufficient permissions.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

Let me know if you need further customization!