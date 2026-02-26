# CLI Reference

The `pd2` CLI is built with Typer.

## Commands

### pd2 validate

Validate an experiment configuration file.

```bash
uv run pd2 validate <config-path>
```

Checks: YAML syntax, agent references, payoff matrix constraints ($T > R > P > S$), condition uniqueness.

### pd2 run

Run an experiment.

```bash
uv run pd2 run <config-path> [OPTIONS]
```

| Option | Default | Description |
|---|---|---|
| `--replicates` | 15 | Number of replicates per condition |
| `--condition` | All | Run only a specific condition by name |
| `--output-dir` | `data/runs/<name>` | Output directory |
| `--seed` | Random | Base seed for reproducibility |

### pd2 analyze

Analyze experiment results (planned).

```bash
uv run pd2 analyze <run-dir> [OPTIONS]
```

### pd2 dashboard

Launch Streamlit dashboard.

```bash
uv run pd2 dashboard [--port 8501]
```
