<!-- markdownlint-disable -->

# Hardening Report: likec4--actions/v1.89.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **likec4--actions/v1.89.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use tag-based (non-SHA-pinned) references. action.yml uses a Docker image with a mutable tag: 'docker://ghcr.io/likec4/actions:v1.89.0'. Workflow files use: actions/checkout@v4, elgohr/Publish-Docker-Github-Action@v5, booxmedialtd/ws-action-parse-semver@v1, wow-actions/use-app-token@v2, jacobtomlinson/gha-find-replace@v3, peter-evans/create-pull-request@v5, bobheadxi/deployments@v1, actions/upload-artifact@v4. None are pinned to a full 40-character commit SHA.

Locations:

- `action.yml:43`
- `.github/workflows/on-push-tag.yaml:22`
- `.github/workflows/on-push-tag.yaml:28`
- `.github/workflows/on-push-tag.yaml:45`
- `.github/workflows/on-push-tag.yaml:64`
- `.github/workflows/on-push-tag.yaml:71`
- `.github/workflows/release-init.yml:26`
- `.github/workflows/release-init.yml:33`
- `.github/workflows/release-init.yml:44`
- `.github/workflows/release-init.yml:52`
- `.github/workflows/release-init.yml:59`
- `.github/workflows/release-init.yml:68`
- `.github/workflows/release-init.yml:76`
- `.github/workflows/release-pr-merged.yml:13`
- `.github/workflows/release-pr-merged.yml:20`
- `.github/workflows/release-pr.yml:24`
- `.github/workflows/release-pr.yml:73`
- `.github/workflows/release-pr.yml:79`
- `.github/workflows/release-pr.yml:93`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:52`

### missing-permissions (severity: medium)

Workflow files release-init.yml, release-pr-merged.yml, and test.yml have no top-level 'permissions:' key and no job-level 'permissions:' keys. Without explicit permissions, workflows inherit the default (potentially write) token permissions, violating least-privilege.

Locations:

- `.github/workflows/release-init.yml:1`
- `.github/workflows/release-pr-merged.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. (a) on-push-tag.yaml 'create release' step: `gh release create "${{ github.ref_name }}"` — github.ref_name is interpolated directly into the shell command. (b) on-push-tag.yaml 'move tags' step: `git config user.name "${{ github.actor }}"`, `MAJOR="v${{ steps.semver_parser.outputs.major }}"`, `MINOR="${MAJOR}.${{ steps.semver_parser.outputs.minor }}"` — all interpolated directly. (c) release-init.yml 'check for tag' step: `TAG="${{ github.event.inputs.version }}"` — user-controlled workflow_dispatch input interpolated directly. (d) release-pr-merged.yml 'get version' step: `HEAD_REF=$(printf "%q" "${{ github.event.pull_request.head.ref || github.head_ref }}")` — attacker-controlled PR head ref interpolated directly. (e) release-pr-merged.yml 'create tag' step: `git config user.name "${{ env.BOT_NAME }}"` — env context interpolated directly. (f) release-pr.yml 'get version' step: same HEAD_REF pattern as (d).

Locations:

- `.github/workflows/on-push-tag.yaml:55`
- `.github/workflows/on-push-tag.yaml:78`
- `.github/workflows/on-push-tag.yaml:81`
- `.github/workflows/on-push-tag.yaml:85`
- `.github/workflows/release-init.yml:41`
- `.github/workflows/release-pr-merged.yml:28`
- `.github/workflows/release-pr-merged.yml:45`
- `.github/workflows/release-pr.yml:90`

### github-env-injection (severity: high)

In release-pr-merged.yml and release-pr.yml, the 'get version' step writes HEAD_REF (derived from the attacker-controllable github.event.pull_request.head.ref) to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Although printf "%q" is used for shell-quoting, it does not strip newlines, so a crafted ref containing newlines could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release-pr-merged.yml:31`
- `.github/workflows/release-pr.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across action.yml and 5 workflow files:

1. unpinned-uses: Pinned all action references to full SHA commits with tag comments. Pinned Docker image in action.yml to sha256 digest.
   - actions/checkout@v4 -> @34e114876b0b11c390a56381ad16ebd13914f8d5
   - elgohr/Publish-Docker-Github-Action@v5 -> @1c2f28ccd9476e8a936ac9a1f287405504c93304
   - booxmedialtd/ws-action-parse-semver@v1 -> @3576f3a20a39f8752fe0d8195f5ed384090285dc
   - wow-actions/use-app-token@v2 -> @0309c8980f645daa15f8909e70e6db96f3794999
   - jacobtomlinson/gha-find-replace@v3 -> @2ff30f644d2e0078fc028beb9193f5ff0dcad39e
   - peter-evans/create-pull-request@v5 -> @4e1beaa7521e8b457b572c090b25bd3db56bf1c5
   - bobheadxi/deployments@v1 -> @648679e8e4915b27893bd7dbc35cb504dc915bc8
   - actions/upload-artifact@v4 -> @ea165f8d65b6e75b540449e92b4886f43607fa02
   - ghcr.io/likec4/actions:v1.89.0 -> @sha256:e4fbddc7732119810cdc38e376e786735ed8edc358979b31a15534c657b6d115

2. missing-permissions: Added permissions blocks to release-init.yml (contents: write, pull-requests: write), release-pr-merged.yml (contents: write), and test.yml (contents: read).

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks and referenced them as plain env vars:
   - on-push-tag.yaml: github.ref_name, github.actor, semver outputs moved to env
   - release-init.yml: github.event.inputs.version moved to env
   - release-pr-merged.yml: github.event.pull_request.head.ref moved to env, env.BOT_NAME moved to env
   - release-pr.yml: github.event.pull_request.head.ref moved to env

4. github-env-injection: Fixed HEAD_REF writes to GITHUB_OUTPUT in release-pr-merged.yml and release-pr.yml by using printf '%s' ... | tr -d '\n\r' to sanitize before writing.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variables $MAJOR and $MINOR in the 'move tags' step of .github/workflows/on-push-tag.yaml. Changed `git tag -fa $MAJOR`, `git push origin $MAJOR --force`, `git tag -fa $MINOR`, and `git push origin $MINOR --force` to use double-quoted expansions `"$MAJOR"` and `"$MINOR"` to prevent shell metacharacter injection.

