# Squid Proxy Auto-Installer for Ubuntu 22.04

One-command automated installation script for Squid Proxy with basic authentication on Ubuntu 22.04 LTS.

## Features

✅ **Fully Automated** - No manual configuration required
✅ **Auto-Generated Credentials** - Random 16-character password
✅ **Basic Authentication** - Secure NCSA authentication with htpasswd
✅ **Firewall Configuration** - Automatic UFW setup
✅ **Connection Testing** - Validates proxy after installation
✅ **Credential Backup** - Saves credentials to `/root/squid-proxy-credentials.txt`

## Requirements

- **OS:** Ubuntu 22.04 LTS (may work on other versions)
- **Access:** Root or sudo privileges
- **Network:** Active internet connection

## Quick Start

### One-Line Installation

```bash
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/shahrukhfiaz/squidproxy/main/install-squid-proxy.sh)"
```

**OR** download and run manually:

```bash
wget https://raw.githubusercontent.com/shahrukhfiaz/squidproxy/main/install-squid-proxy.sh
chmod +x install-squid-proxy.sh
sudo ./install-squid-proxy.sh
```

## What Gets Installed

The script performs the following actions:

1. ✅ Checks Ubuntu version (warns if not 22.04)
2. 📦 Installs Squid and Apache2-utils
3. 🔑 Generates random proxy credentials
4. 📝 Creates Squid configuration with authentication
5. 🔄 Initializes Squid cache directories
6. 🚀 Starts and enables Squid service
7. 🔥 Configures UFW firewall (port 3128)
8. 🧪 Tests proxy connection
9. 💾 Saves credentials to file

## Default Configuration

| Setting | Value |
|---------|-------|
| **Port** | 3128 |
| **Protocol** | HTTP |
| **Username** | `squid_proxy` |
| **Password** | *Auto-generated (16 chars)* |
| **Authentication** | Basic NCSA |
| **Cache Size** | 100 MB |
| **Cache Memory** | 256 MB |

## Post-Installation

### View Credentials

```bash
sudo cat /root/squid-proxy-credentials.txt
```

**Example output:**
```
========================================
Squid Proxy Credentials
========================================
Server IP:    167.99.147.118
Proxy Port:   3128
Protocol:     HTTP
Username:     squid_proxy
Password:     xK9mN2pQ7wR4sT1v

Connection String:
http://squid_proxy:xK9mN2pQ7wR4sT1v@167.99.147.118:3128
```

### Test Proxy Connection

```bash
# Replace with your actual credentials from /root/squid-proxy-credentials.txt
curl -x http://squid_proxy:YOUR_PASSWORD@localhost:3128 http://httpbin.org/ip
```

**Expected response:**
```json
{
  "origin": "YOUR_PROXY_IP"
}
```

### Check Squid Status

```bash
sudo systemctl status squid
```

### View Access Logs

```bash
sudo tail -f /var/log/squid/access.log
```

### Restart Squid

```bash
sudo systemctl restart squid
```

## Using the Proxy

### cURL Example

```bash
curl -x http://squid_proxy:PASSWORD@PROXY_IP:3128 https://example.com
```

### Node.js Example

```javascript
const axios = require('axios');
const HttpsProxyAgent = require('https-proxy-agent');

const proxyUrl = 'http://squid_proxy:PASSWORD@PROXY_IP:3128';
const agent = new HttpsProxyAgent(proxyUrl);

axios.get('https://httpbin.org/ip', { httpsAgent: agent })
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

### Electron Example

```javascript
// In main process before app is ready
app.commandLine.appendSwitch('proxy-server', 'PROXY_IP:3128');

// Handle proxy authentication
app.on('login', (event, webContents, request, authInfo, callback) => {
  if (authInfo.isProxy) {
    event.preventDefault();
    callback('squid_proxy', 'PASSWORD');
  }
});
```

### Browser Configuration

**Manual Proxy Settings:**
- **Proxy Address:** `PROXY_IP`
- **Port:** `3128`
- **Username:** `squid_proxy`
- **Password:** *Your auto-generated password*

## Adding More Users

```bash
# Add additional user
sudo htpasswd -b /etc/squid/passwd new_username new_password

# Restart Squid to apply changes
sudo systemctl restart squid
```

## Configuration Files

| File | Purpose |
|------|---------|
| `/etc/squid/squid.conf` | Main configuration file |
| `/etc/squid/passwd` | Authentication credentials (hashed) |
| `/var/log/squid/access.log` | Access logs |
| `/var/log/squid/cache.log` | Cache and error logs |
| `/root/squid-proxy-credentials.txt` | Your credentials (plaintext) |

## Advanced Configuration

### Change Proxy Port

Edit `/etc/squid/squid.conf`:
```bash
sudo nano /etc/squid/squid.conf
```

Find and modify:
```conf
http_port 3128  # Change to desired port
```

Restart Squid:
```bash
sudo systemctl restart squid
```

Update firewall:
```bash
sudo ufw allow NEW_PORT/tcp
sudo ufw delete allow 3128/tcp
```

### Adjust Cache Size

Edit `/etc/squid/squid.conf`:
```conf
cache_dir ufs /var/spool/squid 1000 16 256  # Change 1000 to desired MB
cache_mem 512 MB  # Change to desired memory
```

Restart Squid:
```bash
sudo systemctl restart squid
```

### Test Configuration

```bash
sudo squid -k parse
```

## Troubleshooting

### Squid Won't Start

Check logs:
```bash
sudo journalctl -u squid -n 50
```

Verify configuration syntax:
```bash
sudo squid -k parse
```

### Authentication Fails

Verify password file:
```bash
sudo cat /etc/squid/passwd
```

Check permissions:
```bash
sudo chmod 640 /etc/squid/passwd
sudo chown root:proxy /etc/squid/passwd
```

### Port Already in Use

Check what's using port 3128:
```bash
sudo netstat -tuln | grep 3128
```

Kill existing process or change Squid port.

### Firewall Blocking Connections

Check UFW status:
```bash
sudo ufw status
```

Allow port 3128:
```bash
sudo ufw allow 3128/tcp
```

### Check Proxy Health

```bash
# From local machine
curl -x http://squid_proxy:PASSWORD@localhost:3128 http://httpbin.org/ip

# From remote machine
curl -x http://squid_proxy:PASSWORD@PROXY_IP:3128 http://httpbin.org/ip
```

## Uninstallation

```bash
# Stop Squid
sudo systemctl stop squid
sudo systemctl disable squid

# Remove Squid
sudo apt-get remove --purge squid squid-common apache2-utils -y
sudo apt-get autoremove -y

# Remove configuration and logs
sudo rm -rf /etc/squid
sudo rm -rf /var/log/squid
sudo rm -rf /var/spool/squid

# Remove credentials file
sudo rm -f /root/squid-proxy-credentials.txt

# Remove firewall rule
sudo ufw delete allow 3128/tcp
```

## Security Recommendations

🔒 **Secure Your Credentials:**
- Change the default username from `squid_proxy`
- Use strong passwords (auto-generated is secure)
- Restrict access to `/root/squid-proxy-credentials.txt`

🔒 **Firewall Configuration:**
- Only allow connections from trusted IPs
- Use UFW to restrict access:
  ```bash
  sudo ufw allow from TRUSTED_IP to any port 3128
  ```

🔒 **Regular Updates:**
- Keep Squid updated: `sudo apt-get update && sudo apt-get upgrade squid`

🔒 **Monitor Access Logs:**
- Review logs regularly: `sudo tail -f /var/log/squid/access.log`
- Set up log rotation

## Supported Protocols

The default configuration supports:

- ✅ HTTP
- ✅ HTTPS (via CONNECT method)
- ❌ SOCKS4/SOCKS5 (not configured by default)

## DNS Configuration

The script configures Squid to use these DNS servers:
- 8.8.8.8 (Google)
- 8.8.4.4 (Google)
- 1.1.1.1 (Cloudflare)

## Performance Tuning

For high-traffic scenarios, edit `/etc/squid/squid.conf`:

```conf
# Increase authentication workers
auth_param basic children 10  # Default: 5

# Increase cache size
cache_dir ufs /var/spool/squid 5000 16 256  # 5GB

# Increase memory cache
cache_mem 512 MB  # Default: 256 MB

# Adjust object sizes
maximum_object_size 10240 KB  # Default: 4096 KB
```

## License

This script is provided as-is without warranty. Feel free to modify and distribute.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Support

For issues or questions:
- Open an issue on GitHub
- Check Squid documentation: http://www.squid-cache.org/Doc/

---

**Created by:** [Shahrukh Fiaz](https://github.com/shahrukhfiaz)
**Repository:** https://github.com/shahrukhfiaz/squidproxy
**Last Updated:** 2026-02-13
