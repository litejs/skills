---
name: litejs-release
description: Use when releasing a LiteJS package with lj release / lj r — version bumping, changelog generation, tagging, and publishing
license: MIT
---

# LiteJS Release Helper

Automates package releases: version bump, changelog, commit, tag, and publish instructions.

## Command

```sh
lj release [version]    # full form
lj r [version]          # shorthand
```

## What It Does (in order)

1. `git fetch` + verify upstream is ancestor of HEAD
2. `rm -rf node_modules && npm install`
3. `lj lint`
4. `npm outdated` (local deps)
5. `npm outdated -g` (global deps, if configured)
6. Bump version in package.json (date-based `YY.MM.0` or increment last segment)
7. Verify git tag `v<version>` does not exist
8. `lj build`
9. `lj test --brief test/index.js`
10. Generate changelog from commits, grouped into categories
11. Open `$EDITOR` (default: vim) for review
12. `git commit -a` + `git tag -a v<version>`
13. Print version and `npm publish` command

## Options

| Flag | Default | Effect |
|---|---|---|
| `--no-build` | build | Skip build step |
| `--no-install` | install | Skip node_modules reinstall |
| `--no-lint` | lint | Skip linting |
| `--no-test` | test | Skip tests |
| `--no-update` | update | Skip outdated check |
| `--no-upstream` | upstream | Skip upstream check |
| `--no-global` | global="" | Skip global outdated check |
| `--rewrite` | false | Amend the last release tag instead of creating new |

Options can also be set in `package.json#litejs` or `.github/litejs.json`.

## Version Bumping

- If current month/year is newer than last version: bumps to `YY.MM.0` (e.g. `26.2.0`)
- Otherwise: increments last segment (e.g. `26.1.0` -> `26.1.1`)
- Explicit version: `lj r 1.2.3` overrides auto-calculation
- Pre-release versions (4+ dot-segments like `1.2.3-beta.1`) always increment last segment

## Changelog Categories

Commits since last tag are grouped by subject line matching:

| Category | Pattern |
|---|---|
| New Features | `/\badd\b/i` |
| Removed Features | `/\b(remove\|drop)\b/i` |
| API Changes | `/\bapi\b/i` |
| Breaking Changes | `/\bbreak[ei]/i` |
| Fixes | `/fix\b/i` |
| Enhancements | catch-all |

The generated message looks like:

```
Release v26.2.0

New Features:

 - Add foo support (Author Name)

Fixes:

 - Fix bar edge case (Author Name)
```

## After Release

After the command completes it prints:

```
VERSION: 26.2.0
PUBLISH: npm publish
```

For pre-release versions it prints `npm publish --tag next`.

### Full release cycle

```sh
lj r                    # creates commit + tag locally
git push && git push --tags   # triggers CI
# CI runs tests, verifies tag signature, creates GitHub release, publishes to npm
```

## Rewrite Mode

`lj r --rewrite` amends the last release — rebases onto previous tag, cherry-picks, then `git commit --amend` and `git tag -f`.

## CI Integration

LiteJS repos use a shared GitHub Actions workflow (`litejs/.github/.github/workflows/release.yml`) triggered by `refs/tags/v*`:

1. Verify SSH signature on tag
2. Create GitHub release from tag message
3. `npm publish --provenance --access public` (tagged `latest` or `next`)

Requires: `RELEASE_SIGNERS` variable and `NPM_TOKEN` secret in GitHub repo settings.

## Troubleshooting

Each step can fail independently. The error message tells you which `--no-*` flag to use to skip it:

```
fatal: dependencies can not be installed! Ignore with --no-install option.
```

Common quick release (skipping slow steps):

```sh
lj r --no-install --no-update --no-global
```
