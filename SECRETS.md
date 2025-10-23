# GitHub Secrets Configuration

This document lists all the GitHub secrets that need to be configured for the BTerminal packages repository to work correctly.

## Required Secrets

Configure these secrets in your GitHub repository settings at:
`https://github.com/binyaminbinni/termux-packages/settings/secrets/actions`

### 1. VPS Deployment Secrets

These secrets are used by the `deploy-to-vps.yml` workflow to upload packages to your Contabo VPS.

#### `VPS_HOST`
- **Description**: The hostname or IP address of your VPS
- **Example**: `packages.bxitools.com` or `123.456.789.10`
- **Required**: Yes

#### `VPS_USER`
- **Description**: SSH username for VPS access
- **Example**: `root` or `deploy`
- **Required**: Yes

#### `VPS_SSH_KEY`
- **Description**: Private SSH key for passwordless authentication to VPS
- **How to generate**:
  ```bash
  # On your local machine
  ssh-keygen -t ed25519 -C "github-actions@bterminal" -f ~/.ssh/bterminal_deploy
  
  # Copy the public key to your VPS
  ssh-copy-id -i ~/.ssh/bterminal_deploy.pub user@packages.bxitools.com
  
  # Use the private key content as the secret value
  cat ~/.ssh/bterminal_deploy
  ```
- **Required**: Yes

#### `VPS_DEPLOY_PATH`
- **Description**: The directory path on VPS where packages will be uploaded
- **Example**: `/var/packages-deploy`
- **Required**: Yes

### 2. Aptly API Secrets

These secrets are used to authenticate with the Aptly API on your VPS for package repository management.

#### `APTLY_API_AUTH`
- **Description**: Base64 encoded authentication credentials for Aptly API
- **How to generate**:
  ```bash
  # Replace USERNAME and PASSWORD with your actual credentials
  echo -n "USERNAME:PASSWORD" | base64
  ```
- **Example**: `YXB0bHk6c2VjcmV0cGFzc3dvcmQ=` (base64 of `aptly:secretpassword`)
- **Required**: Yes (if using Aptly API for package management)

#### `GPG_PASSPHRASE`
- **Description**: Passphrase for the GPG key used to sign packages
- **Note**: Only required if you set a passphrase when creating your GPG key
- **Required**: No (optional, only if GPG key has a passphrase)

## How to Add Secrets

1. Go to your repository on GitHub
2. Click on **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Enter the secret name (e.g., `VPS_HOST`)
5. Enter the secret value
6. Click **Add secret**

Repeat for each required secret.

## Verification

After adding all secrets, you can verify they're correctly configured by:

1. Going to **Actions** tab in your repository
2. Running the **Packages** workflow manually
3. Checking the workflow logs for any authentication errors

## Security Notes

- **Never commit secrets to the repository**
- Keep your SSH private keys secure
- Rotate credentials regularly
- Use strong, unique passwords
- Consider using different credentials for different environments (test/production)
- Regularly audit who has access to repository secrets

## Troubleshooting

### Issue: SSH connection failed
- Verify `VPS_HOST` is correct and reachable
- Check that `VPS_SSH_KEY` matches the public key on the VPS
- Ensure the VPS user has proper permissions

### Issue: Aptly API authentication failed
- Verify `APTLY_API_AUTH` is correctly base64 encoded
- Check that Aptly API is running on the VPS
- Verify the username/password matches what's configured on the VPS

### Issue: GPG signing failed
- If using a passphrase, ensure `GPG_PASSPHRASE` is set correctly
- Verify the GPG key is properly configured on the VPS
- Check GPG key permissions

## Additional Resources

- [GitHub Actions Encrypted Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [VPS Setup Guide](VPS_SETUP.md)
- [SSH Key Generation Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
