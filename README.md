# m2-ci-workflows

Reusable GitHub Actions workflows for testing **MagePsycho** Magento 2
extensions against a clean Magento install. Used as a `workflow_call`
target from each module repo so CI logic lives in one place.

## Usage

In a consumer module's `.github/workflows/test.yml`:

```yaml
name: Test

on:
  push:
    branches: [main, master, develop]
  pull_request:

jobs:
  test:
    uses: MagePsycho/m2-ci-workflows/.github/workflows/m2-extension-test.yml@v1
    with:
      composer_name: magepsycho/magento2-groupswitcherpro
      module_name: MagePsycho_GroupSwitcherPro
      # Optional overrides:
      # php_versions: '["8.3", "8.4"]'
      # magento_versions: '["2.4.8", "2.4.9"]'
```

## What it checks

For each `<php> × <magento>` matrix cell:

1. **Install verify** — boots a clean Magento install, requires the module
   via composer, runs `module:enable`, `setup:upgrade`, `setup:di:compile`,
   and basic HTTP smoke. Built on
   [extdn/github-actions-m2](https://github.com/extdn/github-actions-m2).
2. **Coding standard** — `magento/magento-coding-standard` against `src/`.
3. **Static analysis** — PHPStan with M2 awareness (optional, runs only if
   the consumer module ships a `phpstan.neon`).

If all jobs are green across the matrix, the module is considered safe to
release against the supported Magento versions.

## Maintenance

When a new Magento version drops (e.g. 2.4.10):

1. Bump the default `magento_versions` here.
2. Tag a new release: `git tag v2 && git push --tags`.
3. Bump the `@v1` ref in each consumer module's `.github/workflows/test.yml`.

Consumers stay pinned to a tag (`@v1`, `@v2`) so a breaking workflow
change in `master` doesn't ripple across all 7 module repos at once.
