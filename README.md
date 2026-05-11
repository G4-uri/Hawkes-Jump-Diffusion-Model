# Hawkes-Jump-Diffusion-Model
Work/Paper Under Review 
# Endogenous Order Flow Dynamics and Jump Risk in Smart Order Routing
### A Hawkes Jump-Diffusion Study of BTC Microstructure, RWAs & TradFi Assets

> **Status:** Manuscript submitted to peer-reviewed journal — under review  
> **Author:** Gauri Santhosh Nair  
> **Affiliation:** Centre for Mathematical Needs, CHRIST (Deemed to be University), Bengaluru  
> **Repository:** Private — codebase available on request

---

## Overview

This repository contains the full implementation accompanying the paper. The framework embeds a **bivariate Hawkes Jump-Diffusion process** into the **Cont-Kukanov Smart Order Routing (SOR)** model to study endogenous order flow clustering and jump risk across financial markets.

The core idea: standard SOR models assume trade arrivals follow a Poisson process — memoryless and independent. In reality, trades cluster. One large informed order increases the short-term probability of further trades. Prices also jump suddenly when informed traders hit the book. This model captures both dynamics and uses them to make smarter routing decisions.

---

## Repository Structure

```
├── data/
│   ├── loaders/          # Tick data ingestion — Alpaca, HistData, Dukascopy, QO/A
│   └── preprocessing/    # Lee-Ready direction classification, midprice construction
│
├── models/
│   ├── hawkes/           # Bivariate Hawkes MLE estimation (pure NumPy, 3-start)
│   ├── jump_diffusion/   # Jump detection, parameter estimation
│   └── sor/              # Cont-Kukanov SOR optimisation (linear + quadratic cost specs)
│
├── simulation/
│   └── scenarios/        # Monte Carlo scenario generation (HJD + Poisson+D baseline)
│
├── results/
│   ├── diagnostics/      # Microstructure diagnostics, fill correlation, branching ratio
│   └── plots/            # Cost convergence charts, regime comparison plots
│
└── assets/               # Per-asset pipeline configs: SPY, QQQ, EUR/USD, Crude, XAUUSD
```

---

## Framework

### 1. Hawkes Process (Order Flow Clustering)

The bivariate Hawkes process models the arrival intensity of buy and sell orders as self-exciting — past events increase the likelihood of future events:

```
λ_sell(t) = μ_sell + Σ α_ss · g(t - t_i^sell) + Σ α_bs · g(t - t_i^buy)
λ_buy(t)  = μ_buy  + Σ α_bb · g(t - t_i^buy)  + Σ α_sb · g(t - t_i^sell)
```

Parameters estimated via **Maximum Likelihood Estimation** with 3-start optimisation to avoid local minima. Key output: branching ratio η — measures degree of self-excitation.

### 2. Jump-Diffusion (Price Risk)

Midprice dynamics follow a combined diffusion + jump process:

```
dS = μ dt + σ dW + J dN(λ_j)
```

Jumps detected from tick-level return series; parameters (σ_diff, σ_jump, λ_j) estimated from empirical distribution.

### 3. Cont-Kukanov SOR (Routing Optimisation)

Order allocation across Market, Limit-1, and Limit-2 venues minimises expected execution cost accounting for:
- Current Hawkes intensity (order flow clustering)
- Jump risk (adverse selection exposure)
- Fill probability at each venue

Two cost specifications: **linear** and **quadratic** market impact.

---

## Assets & Data Sources

| Asset | Source | Resolution | Ticks/Window |
|---|---|---|---|
| SPY | Alpaca API | Tick | ~50,000 trades |
| QQQ | Alpaca API | Tick | ~50,000 trades |
| EUR/USD | HistData | Tick | 420–1,495 ticks |
| Brent Crude (QO/A) | QO/A proxy | Tick | — |
| XAUUSD | Dukascopy BID+ASK | Tick | ~2,400 ticks |

---

## Key Results

| Asset | Windows | Cost Saving vs Baseline | Cost CV (HJD) | Fill Rate |
|---|---|---|---|---|
| SPY | 2 × 15-min | +77.6–77.7% | 0.000 | 100% |
| QQQ | 2 × 15-min | +79.5% | 0.000 | 100% |
| EUR/USD | 4 windows | +2.0–3.4% (NY Open) | 0.000 | 100% |
| Brent Crude | 2 × 15-min | +59.0–59.1% | 0.000 | 100% |
| XAUUSD | 2 × 1-hr | Cost model needs tuning | 0.30–0.64 | 88–92% |

**Cost CV = 0.000** on equities and crude means perfectly predictable execution cost across all 300 SOR iterations — the baseline Poisson+Diffusion model has CV = 5.3–5.6.

---

## Microstructure Diagnostics

Key Hawkes parameters calibrated per asset:

| Asset | Branching ratio η | Jump/diff ratio | Notes |
|---|---|---|---|
| SPY | 0.750 | 7.8× | Above typical 0.4–0.6 range |
| QQQ | 0.750 | 8.5× | Consistent with SPY |
| EUR/USD | 0.783–0.800 | 3.9–4.6× | Constraint binding in London Open |
| Brent Crude | 0.341–0.525 | Clean | Within expected range for liquid futures |
| XAUUSD | 0.700–0.706 | 3.4–3.7× | Correct calibration on tick data |
| BTC | ~0.711 | ~8–9× | Original paper dataset |

---

## Dependencies

```
python >= 3.8
numpy
scipy
pandas
matplotlib
```

No external quant libraries used — all Hawkes MLE, jump detection, and SOR optimisation implemented from scratch.

---

## Usage

```python
# Run full pipeline for a single asset
python run_asset.py --symbol SPY --start 09:30 --end 09:45 --date 2026-01-15

# Generate cross-asset summary
python run_all.py

# Plot cost convergence
python plot_convergence.py --assets SPY QQQ
```

---

## Citation

If you reference this work before publication, please cite as:

```
Nair, G. S. (2026). Endogenous Order Flow Dynamics and Jump Risk in Smart Order 
Routing: A Hawkes Jump-Diffusion Study of BTC Microstructure, RWAs and TradFi Assets. 
Manuscript submitted for publication. Centre for Mathematical Needs, CHRIST University.
```

---

## Notes

- Full methodology, derivations, and theoretical results are in the manuscript (available on request once published)
- This repository contains the empirical implementation only — novel theoretical contributions are reserved for the paper
- Parameter estimation code is included; the specific embedding architecture is described in the paper

---

*Repository maintained by Gauri Santhosh Nair · [LinkedIn](https://www.linkedin.com/in/gauri-nair-research) · [GitHub](https://github.com/G4-uri)*
