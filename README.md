# 🏆 FIFA World Cup 2026 — Monte Carlo Tournament Simulator

A machine learning-powered predictor that simulates the 2026 FIFA World Cup thousands of times to estimate each team's probability of winning, reaching specific rounds, and other tournament outcomes.

## 📋 Overview

This project uses:
- **Historical match data** (2010-2026) with tournament importance weighting
- **FIFA Elo rating system** to track team strength over time
- **Random Forest classifier** trained on 1,900+ international matches
- **Monte Carlo simulation** (10,000+ runs) to estimate tournament probabilities
- **Realistic score distributions** based on historical goal patterns

The simulator accounts for:
- Home advantage for host nations (USA, Mexico, Canada)
- Goal difference tie-breaking in group stages
- Best 8 third-place finishers advancement format
- Neutral venue knockout rounds

## 🎯 Key Features

### 1. **Elo Rating System**
Tracks team strength based on match results with tournament-weighted importance:
- World Cup & Continental Finals: **K=60** (high volatility)
- Qualifiers: **K=30-40** (moderate)
- Friendlies: **K=20** (low)

Each team starts at **1500 Elo**. Upsets (beating stronger teams) cause larger rating swings.

### 2. **Machine Learning Model**
- **Algorithm**: Random Forest (100 trees, max_depth=4)
- **Features**: 
  - `elo_diff` — home team Elo advantage
  - `rank_diff` — home team FIFA ranking advantage
  - `h_elo_pre` & `a_elo_pre` — absolute Elo ratings
  - `neutral` — venue advantage flag
- **Output**: Probability of home win, draw, or away win
- **Training**: 1,932 matches, weighted by tournament importance

### 3. **Score Distribution Priors**
Realistic goal scoring patterns:

| Outcome | Distribution |
|---------|--------------|
| **Draws** | 33.8% end 0-0, 44.6% end 1-1, 18.5% end 2-2, 3.1% end 3-3 |
| **Wins** | Win margins: 28.2% (1 goal), 39.8% (2 goals), 19.9% (3 goals), ... |
| **Losses** | Conceded goals: 57.3% (0), 35.5% (1), 6.9% (2), 0.2% (3+) |

### 4. **Tournament Simulation**
Each Monte Carlo run:
1. **Group Stage** — 6 matches per group, teams ranked by points/GD/GF/FIFA rank
2. **Best 8 Third Places** — Top 8 third-place finishers qualify
3. **Round of 32** — 16 qualified teams (2 per group) + best 8 third-place teams
4. **Knockout Rounds** — Quarterfinals → Semifinals → Final
5. **Winner Determination** — Draws impossible in knockouts (50/50 probability split)

## 📊 Data Sources

| File | Contents |
|------|----------|
| `results.csv` | 15,790 international matches (2010-2026) |
| `fifa_rankings.csv` | Current FIFA rankings for all 48 participating teams |
| `world_cup_groups.csv` | 2026 World Cup group assignments (A-L) |
| `former_names.csv` | Country name standardization (e.g., West Germany → Germany) |

## 🚀 Usage

### Installation
```bash
pip install numpy pandas scikit-learn
```

### Running the Simulator
```python
# Run 10,000 tournament simulations
num_simulations = 10000
rng = np.random.default_rng(RANDOM_STATE)

results_by_team = {t: {'champion': 0, 'finalist': 0, 'semifinal': 0, 'round_of_16': 0, 'qualified': 0} for t in RANKED_TEAMS}

for sim_num in range(num_simulations):
    qual, r32w, round_reached, champion = simulate_tournament(rng)
    results_by_team[RANKED_TEAMS[champion]]['champion'] += 1
    # ... track other outcomes
```

### Interpreting Results
The output shows each team's probabilities:
- **Champion**: % chance of winning the entire tournament
- **Finalist**: % chance of reaching the final
- **Semifinalist**: % chance of reaching the semifinals
- **Round of 16**: % chance of reaching the knockouts
- **Qualified**: % chance of advancing from group stage

## 📈 Model Performance

- **Training Data**: 1,932 ranked team matches
- **Feature Importance**: Elo difference > FIFA rank difference > absolute ratings
- **Validation**: Balanced class weights to account for draws (rarer than decisive matches)
- **Generalization**: Shallow trees (max_depth=4) prevent overfitting on historical quirks

## 🔧 Architecture

### Main Components

```
1. Data Loading & Cleaning
   ├─ Load historical results, rankings, groups
   ├─ Standardize country names
   └─ Filter to 2010-2026 period

2. Elo Rating Calculation
   ├─ Walk through all matches chronologically
   ├─ Apply tournament-weighted K factors
   └─ Build final Elo snapshot for all teams

3. Model Training
   ├─ Engineer features (Elo diff, rank diff, etc.)
   ├─ Train Random Forest with sample weights
   └─ Store class probabilities

4. Probability Pre-computation
   ├─ Compute all 2,256 possible matchups (48 teams, 47 opponents each)
   ├─ Cache in 48×48 matrices for fast lookup
   └─ Account for host nation home advantage

5. Tournament Simulation (10,000+ runs)
   ├─ Simulate group stage with realistic scores
   ├─ Rank teams by points, GD, GF, FIFA rank
   ├─ Select best 8 third-place teams
   ├─ Run Round of 32 through Final
   └─ Aggregate statistics across all runs
```

## 📝 Key Insights

### Why This Approach?

1. **Elo captures form** — Teams improve/decline based on recent results
2. **Tournament weighting** — World Cup matches influence the model more than friendlies
3. **Feature engineering** — Differences (not absolutes) matter most for prediction
4. **Realistic simulations** — Goal distributions ensure credible scorelines for tie-breaking
5. **Large sample size** — 10,000 simulations reduce variance and show true probabilities

### Limitations

- Historical data may not reflect recent team changes
- Injuries/roster changes not modeled
- Home advantage quantified only for host nations, not for groups with regional bias
- Random draws in qualification rounds may skew group compositions

## 🎨 Results Visualization

Top 10 teams by Elo (as of 2026-06-08):
1. **Spain** — 2014.1
2. **Argentina** — 1967.1
3. **France** — 1915.8
4. **Morocco** — 1911.5
5. **England** — 1890.6
6. **Japan** — 1885.6
7. **Portugal** — 1867.2
8. **Mexico** — 1866.3
9. **Brazil** — 1857.0
10. **Germany** — 1852.9

## 📚 References

- Elo Rating System: https://en.wikipedia.org/wiki/Elo_rating_system
- FIFA Rankings: https://www.fifa.com/fifa-world-ranking/
- 2026 World Cup Format: https://www.fifa.com/fifaplus/en/articles/fifa-world-cup-2026-format

## 📄 License

This project is open source and available for educational use.


**Last Updated**: June 2026 | **Data Cutoff**: June 8, 2026
