# `.github/workflows/auto-release.yaml`
```yaml
# Trigger when you push a git tag matching the pattern `v*` (e.g., `v1.0.0`, `v2.1.0`)
on:
  push:
    tags:
      - 'v*'

jobs:
  main:
    uses: chrono-meter/actions-wordpress/.github/workflows/ci-plugin.yaml@main
    permissions:
      contents: write
      actions: write
```

See [details](./.github/workflows/ci-plugin.yaml) for `with` options.


# `package.json`
```json
{
  "scripts": {
    "ci:build": "npm ci --force && YOUR BUILD COMMANDS...",
    "ci:test": "npm ci --force && YOUR TEST COMMANDS..."
  }
}
```

## Wait! I'm using WordPress, but I need to configure NPM?
Oh, sorry...


# How It Works

This workflow automates the CI/CD pipeline for WordPress plugin releases:

## Build 
- Executes `npm run ci:build` for building the plugin
  - You can skip this step by `enable-build: false` in `with`
- Creates ZIP archive file `dist.zip`
- Uses `@chrono-meter/wp-zip-package` (and `.distignore` when present) to exclude unnecessary files.

## Test
- Executes `npm run ci:test` for running tests
- Full WordPress instance will be created with Apache (port `80` and `443`, self certified https) and MySQL.
- Installs the built ZIP through WP-CLI before tests run.
- Runs as matrix across `php-versions`, `wp-versions`, and `wp-languages`.
- You can use `playwright` for E2E testing.
- You can skip this job by `enable-tests: false` in `with`

## Test Environment Variables

During `npm run ci:test`, the following environment variables are available:

- `WP_ABSPATH`: Absolute path to the WordPress installation.
- `WP_BASE_URL`: Base URL of WordPress (must end with `/`).
- `WP_USERNAME`: WordPress admin username.
- `WP_PASSWORD`: WordPress admin password.
- `CI`: Always `true`.

## Release
- Renames ZIP archive file as `REPOSITORY-VERSION.zip`.
  - Example: repository `my-plugin` and tag `v1.2.3` -> `my-plugin-1.2.3.zip`
- Creates GitHub Release for the current tag and uploads the ZIP as a release asset.


## Troubleshooting

- Release not created:
  - Confirm workflow was triggered by a tag like `v1.2.3`.
  - Confirm `permissions`.
- Test cannot log in:
  - Ensure `WP_BASE_URL` has trailing `/` and matches your test assumptions.
  - Verify test code uses `WP_USERNAME` and `WP_PASSWORD`.
- Playwright report missing:
  - Report uploads only when `playwright-report/` exists.
