---
name: investigate-upgrade
description: This skill should be used when the user asks to "investigate a dependency upgrade", "check for breaking changes in an upgrade", "analyze upgrade risk", "what breaks if I upgrade", "upgrade impact analysis", or mentions upgrading a dependency and wants to understand the risk. Accepts an optional target version argument.
version: 0.1.0
---

# Investigate Upgrade Skill

## Purpose

Orchestrate a comprehensive investigation of breaking changes between the user's current dependency version and a target version (or latest). Identify all major version releases in that range, research breaking changes in parallel, then assess their impact on the user's codebase. Present a final risk-scored report as an HTML file ordered from highest to lowest risk.

**Important assumption:** This skill assumes the dependency has releases published on GitHub. Make this clear to the user if they seem uncertain.

## Required Information

Before beginning the investigation, gather the following from the user:

1. **Dependency name** - the package, chart, or library being upgraded
2. **GitHub repository** - the `owner/repo` where releases are published (e.g., `prometheus-community/helm-charts`)
3. **Current version** - the version the user is currently on
4. **Target version** (optional) - the version to upgrade to; defaults to "latest"
5. **Release tag naming convention** - how release tags are formatted in the repo (e.g., `kube-prometheus-stack-X.Y.Z`, `vX.Y.Z`, `X.Y.Z`). This is critical for identifying major releases.

If any required information is missing, ask the user before proceeding.

## Investigation Workflow

### Step 1: Identify Major Version Releases

Use the `gh` CLI to find all major version releases (X.0.0 pattern) between the current version and the target version. The release tag naming convention determines the query.

**Example** for `kube-prometheus-stack` in `prometheus-community/helm-charts`:

```sh
gh api \
  repos/prometheus-community/helm-charts/tags \
  --paginate \
  --jq '.[]
        | select(.name | test("^kube-prometheus-stack-[0-9]+\\.0\\.0$"))
        | .name'
```

Adapt the `test()` regex to match the user's release tag naming convention. Filter the results to include only versions between the current version and target version (exclusive of the current, inclusive of the target if it is a major version).

If no major versions are found, inform the user and check if the naming convention needs adjustment.

### Step 2: Understand Codebase Usage Context

Before dispatching agents, gather enough context about how the dependency is used in the codebase to inform the `dependency-usage-expert` where to look. Perform a quick scan:

- Search for import/require statements, configuration files, and manifest references for the dependency
- Identify the primary directories and files where the dependency is configured or consumed
- Note any related tooling (CI/CD pipelines, test fixtures, environment configs) that reference the dependency

Capture this as a **usage context summary** to pass to the `dependency-usage-expert` agent.

### Step 3: Dispatch Breaking Change Investigators

Divide the identified major versions into up to 3 roughly equal groups. Launch up to 3 `breaking-change-investigator` agents **in parallel**, each assigned a subset of the major versions to research.

Provide each agent with:
- The dependency name
- The GitHub repository (`owner/repo`)
- The release tag naming convention
- The specific major versions to investigate

Wait for all agents to complete and collect their breaking change reports.

### Step 4: Dispatch Dependency Usage Expert

Combine all breaking change findings from Step 3 into a single list. Launch the `dependency-usage-expert` agent with:
- The complete list of breaking changes (from all investigators)
- The usage context summary (from Step 2)
- The dependency name and relevant file paths to examine

The agent will return risk-scored assessments for each breaking change.

### Step 5: Compile and Present Final Report as HTML

Assemble the final report as an HTML file written to the project root (e.g., `upgrade-report-{dependency}.html`). The report contains a color-coded table sorted by risk score from highest (10) to lowest (1).

**HTML Report Structure:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Upgrade Report: {dependency} {current} → {target}</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; max-width: 1200px; margin: 0 auto; padding: 2rem; background: #f8f9fa; }
    h1 { color: #1a1a2e; }
    .summary { display: flex; gap: 1rem; margin: 1.5rem 0; }
    .summary-card { padding: 1rem 1.5rem; border-radius: 8px; color: white; font-weight: bold; }
    .card-high { background: #dc3545; }
    .card-med { background: #fd7e14; }
    .card-low { background: #28a745; }
    .card-total { background: #6c757d; }
    table { width: 100%; border-collapse: collapse; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
    th { background: #1a1a2e; color: white; padding: 12px 16px; text-align: left; }
    td { padding: 12px 16px; border-bottom: 1px solid #e9ecef; vertical-align: top; }
    tr.risk-high { border-left: 4px solid #dc3545; }
    tr.risk-med { border-left: 4px solid #fd7e14; }
    tr.risk-low { border-left: 4px solid #28a745; }
    .risk-badge { display: inline-block; padding: 4px 10px; border-radius: 12px; color: white; font-weight: bold; font-size: 0.85rem; }
    .badge-high { background: #dc3545; }
    .badge-med { background: #fd7e14; }
    .badge-low { background: #28a745; }
    .category { display: inline-block; padding: 2px 8px; border-radius: 4px; background: #e9ecef; font-size: 0.8rem; color: #495057; }
    .affected-files { font-family: monospace; font-size: 0.85rem; color: #6c757d; }
  </style>
</head>
<body>
  <h1>Upgrade Investigation: {dependency}</h1>
  <p><strong>{current_version}</strong> → <strong>{target_version}</strong></p>

  <div class="summary">
    <div class="summary-card card-total">Total: {count}</div>
    <div class="summary-card card-high">High Risk (7-10): {count}</div>
    <div class="summary-card card-med">Medium Risk (4-6): {count}</div>
    <div class="summary-card card-low">Low Risk (1-3): {count}</div>
  </div>

  <table>
    <thead>
      <tr>
        <th>Risk</th>
        <th>Version</th>
        <th>Breaking Change</th>
        <th>Category</th>
        <th>Impact &amp; Migration</th>
        <th>Affected Files</th>
      </tr>
    </thead>
    <tbody>
      <!-- For each breaking change, sorted by risk score descending: -->
      <tr class="risk-high|risk-med|risk-low">
        <td><span class="risk-badge badge-high|badge-med|badge-low">{score}/10</span></td>
        <td>{version}</td>
        <td>{description}</td>
        <td><span class="category">{category}</span></td>
        <td>{justification}<br><strong>Action:</strong> {migration_action}</td>
        <td class="affected-files">{file_paths}</td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```

Apply CSS classes based on risk score:
- **7-10**: `risk-high` / `badge-high`
- **4-6**: `risk-med` / `badge-med`
- **1-3**: `risk-low` / `badge-low`

After writing the HTML file, inform the user of the file path so they can open it in a browser. Also provide a brief text summary highlighting the number of high/medium/low risk items.

## Edge Cases

- **No major versions found**: The upgrade may only span minor/patch versions. Inform the user that no major version boundaries were crossed and breaking changes are unlikely, but recommend checking release notes for the specific version range.
- **Too many major versions (>15)**: Warn the user that the investigation will be extensive. Still split across 3 agents, but note that some findings may be less detailed.
- **Monorepo with multiple packages**: Ensure the tag filtering regex is specific to the dependency in question to avoid false matches from other packages in the same repo.
- **No GitHub releases**: If `gh` commands fail to find releases, inform the user that this skill requires GitHub releases and suggest alternative approaches (checking changelogs manually).
