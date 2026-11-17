<!-- markdownlint-disable -->

# Hardening Report: NathanVaughn--actions-cloudflare-purge/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **NathanVaughn--actions-cloudflare-purge/v4.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) is violated: GitHub Actions expressions are placed straight into the composite step's run: script. The offending line is: run: ${{ inputs.python }} ${{ github.action_path }}/main.py --cf-zone="${{ inputs.cf_zone }}" --cf-auth="${{ inputs.cf_auth }}" --urls "${{ inputs.urls }}" --files "${{ inputs.files }}" --tags "${{ inputs.tags }}" --hosts "${{ inputs.hosts }}" --prefixes "${{ inputs.prefixes }}". The values are substituted into the script before bash parses it, so an input containing quotes, $(...), backticks or ';' can run any command. inputs.python is also used as the command itself. Fix: set each value as an env: variable (e.g. INPUT_PYTHON, ACTION_PATH, CF_ZONE, CF_AUTH, URLS, FILES, TAGS, HOSTS, PREFIXES) and reference each one double-quoted in the script.

Locations:

- `action.yml:40`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index c936cda..8d3878a 100644
--- a/action.yml
+++ b/action.yml
@@ -37,4 +37,22 @@ runs:
     steps:
       - name: "Purge Cloudflare Cache"
         shell: bash
-        run: ${{ inputs.python }} ${{ github.action_path }}/main.py --cf-zone="${{ inputs.cf_zone }}" --cf-auth="${{ inputs.cf_auth }}" --urls "${{ inputs.urls }}" --files "${{ inputs.files }}" --tags "${{ inputs.tags }}" --hosts "${{ inputs.hosts }}" --prefixes "${{ inputs.prefixes }}"
+        env:
+          INPUT_PYTHON: ${{ inputs.python }}
+          ACTION_PATH: ${{ github.action_path }}
+          CF_ZONE: ${{ inputs.cf_zone }}
+          CF_AUTH: ${{ inputs.cf_auth }}
+          URLS: ${{ inputs.urls }}
+          FILES: ${{ inputs.files }}
+          TAGS: ${{ inputs.tags }}
+          HOSTS: ${{ inputs.hosts }}
+          PREFIXES: ${{ inputs.prefixes }}
+        run: |
+          "$INPUT_PYTHON" "$ACTION_PATH/main.py" \
+            ${CF_ZONE:+--cf-zone="$CF_ZONE"} \
+            ${CF_AUTH:+--cf-auth="$CF_AUTH"} \
+            ${URLS:+--urls "$URLS"} \
+            ${FILES:+--files "$FILES"} \
+            ${TAGS:+--tags "$TAGS"} \
+            ${HOSTS:+--hosts "$HOSTS"} \
+            ${PREFIXES:+--prefixes "$PREFIXES"}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I moved all nine `${{ }}` expressions out of the `run:` line of the "Purge Cloudflare Cache" step in action.yml and into its `env:` block. The new variables are INPUT_PYTHON, ACTION_PATH, CF_ZONE, CF_AUTH, URLS, FILES, TAGS, HOSTS and PREFIXES, and the script now references each one double-quoted. I haven't run the action or its tests after the change.

The command passes every flag on every run, the same way upstream does. This differs from the previous harden, which dropped a flag whenever its input was empty. Always passing them is safe because main.py already handles empty values: an empty `--cf-zone` or `--cf-auth` falls back to the CLOUDFLARE_ZONE and CLOUDFLARE_AUTH_KEY environment variables, and the list inputs (urls, files, tags, hosts, prefixes) are split on spaces inside main.py, with empty entries removed. That's also why each list goes in as one quoted argument. The `python` input is documented as a single interpreter path, so it is quoted too.

One thing to watch: a value like `uv run python` in `python` will no longer work, because it now runs as a single program name. The README only describes a path, so this shouldn't affect documented use.

