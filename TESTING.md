# Testing and Build Provenance

## Build provenance footer

The rendered dashboard site includes a collapsible footer showing the exact tool versions used for each build. This is a prototype for build traceability across the dashboard pipeline.

### How it works

The provenance system has three parts:

1. **Site build** (`build-site.yaml`): Before the reusable site workflow runs, a `build-provenance` job uses the composite action (`.github/actions/build-provenance`) to query all tool versions and push `build-info.json` to an orphan `build-info` branch. This captures all tool versions, Docker image digests, repo commits, and the build timestamp in one step.

2. **Footer rendering** (`pages/resources/build-footer.html`): Client-side JavaScript fetches `build-info.json` from the GitHub API and renders a collapsible footer on every page. Responses are cached in `sessionStorage` (1 hour TTL) to avoid API rate limits.

### What's recorded

| Tool | Source | Field |
|------|--------|-------|
| predtimechart | Latest GitHub release tag | `predtimechart.version` |
| predevals Docker | Image digest via `crane` | `predevals_docker.digest` |
| site-builder Docker | Image digest via `crane` | `site_builder.digest` |
| Dashboard repo | `github.sha` from workflow | `dashboard.commit` |
| Hub repo | HEAD of `main` via GitHub API | `hub.commit` |
| Control room | Hardcoded ref (currently `main`) | `control_room.ref` |

### Limitations

- Tool versions are queried just before/after the reusable workflows, not during them. If a new release happens mid-build, the recorded version could differ slightly from what was actually used.
- The footer fetches `build-info.json` via the GitHub API at page load. Unauthenticated API requests are rate-limited to 60/hour per IP. Session caching mitigates this for individual users.
- The footer derives the repo owner/name from the GitHub Pages URL (`{owner}.github.io/{repo}`). Custom domains would need additional configuration.

### Future improvements

- Add `workflow_dispatch` inputs for tool version overrides (branch, tag, or SHA) so developers can test with specific dev versions and have the footer reflect exactly which branch was used
- Move version capture into the control-room reusable workflows as output parameters for exact build-time accuracy
- Add the provenance system to the dashboard template so all dashboards can opt in
