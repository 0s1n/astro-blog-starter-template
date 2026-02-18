# Troubleshooting

This document contains solutions to common issues you might encounter when working with this Astro blog starter template.

## GNOME Keyring Error

### Problem

When running Wrangler commands (such as `wrangler login`, `wrangler deploy`, or `wrangler dev`), you may encounter this error:

```
You're running in a GNOME environment but the OS keyring is not available for encryption. 
Ensure you have gnome-keyring or another libsecret compatible implementation installed and running.
```

### Why This Happens

This error occurs when:
- You're running on a Linux system with GNOME desktop environment (or `XDG_SESSION_TYPE=gnome`)
- Wrangler tries to securely store your Cloudflare API tokens using the system keyring
- The `gnome-keyring` service or a compatible `libsecret` implementation is not installed or not running

This commonly happens in:
- Headless Linux servers
- CI/CD environments (GitHub Actions, GitLab CI, etc.)
- Docker containers
- WSL (Windows Subsystem for Linux)
- Minimal Linux installations

### Solutions

#### Solution 1: Use Environment Variables (Recommended for CI/CD)

Instead of using `wrangler login`, authenticate using environment variables:

1. Get your Cloudflare API token from the [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Set the `CLOUDFLARE_API_TOKEN` environment variable:

```bash
export CLOUDFLARE_API_TOKEN="your-api-token-here"
```

3. Or set `CLOUDFLARE_ACCOUNT_ID` along with your API token:

```bash
export CLOUDFLARE_ACCOUNT_ID="your-account-id"
export CLOUDFLARE_API_TOKEN="your-api-token-here"
```

For GitHub Actions, add these as [repository secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) and use them in your workflow:

```yaml
- name: Deploy to Cloudflare
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
  run: npm run deploy
```

#### Solution 2: Disable Keyring Usage

Set the session type to avoid keyring detection:

```bash
export XDG_SESSION_TYPE=
# or
export XDG_SESSION_TYPE=tty
```

Then run your Wrangler commands:

```bash
npx wrangler login
```

#### Solution 3: Install GNOME Keyring (For Desktop Environments)

If you're on a desktop Linux system and want to use the keyring:

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install gnome-keyring libsecret-1-0
```

**Fedora/RHEL:**
```bash
sudo dnf install gnome-keyring libsecret
```

**Arch Linux:**
```bash
sudo pacman -S gnome-keyring libsecret
```

Then ensure the keyring daemon is running:
```bash
gnome-keyring-daemon --start --components=secrets
```

#### Solution 4: Use a Different Credential Store

You can configure Git and other tools to use a different credential helper that doesn't require gnome-keyring. For Wrangler specifically, using environment variables (Solution 1) is the recommended approach.

### Additional Tips

- **For local development**: Solutions 2 or 3 work well
- **For CI/CD**: Solution 1 is the best and most secure approach
- **For Docker containers**: Solution 1 or 2 (environment variables are recommended)

### Related Resources

- [Wrangler Authentication Documentation](https://developers.cloudflare.com/workers/wrangler/commands/#login)
- [Cloudflare API Tokens](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
