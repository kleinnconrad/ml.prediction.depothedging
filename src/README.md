# Source Code

## Table of Contents
- [Overview](#overview)
- [Scripts](#scripts)
  - [`main.py`](#mainpy)
  - [`ingestion.py`](#ingestionpy)
  - [`risk_metrics.py`](#risk_metricspy)
  - [`optimizer.py`](#optimizerpy)
- [Architecture Diagram](#architecture-diagram)

## Overview

This directory contains the core logic of the application. It consists of modules for data ingestion, risk metric calculation, and portfolio optimization.

## Scripts

### `main.py`

This is the main entry point for the portfolio hedging engine pipeline. It coordinates data ingestion, risk metrics calculation, and portfolio optimization, and saves the resulting weights to `data/results.json`. It saves them also if the optimization fails; these weights then do not necessarily meet the constraints (see [Consequences for the Hedging Engine](../README.md#consequences-for-the-hedging-engine)).

### `ingestion.py`

This script handles loading configuration and external data. It provides functions to load settings from the central configuration file (`config/settings.yaml`), fetch the current batch report of the stock predictor (with the buy candidates marked `UP_FINAL_BUY`) from the HTTP endpoint configured as `predictor_url`, and load the current portfolio state from environment variables.

### `risk_metrics.py`

This script calculates risk metrics for the portfolio assets. It retrieves historical market data, determines appropriate benchmark indices based on market capitalization, calculates adjusted downside betas (incorporating Scholes-Williams and Blume's adjustments), and computes a covariance matrix using the Single-Index Model approach.

### `optimizer.py`

This script performs the core portfolio optimization. It executes a Markowitz Mean-Variance optimization with SciPy's `SLSQP` algorithm that maximizes the expected portfolio return. The constraints are: the weights sum to 1.0, the expected return minus `regular_circumstance_z_score` portfolio standard deviations must be at least `min_portfolio_return`, each stock weight is between 0 and `max_stock_weight`, and each hedging asset (such as `CASH`) weight is between 0 and 1.0.

## Architecture Diagram

The following diagram illustrates the interactions between the modules:

```mermaid
sequenceDiagram
    participant Main as src/main.py
    participant Ingestion as src/ingestion.py
    participant RiskMetrics as src/risk_metrics.py
    participant Optimizer as src/optimizer.py
    participant External as External APIs (yfinance, HTTP)

    Note over Main,Ingestion: 1. Data Ingestion
    Main->>Ingestion: load_settings()
    Ingestion-->>Main: settings dict
    
    Main->>Ingestion: fetch_predictions()
    Ingestion->>External: HTTP GET
    External-->>Ingestion: JSON predictions
    Ingestion-->>Main: predictions dict
    
    Main->>Ingestion: load_current_depot()
    Ingestion-->>Main: depot dict
    
    Note over Main,RiskMetrics: 2. Risk Metrics Calculation
    loop For each symbol
        Main->>RiskMetrics: calculate_adjusted_beta(symbol)
        RiskMetrics->>External: Fetch Market Data (yf.download)
        External-->>RiskMetrics: Historical Prices
        RiskMetrics-->>Main: beta, mcap, benchmark
    end
    
    Main->>RiskMetrics: calculate_covariance_matrix(betas)
    RiskMetrics-->>Main: cov_matrix
    
    Note over Main,Optimizer: 3. Portfolio Optimization
    Main->>Optimizer: optimize_portfolio(expected_returns, cov_matrix)
    Optimizer-->>Main: optimal_weights
```
