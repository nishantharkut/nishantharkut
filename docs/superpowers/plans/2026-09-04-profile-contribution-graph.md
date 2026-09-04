# Profile Contribution Graph Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a reliable year-long contribution graph to the end of the profile README and rename Projects to Selected Projects.

**Architecture:** A scheduled GitHub Actions workflow fetches a rolling 365-day contribution calendar into temporary storage, validates its size, day count, XML, and safety constraints, then derives a dark variant locally. Both repository-owned assets are installed and committed only after every check passes, so a failed refresh leaves the last successful graph visible.

**Tech Stack:** GitHub Profile README HTML, GitHub Actions, GitHub Chart, Bash, SVG

---

### Task 1: Add the graph refresh workflow

**Files:**

- Create: `.github/workflows/contribution-graph.yml`

- [ ] **Step 1: Create the scheduled workflow**

```yaml
name: Update contribution graph

on:
  schedule:
    - cron: "17 2 * * *"
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: contribution-graph
  cancel-in-progress: true

jobs:
  update:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Check out profile repository
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          ref: main

      - name: Fetch and validate contribution graphs
        shell: bash
        run: |
          set -euo pipefail
          light="$RUNNER_TEMP/contributions-light.svg"
          dark="$RUNNER_TEMP/contributions-dark.svg"

          curl --fail --silent --show-error --location \
            --retry 3 --retry-delay 5 --retry-all-errors --max-time 45 \
            "https://ghchart.rshah.org/${{ github.repository_owner }}" \
            --output "$light"

          size=$(wc -c < "$light")
          days=$(grep -o 'data-date=' "$light" | wc -l)
          if (( size < 10000 || size > 1000000 )); then
            echo "Unexpected SVG size: $size bytes" >&2
            exit 1
          fi
          if (( days < 365 || days > 371 )); then
            echo "Unexpected contribution day count: $days" >&2
            exit 1
          fi
          if grep -Eiq '<script|<foreignObject|onload[[:space:]]*=|javascript:' "$light"; then
            echo "Unsafe SVG content detected" >&2
            exit 1
          fi

          sed -E -i 's#<!DOCTYPE[^>]*>##' "$light"
          cp "$light" "$dark"
          sed -i \
            -e 's/#eeeeee/#161b22/gI' \
            -e 's/#c6e48b/#0e4429/gI' \
            -e 's/#7bc96f/#006d32/gI' \
            -e 's/#239a3b/#26a641/gI' \
            -e 's/#196127/#39d353/gI' \
            -e 's/#767676/#8c959f/gI' \
            "$dark"

          python - "$light" "$dark" <<'PY'
          import sys
          import xml.etree.ElementTree as ET

          for path in sys.argv[1:]:
              ET.parse(path)
          PY

          mkdir -p profile
          install -m 0644 "$light" profile/contributions-light.svg
          install -m 0644 "$dark" profile/contributions-dark.svg

      - name: Commit refreshed graphs
        shell: bash
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add profile/contributions-light.svg profile/contributions-dark.svg
          git diff --cached --quiet && exit 0
          git commit -m "chore: refresh contribution graph"
          git push
```

- [ ] **Step 2: Validate workflow formatting and staged scope**

Run:

```powershell
npx --yes prettier@3.6.2 --check .github/workflows/contribution-graph.yml
git diff --check -- .github/workflows/contribution-graph.yml
```

Expected: Prettier reports that the workflow matches its style and Git reports no whitespace errors.

- [ ] **Step 3: Commit the workflow**

```powershell
git add -- .github/workflows/contribution-graph.yml
git commit -m "ci: generate yearly contribution graph"
```

### Task 2: Generate and verify the initial graph assets

**Files:**

- Create: `profile/contributions-light.svg`
- Create: `profile/contributions-dark.svg`

- [ ] **Step 1: Push the workflow to the default branch**

```powershell
git push origin HEAD:main
```

Expected: Git reports a fast-forward update to `main`.

- [ ] **Step 2: Run the workflow manually**

```powershell
gh workflow run contribution-graph.yml --ref main
$runId = gh run list --workflow contribution-graph.yml --limit 1 --json databaseId --jq '.[0].databaseId'
gh run watch $runId --exit-status
git pull --ff-only origin main
```

Expected: The workflow succeeds and the pull brings in a `chore: refresh contribution graph` commit containing both SVGs.

- [ ] **Step 3: Validate both generated assets**

```powershell
$files = @("profile/contributions-light.svg", "profile/contributions-dark.svg")
foreach ($file in $files) {
  if (-not (Test-Path $file)) { throw "Missing $file" }
  [xml](Get-Content -Raw $file) | Out-Null
}
```

Expected: Both files exist and PowerShell parses each as valid XML.

### Task 3: Add the selected-projects label and contribution section

**Files:**

- Modify: `README.md`

- [ ] **Step 1: Rename the projects heading**

Replace:

```markdown
## Projects
```

with:

```markdown
## Selected Projects
```

- [ ] **Step 2: Append the contribution graph after the Tools table**

```html
## Contributions

<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="./profile/contributions-dark.svg"
  />
  <img
    src="./profile/contributions-light.svg"
    width="100%"
    alt="Nishant Harkut's GitHub contributions over the past year"
  />
</picture>
```

- [ ] **Step 3: Validate README content and formatting**

Run:

```powershell
npx --yes prettier@3.6.2 --check README.md
rg -n "^## Selected Projects$|^## Contributions$|profile/contributions-(light|dark)\.svg" README.md
git diff --check -- README.md
```

Expected: Prettier passes, both headings and both asset paths are found, and Git reports no whitespace errors.

- [ ] **Step 4: Render light and dark desktop and mobile previews**

Render the README at 1280px and 390px widths in both color schemes. Confirm that the graph stays within the content width, remains legible, and does not overlap the Tools table or page edge.

- [ ] **Step 5: Commit the README update**

```powershell
git add -- README.md
git commit -m "docs: add yearly contribution graph"
```

### Task 4: Publish and verify

**Files:**

- No additional files

- [ ] **Step 1: Run final repository checks**

```powershell
npx --yes prettier@3.6.2 --check README.md .github/workflows/contribution-graph.yml
git diff --check origin/main..HEAD
```

Expected: Both checks pass.

- [ ] **Step 2: Push the final README commit**

```powershell
git push origin HEAD:main
```

- [ ] **Step 3: Verify the remote commit**

```powershell
$local = (git rev-parse HEAD).Trim()
$remote = ((git ls-remote origin refs/heads/main) -split "\s+")[0]
if ($local -ne $remote) { throw "Remote main does not match local HEAD" }
```

Expected: The local and remote commit hashes match.
