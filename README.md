# Dashboard Test Hub Dashboard

Test dashboard for the hubverse dashboard pipeline, paired with [dashboard-test-hub](https://github.com/hubverse-org/dashboard-test-hub).

## Purpose

This repo provides a controlled environment for:

- Testing tool changes (predtimechart, predevals, site-builder) against a known-good hub and config
- Testing new config schema versions against realistic data
- Staging changes end-to-end before releasing
- Running CI smoke tests for PRs to component repos

## Configuration

- **Hub**: [hubverse-org/dashboard-test-hub](https://github.com/hubverse-org/dashboard-test-hub) (fork of example-complex-forecast-hub)
- **Forecast target**: `wk inc flu hosp` (quantile output from 3 models)
- **Eval metrics**: WIS, AE median, interval coverage (50% and 95%)
- **Baseline model**: Flusight-baseline
- **Predevals schema**: v1.0.1

## Workflows

- `build-data.yaml`: Generates forecast and evaluation data via the control-room reusable workflow, then records build provenance
- `build-site.yaml`: Records site-builder version, then builds and deploys the Quarto site to GitHub Pages

See [TESTING.md](TESTING.md) for details on the build provenance system.

## Related

- [dashboard-test-hub](https://github.com/hubverse-org/dashboard-test-hub): Companion hub repo
- [hub-dashboard-control-room](https://github.com/hubverse-org/hub-dashboard-control-room): Workflow orchestration
- [hubverse-claude-skills](https://github.com/hubverse-org/hubverse-claude-skills): Claude Code skills for dashboard operations
