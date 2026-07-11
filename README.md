# LoRaFlexSim-2

LoRaFlexSim-2 is a Python LoRa/LoRaWAN simulator for running campaigns, aggregating results, and visualizing network metrics.

## Quick start: launch the dashboard

```powershell
panel serve loraflexsim/launcher/dashboard.py --show
```

## Basic dashboard usage

1. Launch the dashboard with `panel serve loraflexsim/launcher/dashboard.py --show`.
2. Set `Number of nodes`.
3. Set `Number of gateways`.
4. Set `Number of subchannels` and `Channel distribution`.
5. Set `Packets per node (0=infinite)` and `Max real duration (s)`.
6. Start the run with the `Start simulation` button.
7. Use `Fast-forward to end` if needed.
8. Click `Export results` after the simulation has finished.
9. Dashboard exports are written to `results/dashboard_exports/<timestamp>/`.

## Reproducing the paper experiments

The interactive dashboard can be used for exploratory simulation setup. The experiments reported in the paper were mainly executed through command-line runs to ensure repeatability. CSV files are automatically generated at the end of each simulation run. They can be regenerated from the documented scenario configurations and execution commands.

### Interactive dashboard

Use the dashboard for exploratory runs, quick parameter checks, and interactive inspection of simulation outcomes before committing a scenario to a repeatable batch run.

### Command-line execution

Use command-line runs for repeatable paper-style experiments, scripted campaign execution, and documented scenario configurations. The jamming CLI accepts `run`, `campaign`, and `aggregate` subcommands; the examples below only use options supported by `mobilesfrdth/jamming/cli.py`.

```powershell
loraflexsim --help
loraflexsim run --help
```

Run a short ADR-enabled scenario on three channels with a static jammed channel:

```powershell
loraflexsim run --scenario paper_fast --nodes 10 --adr on --seed 1 --sim-time 60 --channels 868.1,868.3,868.5 --jammed-channel 868.1 --channel-selection static --time-bin-size 10 --out results/cli_runs/paper_fast_seed1
```

Run the same kind of short scenario without ADR on a single channel:

```powershell
loraflexsim run --scenario paper_no_adr --nodes 10 --adr off --seed 2 --sim-time 60 --channels 868.1 --jammed-channel 868.1 --channel-selection static --time-bin-size 10 --out results/cli_runs/paper_no_adr_seed2
```

Run a compact campaign across several node counts, both ADR modes, and an inclusive seed range:

```powershell
loraflexsim campaign --scenario paper_campaign --nodes 10,20 --adr both --seeds 1:3 --sim-time 60 --channels 868.1,868.3,868.5 --jammed-channel 868.1 --channel-selection static --time-bin-size 10 --out results/cli_campaigns/paper_campaign
```

Scenario names are user-defined labels for the jamming CLI. They do not select built-in presets unless a YAML or JSON configuration is passed with `--config`.

### CSV outputs

CSV outputs are generated automatically at the end of each simulation run and can be compared across regenerated scenarios.

Dashboard exports are written under a timestamped directory:

```text
results/dashboard_exports/<timestamp>/
```

For CLI commands, `<--out>` corresponds to the value provided to the `--out` option. A `loraflexsim run` execution writes CSV outputs and the effective configuration under:

```text
<--out>/
  config_used.yaml
  per_run/run_summary.csv
  raw/packet_events_*.csv
  raw/node_metrics_*.csv
  raw/channel_timeseries_*.csv
  raw/sf_timeseries_*.csv
```

A `loraflexsim campaign` execution writes aggregate campaign outputs under:

```text
<--out>/
  aggregate/campaign_summary.csv
  ...
```

### Figure generation

Generate figures from CSV outputs that are already present in `results/`:

```powershell
python generate_figures.py
```

For direct QoS CLI usage, run:

```powershell
python -m qos_cli.lfs_plots --in results --config qos_cli/scenarios.yaml --out qos_cli/figures
```

These commands consume existing CSV results in `results/`. They do not automatically launch the main simulations.

## Recommended public entry path

1. **Primary path**: Panel dashboard.
2. **Scriptable path**: `loraflexsim` CLI.
3. **Technical fallback**: `python -m loraflexsim`.

Legacy names and legacy surfaces (`mobilesfrdth`, `sfrd/`, `src/`, `final/`) are no longer part of the public entry path.

## Quick start (Windows 11)

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e . --no-build-isolation
```

### Launch the dashboard (recommended)

```powershell
panel serve loraflexsim/launcher/dashboard.py --show
```

### Use the CLI (for automation)

```powershell
loraflexsim --help
loraflexsim presets --list
loraflexsim run --preset paper_fast --out runs/quickstart
```

### Fallback without a console entrypoint

```powershell
powershell -ExecutionPolicy Bypass -File scripts/loraflexsim.ps1 --help
python -m loraflexsim --help
```

## Active vs historical folders

| Area | Status | Usage |
| --- | --- | --- |
| `loraflexsim/` | active | engine, dashboard, public CLI |
| `docs/` | active | user documentation |
| `scripts/` | active | bootstrap and automation |
| `qos_cli/` | specialized | advanced QoS campaigns |
| `docs/archive_or_research/` | historical | migration and research memory |
| `pretest_campagne/` | historical/research | reproductions and comparisons |

## Removals / migration

- `sfrd/`, `src/`, and `final/`: removed from the live surface.
- `mobilesfrdth/`: kept only as an internal migration trace, documented as historical.
- Details: `docs/archive_or_research/migration_legacy_surfaces.md`.
