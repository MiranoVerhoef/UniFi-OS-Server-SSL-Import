# UniFi OS Server SSL Import Script

A Bash script to automatically import and update Let's Encrypt SSL certificates for your UniFi OS server (`uosserver`). The script stops the UniFi controller, replaces its TLS key and certificate files with the latest certs from Let's Encrypt, sets correct permissions, and then restarts the controller — ensuring your UniFi server always has a valid SSL certificate.

---

## Features

- Automatically detects if the certificate changed and updates only if needed
- Supports `--force` to reinstall certificate even if unchanged
- Supports `--verbose` for detailed logs and command output
- Creates backups of existing key and cert files before updating
- Logs actions to `/var/log/unifi-ssl-import.log`
- Graceful and clean stop/start of the UniFi controller

---

## Prerequisites

### Install Let's Encrypt and Cloudflare DNS plugin

```bash
apt update
apt install letsencrypt -y
apt install python3-certbot-dns-cloudflare -y
```

### Create a Cloudflare credentials file

```bash
mkdir -p /root/.secrets/
touch /root/.secrets/cloudflare.ini
nano /root/.secrets/cloudflare.ini
```

Add the following content (replace with your actual token):

```
dns_cloudflare_api_token = your_token_here
```

Set proper permissions:

```bash
chmod 0700 /root/.secrets/
chmod 0400 /root/.secrets/cloudflare.ini
```

### Request a certificate

Replace `your.domain.com` with your actual domain:

```bash
certbot certonly --key-type rsa --rsa-key-size 4096 --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d your.domain.com --preferred-challenges dns-01
```

---

## Install the Import Script

Download the script:

```bash
wget https://raw.githubusercontent.com/MiranoVerhoef/UniFi-SSL-Import/refs/heads/main/unifi-osserver-ssl-import -O /usr/local/bin/unifi-osserver-ssl-import.sh
```

Make it executable:

```bash
chmod +x /usr/local/bin/unifi-osserver-ssl-import.sh
```

Set your UniFi domain name in the script:

```bash
nano -w /usr/local/bin/unifi-osserver-ssl-import.sh
```

Look for and modify the following line:

```bash
UNIFI_HOSTNAME=unifi.example.com
```

---

## Running the Script

Run the script manually:

```bash
/usr/local/bin/unifi-osserver-ssl-import.sh
```

Optional flags:
- `--verbose` – Show detailed output of what the script is doing
- `--force` – Force reimport of certificate even if it hasn’t changed

---

## Example Crontab for Automation

Open crontab:

```bash
nano -w /etc/crontab
```

Add the following lines to renew your certificate and update the UniFi server automatically twice a day:

```cron
0 */12 * * * root letsencrypt renew >> /home/renew_log.txt 2>&1
5 */12 * * * root /usr/local/bin/unifi-osserver-ssl-import.sh >> /home/import_log.txt 2>&1
```

---

## License

MIT

---

## Author

[Mirano Verhoef](https://github.com/MiranoVerhoef)
