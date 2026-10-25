[![Run Hedging Engine](https://github.com/kleinnconrad/ml.prediction.depothedging/actions/workflows/run_hedger.yml/badge.svg)](https://github.com/kleinnconrad/ml.prediction.depothedging/actions/workflows/run_hedger.yml)
[![CI](https://github.com/kleinnconrad/ml.prediction.depothedging/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/kleinnconrad/ml.prediction.depothedging/actions/workflows/ci.yml)

# Depot Hedging Strategist

This repository contains a portfolio hedging engine. It ingests predictions from an external ML model and a current depot state, calculates advanced risk metrics (specifically robust Beta), and executes an optimization to find a portfolio allocation that maximizes expected return while guaranteeing a minimum return (e.g., 5%) under regular circumstances (risk-adjusted).

## Table of Contents

- [Folder Structure](#folder-structure)
- [Setup and Execution](#setup-and-execution)
- [Continuous Integration](#continuous-integration)
- [Algorithm and Logic](#algorithm-and-logic)
  - [1. Data Ingestion](#1-data-ingestion)
  - [2. Risk Metrics (Robust Beta)](#2-risk-metrics-robust-beta)
  - [3. Expected Returns (CAPM)](#3-expected-returns-capm)
  - [4. Portfolio Optimization](#4-portfolio-optimization)

## Folder Structure

- `src/`: Core logic of the application.
  - `ingestion.py`: Handles fetching remote JSON data and environment secrets.
  - `risk_metrics.py`: Contains custom functions for calculating advanced Beta.
  - `optimizer.py`: Contains the SciPy optimization logic.
- `config/`: Configuration and parameter files.
  - `settings.yaml`: Centralized configuration file for all user-defined parameters and thresholds.
- `tests/`: Offline tests (pytest).
- `.github/workflows/`: GitHub Actions pipelines (hedging engine, pre-merge checks, Release Please).
  - `main.py`: Entry point for the pipeline.
- `pyproject.toml`: Centralized project configuration, metadata, and dependencies.
- `scripts/`: Standalone utilities (e.g. `update_gist.py` for exporting results).

## Setup and Execution

1. **Environment:** Execute local scripts using the Conda `base` environment to ensure consistency across local development setups. We use `uv` for dependency management to ensure fully reproducible builds. If you introduce new dependencies, add them to the `dependencies` list in `pyproject.toml`, then update the lockfile and your environment:
   ```bash
   uv lock
   uv sync
   ```

2. **Configuration:** All configurable parameters (e.g., target returns, hedging assets, market variance) are located in `config/settings.yaml`. Do not hardcode these in the Python files.

3. **Execution:** 
   For local execution, provide the current depot state as a JSON string via the `CURRENT_DEPOT_JSON` environment variable in a `.env` file at the root of the repository (you can copy `.env.example` to start):
   ```
   CURRENT_DEPOT_JSON={"UN0.DE": 0.5, "ALV.DE": 0.5}
   ```
   Then simply run:
   ```bash
   python -m src.main
   ```
   When executed via GitHub Actions, the `CURRENT_DEPOT_JSON` environment variable is securely injected from the repository's GitHub Secrets.

4. **Results Export:**
   To keep your financial portfolio data private while hosting this algorithm in a public repository, the GitHub Action automatically exports the output (`results.json`) to a Secret GitHub Gist.
   To configure this:
   - Generate a GitHub Personal Access Token (PAT) with the `gist` scope.
   - Add it as a repository secret named `GIST_TOKEN`.
   - The GitHub pipeline will execute `scripts/update_gist.py` to automatically create or update a Secret Gist named `Depot Hedging Strategist - Optimal Weights`.

## Continuous Integration

The `CI` workflow (`.github/workflows/ci.yml`) runs on every pull request and on pushes to `main`. Development tools (pytest, ruff) are in the `dev` dependency group of `pyproject.toml`.

| Job | Checks |
|---|---|
| `Lint` | `uv.lock` matches `pyproject.toml`; `ruff check`; `actionlint` (including `shellcheck`) on the workflow files |
| `Tests` | The tests in `tests/` on Python 3.12 and 3.14. They currently verify that every module in `src/` imports with the locked dependencies. |
| `Conventional Commits` | Every commit of a pull request follows the Conventional Commits format required by Release Please (pull requests only) |
| `CI passed` | Succeeds only if all jobs above succeeded or were skipped; use it as the single required status check in the branch protection of `main` |

Run the Python checks locally before pushing:
```bash
uv lock --check
uv run ruff check .
uv run pytest
```

## Algorithm and Logic

The core logic of the hedging engine operates in three sequential phases: **Data Ingestion**, **Risk Metric Calculation**, and **Portfolio Optimization**.

```mermaid
sequenceDiagram
    participant Main
    participant Ingestion
    participant RiskMetrics
    participant Optimizer
    participant Storage

    Main->>Ingestion: Fetch Predictions
    Ingestion-->>Main: ML Predictions
    Main->>Ingestion: Load Current Depot
    Ingestion-->>Main: Current Portfolio
    Main->>Ingestion: Load Settings
    Ingestion-->>Main: Configuration
    
    loop Asset
        Main->>RiskMetrics: Calculate Adjusted Beta
        RiskMetrics-->>Main: Robust Beta and Market Cap
        Main->>RiskMetrics: Calculate Expected Return
        RiskMetrics-->>Main: Expected Return CAPM
    end
    
    Main->>RiskMetrics: Calculate Covariance Matrix
    RiskMetrics-->>Main: Single Index Covariance Matrix
    
    Main->>Optimizer: Optimize Portfolio
    Optimizer-->>Main: Optimal Weights Array
    
    Main->>Storage: Save Results to JSON
```

### 1. Data Ingestion
The system begins by dynamically fetching data from external and internal sources:
- **ML Predictions:** Retrieves expected returns for various assets from an external machine learning model endpoint.
- **Current Portfolio:** Reads the user's current holdings from the environment (`CURRENT_DEPOT_JSON`).
- **Settings:** Loads configurable constraints (e.g., target returns, maximum asset weights, variances) centrally from `config/settings.yaml`.

### 2. Risk Metrics (Robust Beta)
Standard Beta calculations often fail to capture the asymmetric risk of assets and thin trading. This engine computes a **Robust Beta** using a specialized sequence of adjustments:

1. **Downside Filtering:** Only days where the benchmark yields a negative return are considered, isolating downside risk (Downside Beta).
2. **Scholes-Williams Adjustment:** Compensates for asynchronous and thin trading by summing the covariances of the stock with the benchmark at $t-1$, $t$, and $t+1$.
3. **Blume's Adjustment:** Applies mean-reversion to the Beta (using a $0.67$ weight on the calculated Beta and $0.33$ on a default Beta of $1.0$).

Once the adjusted Betas are computed, they are used to populate a **Single-Index Covariance Matrix**:
- Diagonal (Variance): $\beta_i^2 \times \sigma_{market}^2 + \sigma_{idiosyncratic}^2$
- Off-Diagonal (Covariance): $\beta_i \times \beta_j \times \sigma_{market}^2$

### 3. Expected Returns (CAPM)
Before optimization, each asset is assigned an expected return based on its classification:
- **ML Predicted Stocks:** Assets flagged as `UP_FINAL_BUY` by the external model receive a fixed, high expected return (e.g., $10\%$) as defined in the configuration.
- **Existing Depot Assets:** For other stocks currently held in the portfolio, the **Capital Asset Pricing Model (CAPM)** is applied to calculate a realistic expected return using their previously computed Robust Beta, the risk-free rate ($R_f$), and the expected market return ($E(R_m)$). The formula used is: $E(R_i) = R_f + \beta_i (E(R_m) - R_f)$
- **Cash / Hedging Assets:** The `CASH` position explicitly bypasses the CAPM formula. Instead, it is assigned a hardcoded negative expected return (e.g., $-2.5\%$, mapped from `cash_beta` in the settings) to accurately penalize the asset for inflation risk and loss of purchasing power over time.

### 4. Portfolio Optimization
The final phase employs the **Markowitz Mean-Variance** framework using SciPy's `SLSQP` (Sequential Least Squares Programming) algorithm.

- **Objective Function:** Maximize the expected portfolio return ($W^T R$), which is implemented as minimizing the negative expected return.
- **Constraints:**
  - The sum of all asset weights (including the `CASH` position) must equal exactly $1.0$ ($100\%$).
  - **Risk-Adjusted Minimum Return:** The portfolio must guarantee the `min_portfolio_return` threshold (e.g., $5\%$) even under "regular circumstances", defined mathematically as subtracting the portfolio standard deviation multiplied by a configurable Z-score (e.g., $1.0$) from the expected return. Formula: $W^T R - (Z \times \sqrt{W^T \Sigma W}) \geq \text{min\_portfolio\_return}$.
- **Bounds:** Individual stock allocations are capped by `max_stock_weight` to enforce diversification, while hedging assets (like `CASH`) can float up to $1.0$.
