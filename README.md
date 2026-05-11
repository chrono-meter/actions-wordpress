# actions-wordpress

Reusable GitHub Actions workflows for WordPress plugin and theme development.

## Available Workflows

| Workflow | Description |
|---|---|
| [lint-php.yml](#lint-php) | Lint PHP with PHPCS and WordPress Coding Standards |
| [test-php.yml](#test-php) | Run PHPUnit tests across multiple PHP versions |
| [lint-js.yml](#lint-javascript) | Lint JavaScript with ESLint |
| [test-js.yml](#test-javascript) | Run JavaScript tests (e.g. Jest) |
| [build.yml](#build) | Build front-end assets and upload as an artifact |
| [deploy-plugin.yml](#deploy-wordpress-plugin) | Deploy a plugin to WordPress.org via SVN |
| [deploy-theme.yml](#deploy-wordpress-theme) | Deploy a theme to WordPress.org via SVN |

---

## lint-php

Runs [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) with the
[WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards).

### Inputs

| Name | Description | Default |
|---|---|---|
| `php-version` | PHP version to use for linting | `8.2` |
| `phpcs-standard` | PHPCS standard (`WordPress`, `WordPress-Core`, `WordPress-Extra`, …) | `WordPress` |
| `phpcs-extensions` | Comma-separated list of file extensions to check | `php` |
| `phpcs-path` | Path(s) to run PHPCS against | `.` |

### Usage

```yaml
jobs:
  lint-php:
    uses: chrono-meter/actions-wordpress/.github/workflows/lint-php.yml@main
    with:
      php-version: '8.2'
      phpcs-standard: WordPress
```

---

## test-php

Runs [PHPUnit](https://phpunit.de/) against a live WordPress install (via the WordPress test suite)
across a matrix of PHP versions.

> **Prerequisite:** your repository must include a `bin/install-wp-tests.sh` script
> (the standard script shipped with `wp-cli/wp-cli`).

### Inputs

| Name | Description | Default |
|---|---|---|
| `php-versions` | JSON array of PHP versions to test against | `["8.1", "8.2", "8.3"]` |
| `wp-version` | WordPress version (`latest`, `6.4`, `nightly`, …) | `latest` |
| `mysql-version` | MySQL Docker image tag | `8.0` |
| `phpunit-config` | Path to PHPUnit configuration file | `phpunit.xml.dist` |

### Usage

```yaml
jobs:
  test-php:
    uses: chrono-meter/actions-wordpress/.github/workflows/test-php.yml@main
    with:
      php-versions: '["8.1", "8.2", "8.3"]'
      wp-version: latest
```

---

## lint-javascript

Runs your project's JavaScript linter via an `npm run` script.

### Inputs

| Name | Description | Default |
|---|---|---|
| `node-version` | Node.js version to use | `20` |
| `lint-script` | npm script to run for linting | `lint` |

### Usage

```yaml
jobs:
  lint-js:
    uses: chrono-meter/actions-wordpress/.github/workflows/lint-js.yml@main
    with:
      node-version: '20'
      lint-script: lint
```

---

## test-javascript

Runs your project's JavaScript test suite via an `npm run` script.

### Inputs

| Name | Description | Default |
|---|---|---|
| `node-version` | Node.js version to use | `20` |
| `test-script` | npm script to run for testing | `test` |

### Usage

```yaml
jobs:
  test-js:
    uses: chrono-meter/actions-wordpress/.github/workflows/test-js.yml@main
    with:
      node-version: '20'
      test-script: test
```

---

## build

Builds front-end assets via an `npm run` script and uploads the result as a
workflow artifact that can be consumed by subsequent jobs (e.g. deploy).

### Inputs

| Name | Description | Default |
|---|---|---|
| `node-version` | Node.js version to use | `20` |
| `build-script` | npm script to run for building | `build` |
| `artifact-name` | Name to use for the uploaded artifact | `build` |
| `artifact-path` | Directory to upload as the artifact | `build` |

### Outputs

| Name | Description |
|---|---|
| `artifact-name` | Name of the uploaded build artifact |

### Usage

```yaml
jobs:
  build:
    uses: chrono-meter/actions-wordpress/.github/workflows/build.yml@main
    with:
      node-version: '20'
      build-script: build
      artifact-name: plugin-build
      artifact-path: dist
```

---

## deploy-wordpress-plugin

Deploys a WordPress plugin to the [WordPress.org plugin repository](https://wordpress.org/plugins/)
using [10up/action-wordpress-plugin-deploy](https://github.com/10up/action-wordpress-plugin-deploy).

Trigger this workflow on a Git tag push to automatically publish a new release.

### Inputs

| Name | Description | Default |
|---|---|---|
| `slug` | WordPress.org plugin slug | repository name |
| `assets-dir` | Directory containing WP.org assets (banners, icons, screenshots) | `.wordpress-org` |
| `build-dir` | Directory with the built plugin to deploy (empty = entire repo) | `''` |
| `node-version` | Node.js version (if a build step is needed) | `20` |
| `php-version` | PHP version (if a build step is needed) | `8.2` |
| `dry-run` | Perform a dry run without actually deploying | `false` |

### Secrets

| Name | Description | Required |
|---|---|---|
| `svn-username` | WordPress.org SVN username | ✅ |
| `svn-password` | WordPress.org SVN password | ✅ |

### Usage

```yaml
on:
  push:
    tags:
      - '*'

jobs:
  deploy:
    uses: chrono-meter/actions-wordpress/.github/workflows/deploy-plugin.yml@main
    with:
      slug: my-plugin
    secrets:
      svn-username: ${{ secrets.SVN_USERNAME }}
      svn-password: ${{ secrets.SVN_PASSWORD }}
```

---

## deploy-wordpress-theme

Deploys a WordPress theme to the [WordPress.org theme repository](https://wordpress.org/themes/)
using [10up/action-wordpress-theme-deploy](https://github.com/10up/action-wordpress-theme-deploy).

### Inputs

| Name | Description | Default |
|---|---|---|
| `slug` | WordPress.org theme slug | repository name |
| `assets-dir` | Directory containing WP.org assets (banners, screenshots) | `.wordpress-org` |
| `build-dir` | Directory with the built theme to deploy (empty = entire repo) | `''` |
| `node-version` | Node.js version (if a build step is needed) | `20` |
| `php-version` | PHP version (if a build step is needed) | `8.2` |
| `dry-run` | Perform a dry run without actually deploying | `false` |

### Secrets

| Name | Description | Required |
|---|---|---|
| `svn-username` | WordPress.org SVN username | ✅ |
| `svn-password` | WordPress.org SVN password | ✅ |

### Usage

```yaml
on:
  push:
    tags:
      - '*'

jobs:
  deploy:
    uses: chrono-meter/actions-wordpress/.github/workflows/deploy-theme.yml@main
    with:
      slug: my-theme
    secrets:
      svn-username: ${{ secrets.SVN_USERNAME }}
      svn-password: ${{ secrets.SVN_PASSWORD }}
```

---

## Full CI/CD Example

The following example shows a complete pipeline that lints, tests, builds, and
deploys a WordPress plugin whenever a new tag is pushed.

```yaml
# .github/workflows/ci.yml  (in your plugin/theme repo)
name: CI/CD

on:
  push:
    branches: [main]
    tags: ['*']
  pull_request:

jobs:
  lint-php:
    uses: chrono-meter/actions-wordpress/.github/workflows/lint-php.yml@main

  test-php:
    uses: chrono-meter/actions-wordpress/.github/workflows/test-php.yml@main

  lint-js:
    uses: chrono-meter/actions-wordpress/.github/workflows/lint-js.yml@main

  test-js:
    uses: chrono-meter/actions-wordpress/.github/workflows/test-js.yml@main

  build:
    uses: chrono-meter/actions-wordpress/.github/workflows/build.yml@main
    with:
      artifact-path: dist

  deploy:
    if: startsWith(github.ref, 'refs/tags/')
    needs: [lint-php, test-php, lint-js, test-js, build]
    uses: chrono-meter/actions-wordpress/.github/workflows/deploy-plugin.yml@main
    with:
      build-dir: dist
    secrets:
      svn-username: ${{ secrets.SVN_USERNAME }}
      svn-password: ${{ secrets.SVN_PASSWORD }}
```