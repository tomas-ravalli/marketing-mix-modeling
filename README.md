![My Cover](./assets/dp-cover.jpeg)

# Bayesian Marketing Mix Modeling for Budget Optimization

<p align="left">
  <img src="https://img.shields.io/badge/License-MIT-lightgrey" alt="License">
</p>

> An ML system that quantifies the impact of various marketing channels on sales for a top petcare brand. **Objective:** To leverage a Bayesian framework with flexible functional forms to accurately model advertising's lag and saturation effects, quantify uncertainty, and feed these probabilistic insights into a robust optimization engine to maximize portfolio ROI.

### Outline

- [Key Results](#key-results)
- [Overview](#overview)
- [Architecture](#architecture)
- [Dataset](#dataset)
- [Modeling](#modeling)
- [Structure](#structure)

---

## Key Results

This initiative provides a forward-planning tool to simulate and optimize marketing investments. Based on the historical analysis of the brand's top product performance, the model delivers the following key insights:

| Metric | Result (Posterior Mean) | Description |
| :--- | :--- | :--- |
| 🎯 **Optimal Mix Recommendation** | **15% Budget reallocation%** | The model suggested a budget reallocation of 15% from traditional print and linear TV towards Digital Video (YouTube) and Paid Search. |
| 💰 **Projected ROI Uplift** | **+6.7%** (95% CI: 4.5%–8.9%) | This optimized mix increased incremental revenue by an estimated $3.2M per quarter (+6.7% ROI) at the same level of investment. |
| 📈 **Saturation Insights** | Facebook Ads: saturation at **~$75K/wk** | Analysis of saturation curves revealed that spending on Facebook Ads has reached a point of diminishing returns. In contrast, channels like YouTube and influencer marketing showed significant room for growth before saturation. |
| 📊 **Simulation Impact** | **+9%** PFME effectiveness gain| A simulation doubling the Digital Media spend, funded by proportionally cutting other channels, showed a potential +9% gain in PFME effectiveness, though with declining ROIs for the saturated digital channels. |

## Overview

Top petcare brand's marketing team invests in a diverse portfolio of channels, including traditional media (TV, print), digital campaigns (social media, search), and trade promotions (in-store displays, discounts). The complexity of these simultaneous activities makes it difficult to disentangle their individual impact on sales. 

This project implements a Marketing Mix Model (MMM) to quantify the precise contribution of each marketing lever to sales revenue. By understanding the effectiveness and efficiency of past investments, the system provides a robust framework for optimizing future budget allocations to maximize product revenue.

<p align="center">
  <img src="./assets/mmm1.jpeg" alt="Uncertainty Diagram" width="750">
  <br>
  <em>Fig. 1: The challenge is to isolate the impact of each activity on sales.</em>
</p>


## Architecture

The system is a prescriptive analytics pipeline that translates historical data into an optimal forward-looking strategy. It moves beyond simple prediction to active recommendation by integrating a Bayesian inference core with a consequential optimization engine.

<p align="center">
  <img src="./assets/mmm-scd.png" alt="Bayesian MMM Architecture" width="850">
  <br>
  <em>Fig. 2: [System Context Diagram] Bayesian Marketing Mix Modeling</em>
</p>

The system is designed as a three-stage pipeline:

- **Data Ingestion**: Weekly data from various sources (Nielsen, Google Analytics, Google Ads, Facebook, internal finance) is collected and transformed into a harmonized data warehouse. This stage handles cleaning, feature engineering, and alignment of all data to a weekly granularity.
- **Bayesian MMM Engine**: The core of the system where a hierarchical Bayesian model is trained on the historical data. The model is built using Python with libraries like PyMC.
- **Optimization & Simulation Module**: This component uses the trained model's posterior distributions to run "what-if" scenarios and find the optimal budget allocation that maximizes a given objective function (total revenue or ROI) under specified constraints.

## Dataset

This project uses a time-series dataset of weekly sales and marketing activities, including own-brand activities, competitor actions, and control variables.

### Features

| Primary Category | Secondary Category  | Specific Features                                       |
| :--------------- | :------------------ | :------------------------------------------------------ |
| **Media** | Traditional Media   | TV, OOH, Radio, Print                                   |
| **Media** | Digital Media       | YouTube, VOD, Digital Display, Social Media, Search     |
| **Non-Media** | Own Drivers         | Consumer promo, sampling, trade promo                   |
| **Non-Media** | Own Drivers         | Price, Distribution                                     |
| **Non-Media** | Others              | External factors, trends, seasonality                   |
| **Non-Media** | Others              | Competitive price, distribution, trade promotions & media |

`sales_performance`is the target variable.

## Modeling

The core of the analysis is a Bayesian regression model that estimates the contribution of each sales driver.

### Bayesian Framework
Unlike traditional frequentist methods, this approach yields a full probability distribution for each parameter. This allows us to quantify uncertainty, for example, by stating there is a "95% probability that the ROI for YouTube is between 2.1 and 2.8."

### Adstock Transformation
To capture the lagging effect of advertising, media variables are transformed using a geometric decay function. The model learns the optimal decay rate (memory effect) from the data, which indicates how long advertising's impact lingers.

$adstock_t = x_t + \theta \cdot adstock_{t-1}$

### Saturation Transformation
To model diminishing returns, the adstocked media variables are passed through a Hill function. This S-shaped curve ensures that the incremental sales impact of an additional dollar of advertising spend decreases as the total spend increases.

$Hill(x) = \beta \cdot \frac{x^\alpha}{x^\alpha + K^\alpha}$

### Model Specs
The final model takes the following general form:

$$
\text{Sales}_t = \mu + \sum_{i=1}^{n} \text{Hill}_i(\text{Adstock}_i(\text{media}_{it})) + \sum_{j=1}^{m} \gamma_j \cdot \text{control}_{jt} + \epsilon_t
$$

Where $\mu$ is the base sales, the first sum represents the contribution from various media channels after applying Adstock and Hill transformations, and the second sum captures the linear effect of control variables like price and distribution.

| Component | Description | Toolkit |
| :--- | :--- | :--- |
| **Bayesian Inference** | A hierarchical Bayesian model estimates the posterior distribution for all parameters, leveraging prior knowledge and capturing a full range of plausible values for each channel's effectiveness. | **`PyMC`** |
| **Flexible Transformations** | Media carryover (**Adstock**) and **Saturation** effects are modeled as flexible functions (e.g., Weibull, Hill) whose parameters are learned from the data. | **`Python`** |
| **Model Selection** | The **Bayesian Information Criterion (BIC)** is used to compare and select the most appropriate functional form specifications for the adstock and saturation effects, balancing model fit and complexity. | **`Python`** |
| **Budget Optimization** | The posterior distributions of channel ROIs are passed to a numerical optimizer. It solves for the budget allocation that maximizes the expected revenue, subject to a total budget constraint. | **`SciPy.optimize`** |

## Structure

The project repository is organized into the following directories to ensure reproducibility and maintainability:

```bash
├── assets/
├── data/
│   ├── raw/          # Raw data from various sources
│   └── processed/    # Cleaned and transformed modeling data
├── notebooks/
│   ├── 01-eda.ipynb
│   ├── 02-feature-engineering.ipynb
│   └── 03-modeling.ipynb
├── reports/
│   └── figures/      # Charts for presentations (e.g., saturation curves)
├── src/
│   ├── data.py       # Data processing and pipeline scripts
│   ├── model.py      # Modeling functions and classes
│   └── viz.py        # Visualization functions
└── app/
    └── dashboard.py  # Code for the interactive Streamlit dashboard
```

</br>

> [!WARNING]
> This repository provides a high-level demonstration of the project's architecture and methodology. Certain implementation details and model complexities have been simplified for clarity.

</br>

<p align="center">🌐 © 2025 t.r.</p>
