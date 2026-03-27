# Testing and Build Provenance

## Build provenance footer

The rendered dashboard site includes a collapsible footer showing the exact tool versions used for each build. This is a prototype for build traceability across the dashboard pipeline.

### How it works

The provenance system has three parts:

1. **Data build** (`build-data.yaml`): After the reusable data workflow completes, a `build-info` job queries tool versions and pushes `build-info.json` to the `ptc/data` branch. This captures:
   - predtimechart release tag
   - predevals Docker image digest
   - hub repo commit SHA
   - dashboard repo commit SHA
   - build timestamp

2. **Site build** (`build-site.yaml`): Before the reusable site workflow runs, an `update-site-builder-info` job queries the site-builder Docker image digest and updates `build-info.json` on `ptc/data`.

3. **Footer rendering** (`pages/resources/build-footer.html`): Client-side JavaScript fetches `build-info.json` from the GitHub API and renders a collapsible footer on every page. Responses are cached in `sessionStorage` (1 hour TTL) to avoid API rate limits.

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
