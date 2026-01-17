# git-glfix

Maintain gl: links in a git repository by tracking line changes.

## Synopsis

```
git-glfix [options] [command [args]]
```

## Description

git-glfix tracks and updates gl: links (e.g., `gl:path/to/file#L123`) as files change over time. It records snapshots in the repository and uses git history to adjust line references.

## Options

- `--dry-run` – Print what would be changed without modifying files.
- `--verbose` – Show detailed tracking information.
- `--json` – Output results in JSON format.
- `--update-modified` – Update links even if source files have uncommitted changes.

## Commands

- `status` – Show repository and snapshot status.
- `validate` – Scan for broken links without making changes.
- `cache <subcommand>` – Manage the snapshot cache. Subcommands: `list`, `show <commit>`, `clear`, `prune`, `rebuild`.
- `config <subcommand>` – View or modify configuration. Subcommands: `list`, `set <key> <value>`, `unset <key>`.

## Configuration

Configuration is stored in git config under the `gl-links.*` namespace:

- `gl-links.retention-days` – Days to keep daily snapshots (default: 7).
- `gl-links.retention-months` – Months to keep monthly snapshots (default: 1).
- `gl-links.modification-threshold` – Percentage change to trigger warning (default: 30).
- `gl-links.rename-threshold` – Similarity threshold for rename detection (default: 50).

## Files

- `.git/gl-links/` – Snapshot cache and index.

## Exit Status

0 on success; non-zero on error.

## Examples

Update links in the current repository:

```
git-glfix
```

Validate links without changes:

```
git-glfix validate
```
