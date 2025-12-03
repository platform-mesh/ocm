# Platform Mesh OCM Component Release Process

This document describes how to create draft releases for the platform-mesh OCM component using the `/draft-release` slash command in Claude Code.

## Overview

The release process automates:
1. Fetching component versions from OCM registry and release candidates
2. Analyzing version changes across release candidates
3. Fetching changelogs from GitHub
4. Generating formatted release notes
5. Creating a draft GitHub release

## Prerequisites

### Required Tools

- **GitHub CLI (`gh`)**: For creating releases and fetching changelogs
  - Install: https://cli.github.com
  - Authenticate: `gh auth login`

- **jq**: For JSON processing
  - Install: `brew install jq` (macOS) or `apt-get install jq` (Linux)

- **yq**: For YAML processing
  - Install: `brew install yq` (macOS)

### Optional Tools

- **OCM CLI**: For fetching component versions from OCM registry
  - Install: https://github.com/open-component-model/ocm/releases
  - If not installed, the tool will warn but continue with limited functionality

### Authentication

1. **GitHub**: Must be authenticated with `gh` CLI
   ```bash
   gh auth login
   ```

2. **OCM Registry** (optional): Configure credentials for ghcr.io
   ```bash
   cat > ~/.ocmconfig << EOF
   type: generic.config.ocm.software/v1
   configurations:
     - type: credentials.config.ocm.software
       consumers:
         - identity:
             type: OCIRegistry
             scheme: https
             hostname: ghcr.io
             pathprefix: platform-mesh
           credentials:
             - type: Credentials
               properties:
                 username: github
                 password: <YOUR_GITHUB_TOKEN>
   EOF
   ```

## Usage

### Creating a Release

To create a draft release for version 0.1.0:

```bash
/draft-release 0.1.0
```

### Dry Run (Preview Only)

To preview what would be created without actually creating the release:

```bash
/draft-release 0.1.0 --dry-run
```

This will:
- Show all the release notes that would be generated
- Display which components changed
- Preview the final release format
- NOT create the actual GitHub release

### Process Flow

When you run `/draft-release 0.1.0`, the following happens:

1. **Prerequisites Check** (~5 seconds)
   - Validates required tools are installed
   - Checks GitHub authentication
   - Verifies OCM CLI availability (optional)

2. **Fetch Component Versions** (~30-60 seconds)
   - Queries OCM registry for all release candidates (e.g., 0.1.0-rc.1, 0.1.0-rc.2, etc.)
   - Fetches component versions from each RC
   - Extracts third-party versions from workflow YAML

3. **Analyze Changes** (~10-20 seconds)
   - Compares component versions across RCs
   - Identifies which components changed and in which RCs
   - Detects breaking changes

4. **Fetch Changelogs** (~20-40 seconds)
   - Fetches release notes from GitHub for each changed component
   - Aggregates changes across all RCs

5. **Format Release Notes** (~5 seconds)
   - Generates formatted Markdown
   - Highlights breaking changes with 🔥 emoji
   - Creates collapsible sections for unchanged components

6. **Create Draft Release** (~5 seconds)
   - Creates draft release on GitHub
   - Returns URL for review

**Total time**: ~1-2 minutes

## Release Notes Format

### For Initial Release (e.g., 0.1.0)

The release notes will include:
- Summary of release candidates aggregated
- Component changes across all RCs
- Breaking changes highlighted with 🔥
- Third-party component versions with links
- Complete version manifest (collapsible)
- Installation instructions

Example:
```markdown
# Platform Mesh OCM Component v0.1.0

This is the initial stable release of the Platform Mesh OCM component,
aggregating changes from 3 release candidates: 0.1.0-rc.1 0.1.0-rc.2 0.1.0-rc.3.

## Summary

This release includes 5 component updates, including 2 with breaking changes 🔥.

## Component Changes Across Release Candidates

### Platform Mesh Components

#### account-operator: v0.0.5 → v0.1.0 🔥
**Changes:**
- [rc.1] Feature: Add new account provisioning workflow
- [rc.3] Breaking: Update CRD schema for Account resource

[Full changelog](https://github.com/platform-mesh/account-operator/releases/tag/v0.1.0)
```

### For Subsequent Releases (e.g., 0.2.0)

Similar format, but comparing against the previous stable release instead of aggregating RCs.

## Troubleshooting

### "OCM CLI not found"

This is just a warning. The tool will still work but won't be able to fetch component versions from the OCM registry. You can:
1. Install OCM CLI: https://github.com/open-component-model/ocm/releases
2. Continue without it (limited functionality)

### "Error: Not authenticated with GitHub CLI"

Run: `gh auth login` and follow the prompts.

### "Error: Release X already exists"

The release version already exists on GitHub. Either:
1. Use a different version number
2. Delete the existing release on GitHub first
3. Edit the existing release manually

### Rate Limiting

If you hit GitHub API rate limits:
- Wait for the rate limit to reset (shown in error message)
- Use a personal access token with higher rate limits
- Run with `--dry-run` to preview without making API calls

## Files Created

The tool creates temporary files in the repository root:

- `component-versions.json`: Component versions from RCs
- `changelog.json`: Aggregated changelog data
- `release-notes.md`: Formatted release notes

These files can be safely deleted after the release is created, or kept for reference.

## Architecture

The release tool consists of:

```
.claude/
├── commands/
│   └── draft-release.md          # Main slash command (orchestrator)
└── scripts/
    ├── fetch-versions.sh         # Fetch component versions from OCM
    ├── generate-changelog.sh     # Fetch changelogs from GitHub
    ├── format-notes.sh           # Format release notes
    └── create-release.sh         # Create GitHub draft release
```

Each script is modular and can be run independently for testing:

```bash
# Fetch versions
./.claude/scripts/fetch-versions.sh 0.1.0 component-versions.json

# Generate changelog
./.claude/scripts/generate-changelog.sh component-versions.json changelog.json

# Format notes
./.claude/scripts/format-notes.sh 0.1.0 component-versions.json changelog.json release-notes.md

# Create release (dry run)
./.claude/scripts/create-release.sh 0.1.0 release-notes.md --dry-run
```

## Next Steps After Creating a Release

1. **Review**: Visit the GitHub releases page and review the draft release
2. **Edit**: Make any manual edits to the release notes if needed
3. **Publish**: When ready, publish the draft release to make it official

## Support

For issues or questions:
- Check this documentation
- Review the plan file: `.claude/plans/toasty-riding-cocoa.md`
- Run with `--dry-run` to debug issues
- Check script outputs in the terminal
