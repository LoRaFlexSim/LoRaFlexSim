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

Use command-line runs for repeatable paper-style experiments, scripted campaign execution, and documented scenario configurations.

### CSV outputs

CSV outputs are generated automatically at the end of each simulation run and can be compared across regenerated scenarios.

### Figure generation

Generate figures from the exported or regenerated CSV outputs so plots can be rebuilt from documented experiment data.

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
