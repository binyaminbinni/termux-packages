# VPS Setup Guide for BTerminal Packages

This document describes how to set up your Contabo VPS at `packages.bxitools.com` to host the BTerminal packages repository.

## Prerequisites

- A Contabo VPS with sufficient storage (recommended: 100GB+ for packages)
- Domain name `packages.bxitools.com` pointing to your VPS
- SSH access to the VPS
- Root or sudo access

## 1. VPS Server Setup

### Install Required Software

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Aptly (APT repository management tool)
wget -qO - https://www.aptly.info/pubkey.txt | sudo apt-key add -
echo "deb http://repo.aptly.info/ squeeze main" | sudo tee /etc/apt/sources.list.d/aptly.list
sudo apt update
sudo apt install -y aptly nginx gpg

# Install additional tools
sudo apt install -y rsync openssh-server curl wget
```

### Configure Aptly

```bash
# Create Aptly configuration directory
mkdir -p ~/.aptly

# Create Aptly configuration file
cat > ~/.aptly.conf <<EOF
{
  "rootDir": "/var/aptly",
  "downloadConcurrency": 4,
  "downloadSpeedLimit": 0,
  "architectures": ["aarch64", "arm", "i686", "x86_64", "all"],
  "dependencyFollowSuggests": false,
  "dependencyFollowRecommends": false,
  "dependencyFollowAllVariants": false,
  "dependencyFollowSource": false,
  "dependencyVerboseResolve": false,
  "gpgDisableSign": false,
  "gpgDisableVerify": false,
  "gpgProvider": "gpg",
  "downloadSourcePackages": false,
  "skipLegacyPool": true,
  "ppaDistributorID": "bterminal",
  "ppaCodename": "",
  "FileSystemPublishEndpoints": {
    "main": {
      "rootDir": "/var/www/packages.bxitools.com/apt",
      "linkMethod": "symlink"
    }
  }
}
EOF

# Create directories
sudo mkdir -p /var/aptly
sudo mkdir -p /var/www/packages.bxitools.com/apt
sudo chown -R $USER:$USER /var/aptly /var/www/packages.bxitools.com
```

### Generate GPG Key for Package Signing

```bash
# Generate GPG key (use your information)
gpg --full-generate-key

# Follow prompts:
# - Key type: RSA and RSA
# - Key size: 4096
# - Expiration: 0 (does not expire)
# - Real name: BTerminal Packages
# - Email: packages@bxitools.com
# - Comment: BTerminal Package Repository

# Export public key
gpg --armor --export packages@bxitools.com > /var/www/packages.bxitools.com/PUBLIC.KEY
```

### Create Aptly Repositories

```bash
# Create repositories for each component
aptly repo create -distribution=stable -component=main termux-main
aptly repo create -distribution=root -component=stable termux-root
aptly repo create -distribution=x11 -component=main termux-x11

# Create empty snapshots (initial setup)
aptly snapshot create termux-main-initial from repo termux-main
aptly snapshot create termux-root-initial from repo termux-root
aptly snapshot create termux-x11-initial from repo termux-x11

# Publish repositories
aptly publish snapshot -distribution=stable termux-main-initial filesystem:main:termux-main
aptly publish snapshot -distribution=root termux-root-initial filesystem:main:termux-root
aptly publish snapshot -distribution=x11 termux-x11-initial filesystem:main:termux-x11
```

## 2. Configure Nginx

```bash
# Create Nginx configuration
sudo tee /etc/nginx/sites-available/packages.bxitools.com <<EOF
server {
    listen 80;
    listen [::]:80;
    server_name packages.bxitools.com;
    
    root /var/www/packages.bxitools.com;
    index index.html;
    
    location / {
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
    }
    
    location ~ \.deb$ {
        add_header Content-Type application/x-debian-package;
    }
    
    location /aptly-api {
        proxy_pass http://localhost:8080;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
EOF

# Enable site
sudo ln -sf /etc/nginx/sites-available/packages.bxitools.com /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# Create basic auth for aptly API
sudo apt install -y apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd aptly

# Test and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

## 3. Setup SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot --nginx -d packages.bxitools.com

# Auto-renewal is configured automatically
```

## 4. Setup Aptly API Service

```bash
# Create systemd service for Aptly API
sudo tee /etc/systemd/system/aptly-api.service <<EOF
[Unit]
Description=Aptly API Service
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=/home/$USER
ExecStart=/usr/bin/aptly api serve -listen=:8080 -no-lock
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable aptly-api
sudo systemctl start aptly-api
```

## 5. Create Package Upload Script

```bash
# Create deployment directory
mkdir -p /var/packages-deploy

# Create update script
cat > /var/packages-deploy/update-repo.sh <<'EOF'
#!/bin/bash
set -e

INCOMING_DIR="/var/packages-deploy/incoming"
LOG_FILE="/var/packages-deploy/update.log"

echo "[$(date)] Starting repository update..." | tee -a "$LOG_FILE"

# Function to add packages to aptly repo
add_packages_to_repo() {
    local repo_name=$1
    local deb_pattern=$2
    
    if ls "$INCOMING_DIR"/$deb_pattern 1> /dev/null 2>&1; then
        echo "Adding packages to $repo_name..." | tee -a "$LOG_FILE"
        aptly repo add "$repo_name" "$INCOMING_DIR"/$deb_pattern 2>&1 | tee -a "$LOG_FILE"
    fi
}

# Add packages to respective repositories
add_packages_to_repo "termux-main" "*.deb"
add_packages_to_repo "termux-root" "*.deb"
add_packages_to_repo "termux-x11" "*.deb"

# Update published repositories
echo "Updating published repositories..." | tee -a "$LOG_FILE"

for repo in termux-main termux-root termux-x11; do
    SNAPSHOT_NAME="${repo}-$(date +%Y%m%d-%H%M%S)"
    aptly snapshot create "$SNAPSHOT_NAME" from repo "$repo" 2>&1 | tee -a "$LOG_FILE"
    
    case $repo in
        termux-main)
            aptly publish switch stable filesystem:main:termux-main "$SNAPSHOT_NAME" 2>&1 | tee -a "$LOG_FILE"
            ;;
        termux-root)
            aptly publish switch root filesystem:main:termux-root "$SNAPSHOT_NAME" 2>&1 | tee -a "$LOG_FILE"
            ;;
        termux-x11)
            aptly publish switch x11 filesystem:main:termux-x11 "$SNAPSHOT_NAME" 2>&1 | tee -a "$LOG_FILE"
            ;;
    esac
done

# Clean up incoming directory
rm -f "$INCOMING_DIR"/*.deb

echo "[$(date)] Repository update completed!" | tee -a "$LOG_FILE"
EOF

chmod +x /var/packages-deploy/update-repo.sh
```

## 6. GitHub Actions Secrets Configuration

Add the following secrets to your GitHub repository settings:

1. **VPS_HOST**: `packages.bxitools.com`
2. **VPS_USER**: Your SSH username (e.g., `root` or your user)
3. **VPS_SSH_KEY**: Your private SSH key for VPS access
4. **VPS_DEPLOY_PATH**: `/var/packages-deploy`
5. **APTLY_API_AUTH**: Base64 encoded `username:password` for Aptly API
6. **GPG_PASSPHRASE**: Passphrase for GPG key (if you set one)

To generate APTLY_API_AUTH:
```bash
echo -n "aptly:YOUR_PASSWORD" | base64
```

## 7. Testing the Setup

### Test Package Upload Manually

```bash
# On your local machine, upload a test package
scp test-package.deb user@packages.bxitools.com:/var/packages-deploy/incoming/
ssh user@packages.bxitools.com "/var/packages-deploy/update-repo.sh"
```

### Test Repository Access

```bash
# Test if repository is accessible
curl https://packages.bxitools.com/apt/termux-main/dists/stable/Release
```

## 8. Client Configuration (BTerminal App)

Users will need to configure their BTerminal app to use the new repository:

```bash
# In BTerminal, edit sources.list
nano $PREFIX/etc/apt/sources.list

# Add these lines:
deb https://packages.bxitools.com/apt/termux-main stable main
# deb https://packages.bxitools.com/apt/termux-root root stable
# deb https://packages.bxitools.com/apt/termux-x11 x11 main

# Import GPG key
wget -O - https://packages.bxitools.com/PUBLIC.KEY | apt-key add -

# Update package list
apt update
```

## 9. Monitoring and Maintenance

### Check Aptly Status
```bash
aptly repo list
aptly snapshot list
aptly publish list
```

### Check Logs
```bash
tail -f /var/packages-deploy/update.log
sudo journalctl -u aptly-api -f
sudo tail -f /var/log/nginx/access.log
```

### Backup Strategy
```bash
# Backup Aptly database and GPG keys
sudo tar czf /backup/aptly-backup-$(date +%Y%m%d).tar.gz /var/aptly ~/.gnupg

# Setup automated backups with cron
(crontab -l 2>/dev/null; echo "0 2 * * * tar czf /backup/aptly-backup-\$(date +\%Y\%m\%d).tar.gz /var/aptly ~/.gnupg") | crontab -
```

## Troubleshooting

### Issue: Packages not appearing in repository
- Check `/var/packages-deploy/update.log` for errors
- Verify packages are in correct format (`.deb`)
- Check Aptly repo: `aptly repo show termux-main`

### Issue: SSL certificate not working
- Renew certificate: `sudo certbot renew`
- Check Nginx configuration: `sudo nginx -t`

### Issue: Aptly API not accessible
- Check service status: `sudo systemctl status aptly-api`
- Check if port 8080 is listening: `netstat -tlnp | grep 8080`
- Verify auth credentials in GitHub secrets

## Additional Notes

- The repository will automatically update when packages are built by GitHub Actions
- Bootstrap archives will be available in GitHub Releases
- Ensure sufficient disk space (at least 2x the size of all packages)
- Regular backups are crucial - set them up before going into production
- Monitor server resources and adjust as needed
