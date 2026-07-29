<!-- markdownlint-disable -->

# Hardening Report: Azure--bicep-deploy/v2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--bicep-deploy/v2.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable version tags (e.g. @v3, @v4, @v6, @v7) instead of immutable 40-character commit SHAs. This exposes the workflows to supply-chain attacks if a tag is moved to a malicious commit. Affected references include: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7, azure/login@v3, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4.

Locations:

- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:28`
- `.github/workflows/check-dist.yml:57`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:63`
- `.github/workflows/ci.yml:82`
- `.github/workflows/ci.yml:101`
- `.github/workflows/ci.yml:107`
- `.github/workflows/ci.yml:113`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/test-permissions.yml:13`
- `.github/workflows/test-permissions.yml:16`
- `.github/workflows/upload-package.yml:18`
- `.github/workflows/upload-package.yml:24`
- `.github/workflows/upload-package.yml:45`

### missing-permissions (severity: medium)

The workflow file test-permissions.yml has no top-level 'permissions:' key and the only job ('test-permissions') also has no job-level 'permissions:' key. Without explicit permissions, the workflow inherits the repository default (which may be 'write-all' for older repositories), granting unnecessarily broad access to the GITHUB_TOKEN.

Locations:

- `.github/workflows/test-permissions.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions using the steps.*.outputs.* context are interpolated directly inside run: shell command strings, allowing injection of shell metacharacters before the shell ever sees the value.

In ci.yml, the 'Verify Deployment Outputs' step interpolates ${{ steps.test-deployment-create.outputs.intOutput }} and ${{ steps.test-deployment-create.outputs.stringOutput }} directly in a bash run: block. The 'Verify Stack Outputs' step similarly interpolates ${{ steps.test-stack-create.outputs.intOutput }} and ${{ steps.test-stack-create.outputs.stringOutput }}.

In upload-package.yml, the 'Upload files to partner drop folder' step interpolates ${{ steps.pack.outputs.filename }} and ${{ steps.pack.outputs.version }} directly in a run: block used to construct an az storage blob upload command. If the filename or version values contain shell metacharacters, arbitrary commands could be injected.

Locations:

- `.github/workflows/ci.yml:121`
- `.github/workflows/ci.yml:148`
- `.github/workflows/upload-package.yml:52`

### github-env-injection (severity: high)

In upload-package.yml, the 'Create package tarball' step writes the output of 'npm pack --json' (specifically the filename and version fields) directly to $GITHUB_OUTPUT without sanitization: 'echo "filename=$(echo $packOutput | jq -r '.[0].filename')" >> $GITHUB_OUTPUT' and 'echo "version=$(echo $packOutput | jq -r '.[0].version')" >> $GITHUB_OUTPUT'. The package name and version come from package.json, which can be influenced by an attacker in a pull request. No 'printf "%s" ... | tr -d "\n\r"' sanitization step is applied before the write. These unsanitized values are then used in a subsequent run: block to construct shell commands, compounding the risk.

Locations:

- `.github/workflows/upload-package.yml:38`
- `.github/workflows/upload-package.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings across 5 workflow files:

1. unpinned-uses: Pinned all 7 distinct action references to full 40-char commit SHAs with tag comments in check-dist.yml, ci.yml, codeql-analysis.yml, test-permissions.yml, and upload-package.yml.

2. missing-permissions: Added 'permissions: contents: read' top-level block to test-permissions.yml.

3. script-injection: In ci.yml, moved steps.test-deployment-create.outputs.* and steps.test-stack-create.outputs.* expressions from run: blocks into env: blocks (INT_OUTPUT, STRING_OUTPUT). In upload-package.yml, moved steps.pack.outputs.filename and steps.pack.outputs.version into an env: block (PACK_FILENAME, PACK_VERSION).

4. github-env-injection: In upload-package.yml 'Create package tarball' step, sanitized filename and version values using 'printf "%s" ... | tr -d "\n\r"' before writing to $GITHUB_OUTPUT to prevent newline injection.

