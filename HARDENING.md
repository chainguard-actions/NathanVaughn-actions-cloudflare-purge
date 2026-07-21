<!-- markdownlint-disable -->

# Hardening Report: NathanVaughn--actions-cloudflare-purge/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **NathanVaughn--actions-cloudflare-purge/v4.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` step in action.yml directly interpolates nine GitHub Actions expressions into the shell command string (sub-rule a). GitHub Actions performs template substitution before the shell parses the string, so any of these values can contain shell metacharacters, quote characters, or command separators that break out of the intended command structure and execute arbitrary code.

Offending expressions on the `run:` line:
- `${{ inputs.python }}` — used as the interpreter itself; an attacker can supply a value like `python3 -c 'import os; os.system("...")'` or inject shell operators.
- `${{ github.action_path }}` — a github.* context value interpolated directly into the shell string.
- `${{ inputs.cf_zone }}`, `${{ inputs.cf_auth }}`, `${{ inputs.urls }}`, `${{ inputs.files }}`, `${{ inputs.tags }}`, `${{ inputs.hosts }}`, `${{ inputs.prefixes }}` — all inputs.* values interpolated directly; even though some are inside double-quotes in the YAML, the substitution happens before the shell sees the string, so embedded `"` or `$()` in the value escapes the quoting.

Fix: Move every expression into an `env:` block and reference the resulting environment variables (properly double-quoted) inside the `run:` script. For example:
```yaml
env:
  INPUT_PYTHON: ${{ inputs.python }}
  ACTION_PATH: ${{ github.action_path }}
  CF_ZONE: ${{ inputs.cf_zone }}
  CF_AUTH: ${{ inputs.cf_auth }}
  URLS: ${{ inputs.urls }}
  FILES: ${{ inputs.files }}
  TAGS: ${{ inputs.tags }}
  HOSTS: ${{ inputs.hosts }}
  PREFIXES: ${{ inputs.prefixes }}
run: |
  "$INPUT_PYTHON" "$ACTION_PATH/main.py" \
    --cf-zone="$CF_ZONE" --cf-auth="$CF_AUTH" \
    --urls "$URLS" --files "$FILES" --tags "$TAGS" \
    --hosts "$HOSTS" --prefixes "$PREFIXES"
```

Locations:

- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.python }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cf_zone }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cf_auth }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.urls }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.files }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.tags }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.hosts }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.prefixes }}" appears directly in run: block of step "Purge Cloudflare Cache"; move to env: map

Locations:

- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all 9 script injection findings in action.yml. Moved all ${{ }} expressions (${{ inputs.python }}, ${{ github.action_path }}, ${{ inputs.cf_zone }}, ${{ inputs.cf_auth }}, ${{ inputs.urls }}, ${{ inputs.files }}, ${{ inputs.tags }}, ${{ inputs.hosts }}, ${{ inputs.prefixes }}) out of the run: shell string and into an env: block (INPUT_PYTHON, ACTION_PATH, CF_ZONE, CF_AUTH, URLS, FILES, TAGS, HOSTS, PREFIXES). The shell script now references these as properly double-quoted environment variables. Optional inputs use the ${VAR:+--flag "$VAR"} pattern so they are omitted entirely when empty, preserving correct argument count for the Python script.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the reusable workflow reference in .github/workflows/tests.yml from `NathanVaughn/reusable-actions/.github/workflows/python-test-uv.yml@main` to `NathanVaughn/reusable-actions/.github/workflows/python-test-uv.yml@a0a27d752a10a92dcf2f92fdcc4fdd04ff9eb30d # main`, preventing supply-chain attacks via mutable branch references.

