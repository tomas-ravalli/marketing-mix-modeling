![My Cover](./assets/dp-cover.jpeg)

# Bayesian Marketing Mix Modeling for Budget Optimization

<p align="left">
  <img src="https://img.shields.io/badge/License-MIT-lightgrey" alt="License">
</p>

> An ML system that quantifies the impact of various marketing channels on sales. **Objective:** To leverage a Bayesian framework with flexible functional forms to accurately model advertising's lag and saturation effects, quantify uncertainty, and feed these probabilistic insights into a robust optimization engine to maximize portfolio ROI.

### Outline

- [Key Results](#key-results)
- [Overview](#overview)
- [Architecture](#architecture)
- [Dataset](#dataset)
- [Modeling](#modeling)
- [Structure](#structure)

---

## Key Results

| Metric | Result (Posterior Mean) | Description |
| :--- | :--- | :--- |
| 📈 **ROAS Improvement** | **+22%** | The optimized budget allocation, derived from the full posterior distributions, showed a significant expected increase in ROAS. |
| 💰 **Sales Uplift** | **+9%** (95% Credible Interval: 6–12%) | The model forecasted a robust increase in total sales, providing a credible range of outcomes for the proposed marketing strategy. |
| 🎯 **Budget Optimization** | **30%** Budget Reallocation | The optimizer recommended reallocating 30% of the budget from channels with low and uncertain returns to those with high, stable effectiveness. |
| 📊 **Model Fit** | **R²: 0.91** (BIC Selected) | The model's posterior predictive checks confirmed a strong fit to historical data, with functional forms chosen via Bayesian Information Criterion. |

## Overview

The core business challenge is allocating a multi-million dollar marketing budget under uncertainty. Advertisers use MMM to measure effectiveness, but advertising's complex dynamics–lag effects (carryover) and diminishing returns (saturation)–are difficult to capture with standard linear regression. This project addresses this by implementing a **Bayesian Marketing Mix Model**.

This approach allows us to incorporate prior knowledge from previous studies and treats all model parameters as probability distributions. By using **PyMC** with flexible functional forms, we can capture the full posterior distribution for each channel's contribution and ROI. This probabilistic output then feeds directly into an optimization layer (`scipy.optimize`), which finds the optimal budget allocation based not just on expected return, but also on the uncertainty of those returns.

<p align="center">
  <img src=".png" alt="Uncertainty Diagram" width="600">
  <br>
  <em>Fig. 1: Bayesian posteriors show Channel B has a higher mean ROI but also greater uncertainty than the more predictable Channel A.</em>
</p>

| 🚩 The Problem | 💡 The Solution |
| :--- | :--- |
| **Point-Estimate ROI**: Traditional models ignore the uncertainty around ROI, treating all estimates as equally certain. | **Probabilistic ROI**: Delivers a full probability distribution for each channel's ROI, enabling risk-aware decision-making. |
| **Rigid Assumptions**: Uses fixed, assumed shapes for adstock and saturation, which may not reflect reality. | **Flexible Functional Forms**: Models carryover (e.g., Weibull decay) and saturation (e.g., Hill function) as flexible curves whose parameters are learned from the data. |
| **Ignoring Past Knowledge**: Standard models are often built from scratch, ignoring valuable insights from previous analyses. | **Informative Priors**: The Bayesian framework systematically incorporates prior knowledge from past models to improve parameter estimation. |
| **Manual "What-Ifs"**: Planners must manually test a few budget scenarios, leaving the vast solution space unexplored. | **Automated Optimization**: Integrates with a numerical optimizer to automatically find the budget mix that maximizes a target KPI (e.g., revenue). |

## Architecture

The system is a prescriptive analytics pipeline that translates historical data into an optimal forward-looking strategy. It moves beyond simple prediction to active recommendation by integrating a Bayesian inference core with a consequential optimization engine.

<p align="center">
  <img src=".png" alt="Bayesian MMM Architecture" width="850">
  <br>
  <em>Fig. 2: [System Context Diagram] Bayesian MMM & Optimization Engine.</em>
</p>

## Dataset

This project uses a synthetic time-series dataset (`data.csv`) that simulates weekly sales and marketing activities. The data is structured to be representative of a typical MMM problem, including own-brand activities, competitor actions, and control variables.

### Features

| Category | Features | Description |
| :--- | :--- | :--- |
| **Response** | `y` | The quantity of our product sold per week. (Target Variable) |
| **Own Levers** | `x1` (Price), `x2` (Promotions), `x3` (Distribution), `x8` (TV Spend) | Key marketing and sales levers controlled by the company. |
| **Competitor** | `x4`, `x5`, `x6`, `x7`, `x9` | Price, promotion, distribution, and TV spend for two key competitors. |

## Modeling

The system's core is a hierarchical Bayesian model built in **`PyMC`**. This approach allows us to quantify uncertainty at every stage, from feature effects to the final ROI calculation. Attribution metrics like ROAS and channel contribution are calculated directly from the posterior samples, providing a complete distributional view.

<p align="center">
  <img src=".png" alt="Modeling and Optimization Loop" width="400">
  <br>
  <em>Fig. 3: The integrated modeling and optimization loop.</em>
</p>

| Component | Description | Toolkit |
| :--- | :--- | :--- |
| **Bayesian Inference** | A hierarchical Bayesian model estimates the posterior distribution for all parameters, leveraging prior knowledge and capturing a full range of plausible values for each channel's effectiveness. | **`PyMC`** |
| **Flexible Transformations** | Media carryover (**Adstock**) and **Saturation** effects are modeled as flexible functions (e.g., Weibull, Hill) whose parameters are learned from the data. | **`Python`** |
| **Model Selection** | The **Bayesian Information Criterion (BIC)** is used to compare and select the most appropriate functional form specifications for the adstock and saturation effects, balancing model fit and complexity. | **`Python`** |
| **Budget Optimization** | The posterior distributions of channel ROIs are passed to a numerical optimizer. It solves for the budget allocation that maximizes the expected revenue, subject to a total budget constraint. | **`SciPy.optimize`** |

## Structure

The repository is structured to separate data processing, modeling, inference, and optimization logic.

```
fcb-smartbooking/
├── .gitignore                            # (Public) Specifies files for Git to ignore.
├── LICENSE                               # (Public) Project license.
├── README.md                             # (Public) This project overview.
├── requirements.txt                      # (Private) The requirements file for the full project.
├── config.py                             # (Private) Configuration file for paths and parameters.
├── assets/                               # (Public) Diagrams and images for documentation.
├── data/
│   └── 03_synthetic/
│       ├── club_members_app.csv          # (Public) Synthetic raw seat release events.
│       └── match_data_timeseries.csv     # (Public) The final time-series modeling dataset.
├── notebooks/
│   └── eda.ipynb                         # (Private) Exploratory Data Analysis.
└── src/
    ├── __init__.py
    ├── data/
    │   ├── make_dataset_members.py       # (Public) Script to generate the members app data.
    │   └── make_dataset_match.py         # (Public) Script to generate the final time-series data.
    ├── features/
    │   └── build_features.py             # (Private) Feature engineering scripts.
    └── models/
        ├── train_availability_model.py   # (Private) Script for model training.
        └── predict_availability.py       # (Private) Script for generating predictions.
```

</br>

> [!WARNING]
> * **Data:** The data in this repository is synthetically generated for demonstration purposes and it may not mirror the statistical properties of the original dataset.
> * **Complexity:** This repository provides a high-level demonstration of the project's architecture and methodology. Certain implementation details and model complexities have been simplified for clarity.

</br>

<p align="center">🌐 © 2025 t.r.</p>
