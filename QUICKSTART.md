# BTerminal Packages - Quick Start Guide

This guide will help you set up the complete BTerminal packages build and deployment system.

## Overview

The system consists of:
1. **GitHub Actions**: Automatically builds packages when code changes
2. **GitHub Releases**: Hosts bootstrap archives
3. **VPS (Contabo)**: Hosts the package repository at `packages.bxitools.com`

## Prerequisites

- GitHub account with access to this repository
- Contabo VPS or similar server
- Domain name `packages.bxitools.com` (or configure your own)
- Basic knowledge of Linux, SSH, and GitHub Actions

## Step-by-Step Setup

### Step 1: Set Up Your VPS

Follow the detailed instructions in [VPS_SETUP.md](VPS_SETUP.md) to:

1. Install required software (Aptly, Nginx, GPG)
2. Configure Aptly repositories
3. Set up Nginx with SSL
4. Create deployment scripts
5. Configure Aptly API

**Time required**: 1-2 hours

### Step 2: Configure GitHub Secrets

Add the required secrets to your GitHub repository as described in [SECRETS.md](SECRETS.md):

1. `VPS_HOST` - Your VPS hostname
2. `VPS_USER` - SSH username
3. `VPS_SSH_KEY` - Private SSH key
4. `VPS_DEPLOY_PATH` - Deployment directory path
5. `APTLY_API_AUTH` - Aptly API credentials (base64)
6. `GPG_PASSPHRASE` - GPG key passphrase (if set)

**Time required**: 15-30 minutes

### Step 3: Test the Build System

1. Make a small change to a package (e.g., update version in `build.sh`)
2. Commit and push to the `master` branch
3. GitHub Actions will automatically:
   - Build the package for all architectures
   - Upload to packages.bxitools.com (if configured)
4. Check the Actions tab for build status

**Time required**: 10 minutes + build time (varies)

### Step 4: Test Bootstrap Generation

1. Manually trigger the **Bootstrap Archives** workflow:
   - Go to Actions → Bootstrap Archives → Run workflow
2. Wait for completion (takes 30-60 minutes)
3. Check Releases for the new bootstrap archives

**Time required**: 30-60 minutes

### Step 5: Configure BTerminal App

Update your BTerminal app to use the new package repository:

```bash
# In BTerminal
echo "deb https://packages.bxitools.com/apt/termux-main stable main" > $PREFIX/etc/apt/sources.list

# Import GPG key
wget -O - https://packages.bxitools.com/PUBLIC.KEY | apt-key add -

# Update
apt update && apt upgrade
```

## Package Naming Changes

All packages now use the `com.bterminal` namespace instead of `com.termux`:

| Original | BTerminal |
|----------|-----------|
| `com.termux` | `com.bterminal` |
| `com.termux.api` | `com.bterminal.api` |
| `/data/data/com.termux` | `/data/data/com.bterminal` |

## Repository Structure

```
https://packages.bxitools.com/
├── apt/
│   ├── termux-main/     # Main packages
│   │   └── dists/stable/main/binary-{arch}/
│   ├── termux-root/     # Root-only packages
│   │   └── dists/root/stable/binary-{arch}/
│   └── termux-x11/      # X11 packages
│       └── dists/x11/main/binary-{arch}/
└── PUBLIC.KEY           # GPG public key
```

## Workflows

### 1. Packages Workflow (`.github/workflows/packages.yml`)
- **Triggers**: Push to master/dev branches, pull requests
- **Purpose**: Build packages when code changes
- **Output**: .deb files uploaded to VPS

### 2. Bootstrap Archives Workflow (`.github/workflows/bootstrap_archives.yml`)
- **Triggers**: Weekly schedule (Sunday), manual trigger
- **Purpose**: Generate bootstrap archives
- **Output**: Bootstrap .zip files in GitHub Releases

### 3. Deploy to VPS Workflow (`.github/workflows/deploy-to-vps.yml`)
- **Triggers**: After successful Packages workflow
- **Purpose**: Upload packages to VPS
- **Output**: Updated repository at packages.bxitools.com

## Monitoring

### Check Build Status
- Go to the **Actions** tab in GitHub
- View workflow runs and logs

### Check VPS Status
```bash
# SSH to VPS
ssh user@packages.bxitools.com

# Check Aptly service
sudo systemctl status aptly-api

# Check Nginx
sudo systemctl status nginx

# View deployment logs
tail -f /var/packages-deploy/update.log
```

### Check Repository
```bash
# List packages in repository
curl https://packages.bxitools.com/apt/termux-main/dists/stable/main/binary-aarch64/Packages | grep "Package:"
```

## Troubleshooting

### Build fails in GitHub Actions
1. Check the Actions logs for errors
2. Verify package `build.sh` syntax
3. Test build locally with Docker

### Packages not appearing in repository
1. Check VPS deployment logs: `/var/packages-deploy/update.log`
2. Verify Aptly API is running: `sudo systemctl status aptly-api`
3. Check GitHub secrets are correct

### SSL certificate issues
```bash
# Renew certificate
sudo certbot renew

# Check certificate status
sudo certbot certificates
```

### Repository not accessible
1. Check Nginx status: `sudo systemctl status nginx`
2. Verify DNS points to correct IP
3. Check firewall: `sudo ufw status`

## Common Tasks

### Add a new package
1. Create package directory: `packages/package-name/`
2. Create `build.sh` with package metadata
3. Commit and push to trigger build

### Update an existing package
1. Modify `TERMUX_PKG_VERSION` in `build.sh`
2. Update patches if needed
3. Commit and push

### Force rebuild
1. Go to Actions → Packages
2. Click "Run workflow"
3. Enter package names (space-separated)

### Generate new bootstrap
1. Go to Actions → Bootstrap Archives
2. Click "Run workflow"
3. Wait for completion
4. Download from Releases

## Maintenance

### Regular Tasks
- **Weekly**: Review build logs
- **Monthly**: Check disk space on VPS
- **Quarterly**: Update system packages on VPS
- **Yearly**: Rotate SSH keys and credentials

### Backup Strategy
```bash
# On VPS (automated with cron)
tar czf /backup/aptly-$(date +%Y%m%d).tar.gz /var/aptly
tar czf /backup/gpg-$(date +%Y%m%d).tar.gz ~/.gnupg
```

## Support and Contributing

- Report issues: [GitHub Issues](https://github.com/binyaminbinni/termux-packages/issues)
- Read contributing guide: [CONTRIBUTING.md](CONTRIBUTING.md)
- Based on: [Termux packages](https://github.com/termux/termux-packages)

## Next Steps

1. ✅ Complete VPS setup
2. ✅ Configure GitHub secrets
3. ✅ Test package build
4. ✅ Generate bootstrap archives
5. ✅ Update BTerminal app
6. 📝 Document custom packages
7. 🚀 Start building!

## Additional Resources

- [VPS Setup Guide](VPS_SETUP.md) - Detailed VPS configuration
- [Secrets Configuration](SECRETS.md) - GitHub secrets setup
- [Contributing Guide](CONTRIBUTING.md) - How to contribute
- [Termux Wiki](https://github.com/termux/termux-packages/wiki) - Package building docs
