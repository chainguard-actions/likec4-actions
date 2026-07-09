<!-- markdownlint-disable -->

# Hardening Report: likec4--actions/v1.85.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **likec4--actions/v1.85.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag/version refs instead of pinned full SHA commits. action.yml uses a Docker image tag (docker://ghcr.io/likec4/actions:v1.85.0) instead of a SHA digest. Workflow files reference: actions/checkout@v4, elgohr/Publish-Docker-Github-Action@v5, booxmedialtd/ws-action-parse-semver@v1, wow-actions/use-app-token@v2, jacobtomlinson/gha-find-replace@v3, peter-evans/create-pull-request@v5, bobheadxi/deployments@v1, actions/upload-artifact@v4.

Locations:

- `action.yml:44`
- `.github/workflows/on-push-tag.yaml:22`
- `.github/workflows/on-push-tag.yaml:27`
- `.github/workflows/on-push-tag.yaml:43`
- `.github/workflows/on-push-tag.yaml:58`
- `.github/workflows/on-push-tag.yaml:64`
- `.github/workflows/release-init.yml:25`
- `.github/workflows/release-init.yml:31`
- `.github/workflows/release-init.yml:43`
- `.github/workflows/release-init.yml:48`
- `.github/workflows/release-init.yml:55`
- `.github/workflows/release-init.yml:61`
- `.github/workflows/release-init.yml:68`
- `.github/workflows/release-pr-merged.yml:13`
- `.github/workflows/release-pr-merged.yml:19`
- `.github/workflows/release-pr.yml:22`
- `.github/workflows/release-pr.yml:65`
- `.github/workflows/release-pr.yml:71`
- `.github/workflows/release-pr.yml:88`
- `.github/workflows/release-pr.yml:97`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:52`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing script injection. (1) on-push-tag.yaml 'create release' step: `gh release create "${{ github.ref_name }}"` and 'move tags' step: `git config user.name "${{ github.actor }}"`, `MAJOR="v${{ steps.semver_parser.outputs.major }}"`, `MINOR="${MAJOR}.${{ steps.semver_parser.outputs.minor }}"`. (2) release-init.yml 'check for tag' step: `TAG="${{ github.event.inputs.version }}"`. (3) release-pr-merged.yml 'get version' step: `HEAD_REF=$(printf "%q" "${{ github.event.pull_request.head.ref || github.head_ref }}")` and 'create tag' step: `git config user.name "${{ env.BOT_NAME }}"`. (4) release-pr.yml 'get version' step: `HEAD_REF=$(printf "%q" "${{ github.event.pull_request.head.ref || github.head_ref }}")`.

Locations:

- `.github/workflows/on-push-tag.yaml:49`
- `.github/workflows/on-push-tag.yaml:68`
- `.github/workflows/release-init.yml:36`
- `.github/workflows/release-pr-merged.yml:25`
- `.github/workflows/release-pr-merged.yml:43`
- `.github/workflows/release-pr.yml:79`

### github-env-injection (severity: high)

run: blocks write values derived from github context expressions to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). (1) release-pr-merged.yml 'get version' step: HEAD_REF is derived from ${{ github.event.pull_request.head.ref || github.head_ref }} and written to $GITHUB_OUTPUT via `echo "version=${HEAD_REF}" >> "$GITHUB_OUTPUT"` without newline stripping. (2) release-pr.yml 'get version' step: same pattern.

Locations:

- `.github/workflows/release-pr-merged.yml:28`
- `.github/workflows/release-pr.yml:82`

### missing-permissions (severity: medium)

Workflow files have no top-level permissions: key and no job-level permissions: key on any job, granting default (potentially broad) permissions. Affected files: release-pr-merged.yml, release-init.yml, and test.yml.

Locations:

- `.github/workflows/release-pr-merged.yml:1`
- `.github/workflows/release-init.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 6 files:

1. action.yml: Pinned docker://ghcr.io/likec4/actions:v1.85.0 to SHA digest sha256:dda2e14f45231ef0e562609e40ebfd266379fc6499e63795d75161cae361bbd4.

2. on-push-tag.yaml: Pinned actions/checkout@v4 -> @34e114876b0b11c390a56381ad16ebd13914f8d5, elgohr/Publish-Docker-Github-Action@v5 -> @1c2f28ccd9476e8a936ac9a1f287405504c93304, booxmedialtd/ws-action-parse-semver@v1 -> @3576f3a20a39f8752fe0d8195f5ed384090285dc. Fixed script injection in 'create release' step (moved github.ref_name to REF_NAME env var) and 'move tags' step (moved github.actor, semver outputs to env vars).

3. release-init.yml: Added permissions block (contents: write, pull-requests: write). Pinned actions/checkout@v4, wow-actions/use-app-token@v2, jacobtomlinson/gha-find-replace@v3 (x3), peter-evans/create-pull-request@v5 to full SHAs. Fixed script injection in 'check for tag' step (moved github.event.inputs.version to VERSION env var).

4. release-pr-merged.yml: Added permissions block (contents: write). Pinned wow-actions/use-app-token@v2 and actions/checkout@v4 to full SHAs. Fixed script injection in 'get version' step (moved github context expression to HEAD_REF_INPUT env var, removed printf %q injection vector). Fixed github-env-injection by sanitizing with printf '%s' | tr -d '\n\r' before writing to GITHUB_OUTPUT. Fixed script injection in 'create tag' step (moved env.BOT_NAME to BOT_NAME_VAL env var).

5. release-pr.yml: Pinned actions/checkout@v4, bobheadxi/deployments@v1 (x2), elgohr/Publish-Docker-Github-Action@v5 to full SHAs. Fixed script injection in 'get version' step (moved github context expression to HEAD_REF_INPUT env var). Fixed github-env-injection by sanitizing with printf '%s' | tr -d '\n\r' before writing to GITHUB_OUTPUT.

6. test.yml: Added permissions block (contents: read). Pinned actions/checkout@v4 and actions/upload-artifact@v4 to full SHAs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variables $MAJOR and $MINOR in the 'move tags' step of hardened/action/.github/workflows/on-push-tag.yaml. Changed `git tag -fa $MAJOR`, `git push origin $MAJOR --force`, `git tag -fa $MINOR`, and `git push origin $MINOR --force` to use properly double-quoted variables (`"$MAJOR"` and `"$MINOR"`) to prevent shell metacharacter injection from workflow-controllable step outputs.

