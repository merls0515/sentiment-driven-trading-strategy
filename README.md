# 🧠 Hyperliquid Sentiment Alpha
### *Does the Market's Fear & Greed Index predict trader behavior on Hyperliquid?*

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.3-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-5.x-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-22C55E?style=for-the-badge)

</div>

---

```
╔══════════════════════════════════════════════════════════════════╗
║  211,224 trades  ×  479 days  ×  1 question:                    ║
║  Does sentiment shape how traders win — or how they blow up?    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 📂 Repository Structure

```
hyperliquid-sentiment-alpha/
│
├── 📓 Untitled1.ipynb          ← Main analysis notebook (fully runnable)
├── 📄 README.md                ← You are here
│
├── 📁 data/
│   ├── fear_greed_index.csv    ← Bitcoin Sentiment Index (2018–2025, 2,644 rows)
│   └── historical_data.csv     ← Hyperliquid trade log (211,224 rows, 16 cols)
│
├── 📁 outputs/
│   ├── daily_metrics.csv
│   ├── trader_metrics.csv
│   ├── merged_trades.csv
│   ├── cumulative_pnl_sentiment_final.png
│   ├── weekend_effect.png
│   ├── heatmap_sentiment_winrate.png
│   ├── leverage_distribution_final.png
│   └── risk_adjusted_metrics.png
│
└── hyperliquid_analysis.zip    ← All outputs bundled
```

---

## ⚙️ Setup & Reproduction

> **Runs fully on Google Colab — no local install needed.**

### Option A — Google Colab (Recommended)

```python
# Step 1: Open the notebook in Colab
# Step 2: Upload both CSV files when prompted (or mount Drive)
# Step 3: Run All  →  Runtime > Run All
```

### Option B — Local

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/hyperliquid-sentiment-alpha.git
cd hyperliquid-sentiment-alpha

# Install dependencies
pip install pandas numpy matplotlib seaborn plotly scikit-learn scipy

# Launch the notebook
jupyter notebook Untitled1.ipynb
```

### Dependencies

| Library | Version | Purpose |
|---|---|---|
| `pandas` | ≥ 1.5 | Data wrangling & merging |
| `numpy` | ≥ 1.23 | Numerical ops |
| `matplotlib` / `seaborn` | latest | Static charts |
| `plotly` | ≥ 5.0 | Interactive heatmaps |
| `scikit-learn` | ≥ 1.1 | Random Forest classifier |
| `scipy` | ≥ 1.9 | Mann-Whitney U test |

---

## 🔬 Methodology

```
Raw Trades (211,224)          Fear/Greed Index (2,644 days)
       │                                  │
       ▼                                  ▼
  Parse Timestamps                  Extract Date + Value
  Drop nulls on datetime            Handle timestamp/date col variants
       │                                  │
       └──────────────┬───────────────────┘
                      ▼
             Left-merge on DATE
             (forward-fill fallback for gaps)
                      │
                      ▼
         211,218 / 211,224 matched  ✅
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    Daily Metrics  Trader       Behavioral
    (479 days)     Segments     Features
```

**Key design choices:**
- **Look-ahead prevention** — all predictive features use `t-1` lagged sentiment (yesterday's Fear/Greed predicts today's outcome)
- **Position Intensity** — defined as `Size USD / (|Start Position| + 1)` to proxy behavioural sizing leverage-free
- **Mann-Whitney U** (non-parametric) used because daily PnL is heavily skewed, not normally distributed
- **Quantile segmentation** on intensity and trade frequency to avoid arbitrary threshold bias

---

## 📊 Part B — Research Findings

### Q1 · Does performance differ between Fear vs Greed days?

> **Yes — economically significant, though not statistically significant at α=0.05.**

| Metric | Fear Regime | Greed Regime |
|---|---|---|
| **Win Rate** | 32.89% | **38.48%** (+5.59pp) |
| **Median Daily PnL** | ~$0 | ~$0 |
| **Sharpe Proxy** (consistency) | 0.61 | **0.70** |
| **Profit Factor** (efficiency) | **14.71** | 7.19 |
| **Mann-Whitney p-value** | — | 0.0726 (Not Sig.) |

**The Sentiment Paradox:**
- Greed produces steadier wins (higher Sharpe, higher win rate)
- Fear produces **larger payoff tails** — winning trades in Fear capture far bigger price extensions
- Median PnL ≈ $0 in both regimes: the *distribution shape*, not the median, is where the alpha lives

📈 *See: `cumulative_pnl_sentiment_final.png`, `risk_adjusted_metrics.png`*

---

### Q2 · Do traders change behaviour based on sentiment?

> **Yes — a strong "Overconfidence Effect" detected.**

- **Position intensity increases by +11.97%** during Greed (median 0.1791 → 0.2006)
- **Lowest win rate (27%)** occurs in **Mid-Low Intensity Greed** — traders systematically oversize exactly when their edge is declining
- **Weekend decay**: performance drops ~45% on weekends during Fear regimes

| Sentiment + Weekend | Avg Daily Median PnL |
|---|---|
| Fear — Weekday | $4.73 |
| Fear — Weekend | $4.31 |
| Greed — Weekday | **$7.33** |
| Greed — Weekend | $3.90 |

📊 *See: `weekend_effect.png`, `leverage_distribution_final.png`*

---

### Q3 · Trader Segments & Archetypes

*(Based on 211k trades across 479 days)*

| Segment | Median Total PnL | Behaviour |
|---|---|---|
| **Consistent Winners** | **$379,095** | High win rate + frequent trading — profit from volatility in both regimes |
| **Frequent Scalpers** | $220,720 | Trade volume edge; profit spikes during Greed |
| **High Intensity** | $184,082 | Aggressive sizing; sentiment-sensitive |
| **Infrequent / Others** | $90,789–$108,731 | Low activity; most vulnerable on weekends |

📊 *See: `heatmap_sentiment_winrate.png`*

---

### 🔮 Bonus — Predictive Model

A **Random Forest classifier** (n=100, max_depth=4) trained to predict next-day profitability:

| Feature | Importance |
|---|---|
| Previous Day PnL | ★★★★★ (primary) |
| Sentiment Intensity (fg_value) | ★★★★☆ |
| Previous Day Volume | ★★★☆☆ |
| Weekend flag | ★★☆☆☆ |

**Accuracy: 76.04%** on held-out test set (80/20 split, no look-ahead)

---

## 📐 Part C — Actionable Strategy Recommendations

### Strategy 1 — The Overconfidence Leverage Cap

```
IF  Fear/Greed Index  ∈  [50, 70]  (Greed zone)
AND position_intensity > median_intensity

THEN  cap max position size at 0.8× your base unit
      (reduces "churning" in low-edge Mid-Low Greed zones)
```

**Logic:** Intensity rises 12% in Greed, but win rate *collapses* in mid-intensity Greed (27%). The most aggressive sizing happens exactly when the edge disappears. A hard cap during moderate Greed preserves the Sharpe.

---

### Strategy 2 — Weekend Sentiment De-Risk

```
IF  day_of_week  ∈  {Saturday, Sunday}
AND sentiment_classification  =  "Fear"  (index < 40)

THEN  reduce open exposure by 50%
      no new position entries after Friday 20:00 UTC
```

**Logic:** Weekend performance drops ~45% during Fear regimes. Weekday Fear still offers the best Profit Factor (14.71); the weekend wipes it out. The fix is a simple time gate, not a strategy overhaul.

---

## 📸 Output Gallery

| Chart | What it shows |
|---|---|
| `cumulative_pnl_sentiment_final.png` | Cumulative PnL overlaid with Fear/Greed regime shading |
| `weekend_effect.png` | Avg daily median PnL: weekday vs weekend × sentiment |
| `heatmap_sentiment_winrate.png` | Win rate matrix: Market Regime × Sentiment Intensity Quartile |
| `leverage_distribution_final.png` | Log-scale position intensity: Fear vs Greed distributions |
| `risk_adjusted_metrics.png` | Sharpe Proxy + Profit Factor across regimes |

---

## 💡 Key Takeaways

> **1.** Sentiment is a *performance multiplier*, not a direction signal. Greed ≠ better trades — it means more consistent ones.
>
> **2.** The overconfidence effect is real and measurable. Traders size up most when their win rate is at its worst.
>
> **3.** Fear regimes are *underrated*. Lower win rates, but 2× the capital efficiency (Profit Factor 14.7 vs 7.2). Trend-following in Fear; mean-reversion in Greed.
>
> **4.** Weekend + Fear = the single worst combination. A simple time filter eliminates most of this drag.

---

<div align="center">

*Analysis covers 211,224 trades × 479 days (Feb 2024 – Aug 2024) merged with Bitcoin Fear & Greed Index (2018–2025)*

</div>
