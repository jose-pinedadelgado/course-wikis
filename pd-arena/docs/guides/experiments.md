# Running Experiments

## Experiment Setup

### 1. Create an Experiment

Go to **Experiments** → **New Experiment**.

### 2. Configure Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **Name** | — | Descriptive name for the experiment |
| **Rounds** | 100 | Number of rounds per game |
| **Horizon** | Fixed | Fixed (known end) or Geometric (unknown end) |
| **Stop probability** | 0.02 | For geometric horizon: probability game ends each round |
| **Replicates** | 10 | Games per condition (for statistical power) |

### 3. Set the Payoff Matrix

Default (standard PD):

| | Opponent: C | Opponent: D |
|---|:---:|:---:|
| **You: C** | 3, 3 | 0, 5 |
| **You: D** | 5, 0 | 1, 1 |

You can adjust all four values from the setup page.

### 4. Add Conditions (Drag & Drop)

The setup page has three panels:

- **Left:** Agent roster (LLM agents + Policy agents)
- **Center:** Condition builder with drop zones
- **Right:** Parameters (configured above)

**To add a condition:**

1. Drag an agent from the left panel into the "Agent A" drop zone
2. Drag another agent into the "Agent B" drop zone
3. The condition name auto-generates (e.g., "Cooperative vs ALLD")
4. Click **+ Add Condition** for more matchups

### 5. Save or Run

- **Save as Draft** — saves the experiment without running
- **Run Experiment** — saves and immediately executes all games

## What Happens During a Run

1. Experiment status → "Running"
2. For each condition:
    - For each replicate:
        - A `Game` object is created with a random seed
        - The game loop runs all rounds
        - Agent decisions are collected (mock or LLM)
        - Round results are bulk-saved to the database
        - Metrics are computed and stored on the Game
3. Experiment status → "Completed"

!!! note
    Currently runs synchronously — the page waits until all games finish. For large experiments (100+ games), this can take a while with mock agents (~seconds) but will take minutes-hours with real LLMs.

## Experiment Design Tips

### Fontana Replication

To replicate the baseline finding ("LLMs cooperate more than humans"):

1. Create 5 LLM persona agents (already seeded)
2. Set 100 rounds, fixed horizon, 10 replicates
3. Add conditions: each LLM persona vs. Random(0.0), Random(0.25), Random(0.5), Random(0.75), Random(1.0)
4. That's 25 conditions × 10 replicates = 250 games

### Pairwise Tournament

To compare all LLM personas against each other:

1. 5 personas = 10 unique pairs (+ 5 self-play = 15 conditions)
2. 10 replicates = 150 games

### LLM vs. All Policies

To test how one LLM persona handles every classic strategy:

1. 1 LLM × 6 policies = 6 conditions
2. 10 replicates = 60 games

## Viewing Results

After an experiment completes, the results page shows:

### Summary Cards

- Total games played
- Average cooperation rate across all conditions
- Best and worst performing matchups

### Cooperation Over Time Chart

A Chart.js line plot showing sliding-window cooperation rate per round, averaged across replicates. One line per condition. This is the key visualization for the paper.

### Condition Breakdown Table

Each condition with aggregated metrics:

- Cooperation rates (A and B)
- Mutual cooperation / defection rates
- Average payoff per round

Click into a condition to see individual replicate results.

### Game Detail

Click into any game to see the round-by-round table:

| Round | Agent A | Agent B | Payoff A | Payoff B |
|:---:|:---:|:---:|:---:|:---:|
| 1 | C | C | 3 | 3 |
| 2 | C | D | 0 | 5 |
| 3 | D | D | 1 | 1 |
| ... | ... | ... | ... | ... |

Plus extended metrics: retaliation rate, forgiveness rate, exploitation rates, cooperation-over-time plots.
