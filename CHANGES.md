# BTerminal Package Repository - Change Summary

This document summarizes all changes made to rebrand Termux packages to BTerminal and set up automated builds and deployments.

## Major Changes

### 1. Package Name Rebranding

**From**: `com.termux` → **To**: `com.bterminal`

#### Files Modified:
- `scripts/properties.sh`
  - `TERMUX_APP__PACKAGE_NAME` = `"com.bterminal"`
  - `TERMUX_APP__NAMESPACE` = `"com.bterminal"`
  - `TERMUX_API_APP__PACKAGE_NAME` = `"com.bterminal.api"`
  - `TERMUX_API_APP__NAMESPACE` = `"com.bterminal.api"`
  - Updated all default value comments from `/data/data/com.termux` to `/data/data/com.bterminal`

- `scripts/setup-ubuntu.sh`
  - Updated symlink path to use `com.bterminal`

- `CONTRIBUTING.md`
  - Updated example paths to use `com.bterminal`

### 2. Repository URL Changes

**From**: `packages-cf.termux.dev` → **To**: `packages.bxitools.com`

#### Files Modified:
- `repo.json`
  - Updated all repository URLs to `https://packages.bxitools.com/apt/*`
  
- `scripts/generate-bootstraps.sh`
  - Updated default APT repository URL to `https://packages.bxitools.com/apt/termux-main`

### 3. GitHub Actions Workflows

#### Files Modified:

**`.github/workflows/packages.yml`**
- Changed repository check from `termux/termux-packages` to `binyaminbinni/termux-packages`
- Updated upload destination from `packages.termux.dev` to `packages.bxitools.com`
- Modified Aptly API URL to `https://packages.bxitools.com/aptly-api`

**`.github/workflows/bootstrap_archives.yml`**
- Changed repository check from `termux/termux-packages` to `binyaminbinni/termux-packages`
- Keeps automatic weekly schedule (Sunday at midnight)
- Publishes bootstrap archives to GitHub Releases

### 4. New Files Added

#### Workflows:
- **`.github/workflows/deploy-to-vps.yml`**
  - New workflow for automatic deployment to VPS
  - Triggers after successful package builds
  - Uses SSH and rsync to upload packages
  - Requires VPS secrets configuration

#### Documentation:
- **`VPS_SETUP.md`**
  - Complete guide for setting up Contabo VPS
  - Instructions for Aptly, Nginx, SSL, GPG configuration
  - Deployment scripts and maintenance guides

- **`SECRETS.md`**
  - List of required GitHub secrets
  - Instructions for generating and configuring secrets
  - Security best practices

- **`QUICKSTART.md`**
  - Step-by-step setup guide
  - Workflow descriptions
  - Common tasks and troubleshooting

- **`CHANGES.md`** (this file)
  - Summary of all changes made

#### Updated:
- **`README.md`**
  - Updated branding to BTerminal
  - Changed badge URLs to point to new repository
  - Added repository setup instructions
  - Removed sponsor sections
  - Added credits to original Termux project

## Package Path Changes

| Component | Old Path | New Path |
|-----------|----------|----------|
| App Package | `com.termux` | `com.bterminal` |
| API Package | `com.termux.api` | `com.bterminal.api` |
| Data Directory | `/data/data/com.termux` | `/data/data/com.bterminal` |
| Namespace | `com.termux` | `com.bterminal` |

## Repository Structure

```
https://packages.bxitools.com/
├── apt/
│   ├── termux-main/
│   │   └── dists/stable/main/binary-{aarch64,arm,i686,x86_64,all}/
│   ├── termux-root/
│   │   └── dists/root/stable/binary-{aarch64,arm,i686,x86_64,all}/
│   └── termux-x11/
│       └── dists/x11/main/binary-{aarch64,arm,i686,x86_64,all}/
├── aptly-api/        # API endpoint for package uploads
└── PUBLIC.KEY        # GPG public key
```

## Automation Features

### Automatic Package Building
- Triggered on push to master/dev branches
- Builds for all architectures (aarch64, arm, i686, x86_64)
- Lints packages before building
- Uploads built packages to VPS

### Automatic Bootstrap Generation
- Scheduled weekly (Sunday at midnight UTC)
- Can be triggered manually
- Generates for all architectures
- Publishes to GitHub Releases with checksums

### Automatic Deployment
- Deploys after successful package builds
- Uses SSH for secure transfer
- Updates Aptly repository automatically
- Maintains deployment logs

## Required GitHub Secrets

For the automated system to work, the following secrets must be configured:

1. `VPS_HOST` - VPS hostname (packages.bxitools.com)
2. `VPS_USER` - SSH username
3. `VPS_SSH_KEY` - Private SSH key
4. `VPS_DEPLOY_PATH` - Deployment directory path
5. `APTLY_API_AUTH` - Aptly API credentials (base64)
6. `GPG_PASSPHRASE` - GPG key passphrase (optional)

See [SECRETS.md](SECRETS.md) for details.

## Testing Checklist

Before deploying to production:

- [ ] Verify VPS is set up correctly (see VPS_SETUP.md)
- [ ] Configure all GitHub secrets (see SECRETS.md)
- [ ] Test package build workflow
- [ ] Test bootstrap generation workflow
- [ ] Test VPS deployment workflow
- [ ] Verify packages are accessible at packages.bxitools.com
- [ ] Test package installation in BTerminal app
- [ ] Verify GPG signature validation
- [ ] Check SSL certificate is valid
- [ ] Set up monitoring and backups

## Compatibility Notes

### Backward Compatibility
- Packages built with the new system use `com.bterminal` paths
- Not compatible with original Termux app (`com.termux`)
- BTerminal app must be configured to use new package names

### Breaking Changes
- Package name change requires BTerminal app modification
- Cannot mix packages from termux.dev and bxitools.com
- Users must reconfigure their sources.list

## Migration Path

For users migrating from Termux:

1. **Backup data**: `tar czf ~/backup.tar.gz $PREFIX`
2. **Install BTerminal app** with `com.bterminal` package name
3. **Configure repository**:
   ```bash
   echo "deb https://packages.bxitools.com/apt/termux-main stable main" > $PREFIX/etc/apt/sources.list
   wget -O - https://packages.bxitools.com/PUBLIC.KEY | apt-key add -
   apt update && apt upgrade
   ```
4. **Restore data if needed**

## Future Enhancements

Potential improvements:
- [ ] Add package build caching
- [ ] Implement mirror support
- [ ] Add package signing verification in CI
- [ ] Create Docker image for local builds
- [ ] Add metrics and monitoring dashboard
- [ ] Implement staged rollouts
- [ ] Add automated testing for packages

## References

- Original Termux: https://github.com/termux/termux-packages
- Termux Wiki: https://github.com/termux/termux-packages/wiki
- Aptly Documentation: https://www.aptly.info/doc/
- GitHub Actions: https://docs.github.com/en/actions

## Credits

This fork is based on the excellent work by the Termux team. All credit for the original build system and packages goes to them.

## Support

- Issues: https://github.com/binyaminbinni/termux-packages/issues
- Documentation: See QUICKSTART.md, VPS_SETUP.md, SECRETS.md
