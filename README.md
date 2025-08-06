# UniFi OS Server SSL Import Script

A script to automatically import and update SSL certificates for your UniFi OS server (`uosserver`) from multiple certificate providers. The script supports both **Let's Encrypt via Certbot** and **acme.sh**, with DNS challenge support for all out-of-the-box DNS providers. The script stops the UniFi controller, replaces its TLS key and certificate files with the latest certs, sets correct permissions, and then restarts the controller — ensuring your UniFi server always has a valid SSL certificate.

---

## Features

- **Multiple Certificate Providers**: Support for both Certbot and acme.sh
- **DNS Challenge Support**: Works with Cloudflare and all supported acme.sh DNS providers
- **Smart Updates**: Automatically detects if the certificate changed and updates only if needed
- **Command Line Options**: 
  - `--force` to reinstall certificate even if unchanged
  - `--verbose` for detailed logs and command output
  - `--provider=certbot|acme` to specify certificate provider
  - `--dns=cloudflare` to specify DNS provider (for acme.sh)
- **Safety Features**: Creates backups of existing key and cert files before updating
- **Comprehensive Logging**: Logs actions to `/var/log/unifi-ssl-import.log`
- **Graceful Operation**: Clean stop/start of the UniFi controller

---

## Prerequisites

Choose one of the following certificate providers:

### Option 1: Certbot with Let's Encrypt

#### Install Certbot and DNS plugins

Make sure you run with root privileges:

```bash
apt update
apt install letsencrypt -y
# For Cloudflare
apt install python3-certbot-dns-cloudflare -y
```

#### Create a Cloudflare credentials file

```bash
mkdir -p /root/.secrets/
touch /root/.secrets/cloudflare.ini
nano /root/.secrets/cloudflare.ini
```

Add the following content (replace with your actual token):
**Make sure the Cloudflare token includes DNS Rights**

```
dns_cloudflare_api_token = your_token_here
```

Set proper permissions:

```bash
chmod 0700 /root/.secrets/
chmod 0400 /root/.secrets/cloudflare.ini
```

#### Request a certificate with Certbot

Replace `your.domain.com` with your actual domain:

```bash
certbot certonly --key-type rsa --rsa-key-size 4096 --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d your.domain.com --preferred-challenges dns-01
```

### Option 2: acme.sh with DNS Challenge

#### Install acme.sh

```bash
curl https://get.acme.sh | sh -s email=my@example.com
source ~/.bashrc
```

#### Configure DNS Provider

##### Example 1: Cloudflare:

```bash
export CF_Token="your_cloudflare_api_token"
export CF_Account_ID="your_account_id"
```

##### Example 2: Hetzner DNS:

```bash
export HETZNER_Token="your_hetzner_dns_api_token"
```

#### Request a certificate with acme.sh

Replace `your.domain.com` with your actual domain:

##### Example 1: Cloudflare:
```bash
acme.sh --issue --dns dns_cf -d your.domain.com --keylength 4096
```

##### Example 2: Hetzner DNS:
```bash
acme.sh --issue --dns dns_hetzner -d your.domain.com --keylength 4096
```

**Important**: UniFi OS Server only supports RSA certificates, so always use `--keylength 2048` (or higher RSA key lengths) with acme.sh. ECC certificates are not supported.

---

## Install the Import Script

Download the script:

```bash
wget https://raw.githubusercontent.com/MiranoVerhoef/UniFi-OS-Server-SSL-Import/refs/heads/main/unifi-osserver-ssl-import -O /usr/local/bin/unifi-osserver-ssl-import.sh
```

Make it executable:

```bash
chmod +x /usr/local/bin/unifi-osserver-ssl-import.sh
```

Set your UniFi domain name in the script:

```bash
nano -w /usr/local/bin/unifi-osserver-ssl-import.sh
```

Look for and modify the following configuration variables:

```bash
# Domain Name:
UNIFI_HOSTNAME="unifi.example.com"

# Certificate Provider: "certbot" or "acme"
CERT_PROVIDER="certbot"

# DNS Provider (for acme.sh): "cloudflare", "hetzner", etc.
DNS_PROVIDER="cloudflare"
```

---

## Running the Script

### Basic Usage

Run the script manually (uses configuration from script file):

```bash
/usr/local/bin/unifi-osserver-ssl-import.sh
```

### Command Line Options

You can override the configuration using command line arguments:

```bash
# Use certbot with Cloudflare (default)
/usr/local/bin/unifi-osserver-ssl-import.sh --provider=certbot

# Use acme.sh with Hetzner DNS
/usr/local/bin/unifi-osserver-ssl-import.sh --provider=acme --dns=hetzner

# Use acme.sh with Cloudflare DNS
/usr/local/bin/unifi-osserver-ssl-import.sh --provider=acme --dns=cloudflare

# Force certificate reinstallation
/usr/local/bin/unifi-osserver-ssl-import.sh --force

# Verbose output for troubleshooting
/usr/local/bin/unifi-osserver-ssl-import.sh --verbose

# Combine multiple options
/usr/local/bin/unifi-osserver-ssl-import.sh --provider=acme --dns=hetzner --verbose --force
```

### Available Options:
- `--provider=certbot|acme` – Specify certificate provider
- `--dns=cloudflare|hetzner` – Specify DNS provider (for acme.sh only)
- `--verbose` – Show detailed output of what the script is doing
- `--force` – Force reimport of certificate even if it hasn't changed

---


## To check logging:
```
tail -f /home/uosserver/.local/share/containers/storage/volumes/uosserver_data/_data/unifi-core/logs/http.log
```

## Example Crontab for Automation

Open crontab:

```bash
nano -w /etc/crontab
```

### For Certbot users:

Add the following lines to renew your certificate and update the UniFi server automatically twice a day:

```cron
0 */12 * * * root letsencrypt renew >> /home/renew_log.txt 2>&1
5 */12 * * * root /usr/local/bin/unifi-osserver-ssl-import.sh --provider=certbot >> /home/import_log.txt 2>&1
```

### For acme.sh users:

acme.sh automatically installs its own cron job for renewal. You only need to add the import script:

```cron
# Check for certificate updates twice a day
5 */12 * * * root /usr/local/bin/unifi-osserver-ssl-import.sh --provider=acme --dns=hetzner >> /home/import_log.txt 2>&1
```

### Provider-agnostic approach:

You can also set the provider in the script configuration and use:

```cron
5 */12 * * * root /usr/local/bin/unifi-osserver-ssl-import.sh >> /home/import_log.txt 2>&1
```

---

## License

[MIT](LICENSE)

---

## Author

[Mirano Verhoef](https://github.com/MiranoVerhoef)

## Contributor

[Martin Seener](https://github.com/martinseener)
