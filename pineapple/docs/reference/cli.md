# CLI Commands

Pineapple provides four CLI commands via crewAI:

## pineapple run

Execute the agent crew for research and reporting.

```bash
uv run pineapple run
```

This is the default command. It runs all tasks sequentially and generates `report.md`.

## pineapple train

Train the crew for multiple iterations with performance tracking.

```bash
uv run pineapple train --n-iterations 5
```

| Flag | Description |
|---|---|
| `--n-iterations` | Number of training iterations |

## pineapple replay

Replay a previous task execution from a specific task ID.

```bash
uv run pineapple replay <task-id>
```

Useful for debugging and analyzing previous executions without re-running the full crew.

## pineapple test

Test crew execution with evaluation metrics.

```bash
uv run pineapple test --n-iterations 3
```

Runs the crew multiple times and reports performance metrics.
