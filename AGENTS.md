## Repository Description
- `ocm` contains Platform Mesh OCM component definitions, release helpers, and workflow automation for building and publishing OCM artifacts.
- The main moving parts are constructor descriptors under `constructor/`, GitHub Actions workflows under `.github/workflows/`, and release/helper scripts under `hack/`.
- Read the org-wide [AGENTS.md](https://github.com/platform-mesh/.github/blob/main/AGENTS.md) for general conventions.

## Core Principles
- Keep changes small and auditable. This repo affects release metadata and published component artifacts.
- Prefer updating existing constructors, workflows, and scripts over introducing parallel release paths.
- Verify the exact file or workflow you are changing before editing it.
- Keep this file focused on agent execution and repository-specific constraints.

## Project Structure
- `constructor/`: OCM component constructor descriptors and example service component definitions.
- `.github/workflows/`: release, patch, validation, and publishing workflows.
- `hack/`: shell helpers for changelogs, releases, signing, and validation.
- `docs/`: workflow and release-process documentation.
- `signature/`: signing-related material used by the release flow.

## Architecture
This is a release automation and descriptor repo, not an application runtime.

### Release model
- Constructor YAML files define the inputs and structure used to build Platform Mesh OCM components.
- GitHub Actions workflows are the primary execution path for creating, patching, and publishing OCM artifacts.
- Shell scripts in `hack/` support those workflows and should stay aligned with the documented release process.

### Risk areas
- Small changes to constructor descriptors or release scripts can alter published component versions, metadata, or signing behavior.
- Workflow edits should be treated as production changes because they affect release outputs directly.

## Commands
- `bash hack/validate-links.sh` — validate repository links when touching docs or markdown.
- `bash hack/fetch-versions.sh` — inspect version data used by release helpers.
- `bash hack/generate-changelog.sh` — generate changelog content for release work.
- `bash hack/create-release.sh` — run the release helper script.

## Code Conventions
- Keep workflow logic in `.github/workflows/` and reusable shell logic in `hack/`.
- Prefer explicit, readable shell steps over dense one-liners when adjusting scripts.
- Update documentation in `docs/` when workflow or release behavior changes.
- Be careful with credentials, signing material, and release inputs; never hardcode secrets.

## Generated Artifacts
- Treat generated release notes or derived version files as outputs of the repo workflows and helpers.
- Do not mix unrelated manual edits into release-output changes.

## Do Not
- Introduce new release paths when an existing workflow or helper already covers the task.
- Change signing or release behavior casually.
- Commit secrets, private keys, or credential material.

## Hard Boundaries
- Ask before making changes that alter release semantics, signing behavior, or published component naming/versioning.
- Be especially careful when editing files under `.github/workflows/`, `constructor/`, and `signature/`.

## Human-Facing Guidance
- Use `README.md` for local certificate setup, startup arguments, and service context.
- Use `CONTRIBUTING.md` for contribution process, DCO, and broader developer workflow expectations.
