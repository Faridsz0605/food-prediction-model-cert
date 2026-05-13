<p align="center">
  <img src="assets/readme-banner.svg" alt="Tasty Bytes Recipe Traffic Prediction banner" width="100%" />
</p>

<p align="center">
  <a href="food-prediction-model.ipynb"><img alt="Notebook" src="https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white"></a>
  <img alt="Python" src="https://img.shields.io/badge/Python-3.13.5-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="pandas" src="https://img.shields.io/badge/pandas-Data%20Validation-150458?style=for-the-badge&logo=pandas&logoColor=white">
  <img alt="scikit-learn" src="https://img.shields.io/badge/scikit--learn-Modeling-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white">
  <img alt="Status" src="https://img.shields.io/badge/status-Analysis%20Complete-22C55E?style=for-the-badge">
</p>

# Food Prediction Model

A GitHub-ready data science report for **Tasty Bytes**, focused on predicting whether a recipe featured on the homepage will generate **high traffic**.

The analysis lives in [`food-prediction-model.ipynb`](food-prediction-model.ipynb) and follows the practical exam brief in [`project-instructions.md`](project-instructions.md): validate the data, explore drivers of traffic, train baseline and comparison models, evaluate them, and recommend a business monitoring metric.

> [!NOTE]
> The notebook expects `recipe_site_traffic_2212.csv` in the project root. The CSV is not included in this repository snapshot.

## Project objective

The product team wants to select homepage recipes that are popular at least **80% of the time** while minimizing unpopular recommendations.

This notebook frames the task as a **supervised binary classification** problem:

| Label | Meaning |
| --- | --- |
| `1` | `high_traffic == "High"` |
| `0` | missing / null `high_traffic` |

## What the notebook does

- Validates all expected columns: `recipe`, nutrients, `category`, `servings`, and `high_traffic`.
- Cleans recipe categories, including `Chicken Breast` → `Chicken`.
- Extracts numeric servings and preserves snack-specific serving labels with `is_snack_serving`.
- Imputes missing nutritional values by category median.
- Compares three classifiers:
  - `DummyClassifier` reference model
  - Logistic Regression baseline
  - Random Forest comparison model
- Selects decision thresholds using a precision-first rule with a **Wilson 95% lower confidence bound**.
- Defines business metrics: recommendation coverage, precision lift, false positives per 100 recommendations, and net value.

## Key findings

Dataset summary from the notebook:

| Metric | Value |
| --- | ---: |
| Rows | 947 |
| Columns | 8 |
| High traffic recipes | 574 |
| Not high traffic recipes | 373 |
| Nutrient rows with missing values | 52 |
| Clean recipe categories | 10 |

Exploratory analysis found that recipe **category** is one of the strongest available signals. Nutritional variables have weak linear correlation with the target, which motivated comparing a linear model against a non-linear ensemble.

## Model results

Final test-set comparison:

| Model | Threshold | Recommended | Coverage | Precision | Wilson precision L95 | Recall | F1 | Avg. precision |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Logistic Regression baseline | 0.65 | 49 | 25.8% | **100.0%** | **92.7%** | 42.6% | 0.598 | **0.919** |
| Random Forest comparison | 0.75 | 40 | 21.1% | 97.5% | 87.1% | 33.9% | 0.503 | 0.891 |
| Dummy reference | 0.01 | 190 | 100.0% | 60.5% | 53.4% | 100.0% | 0.754 | 0.605 |

The selected model is **Logistic Regression** because it satisfies the 80% precision requirement with statistical confidence and recommends more recipes than the Random Forest under the same precision-first framing.

## Business recommendation

Use the Logistic Regression pipeline as the first production candidate for homepage recipe selection.

| Business metric | Logistic Regression result |
| --- | ---: |
| Recommended recipes in test | 49 |
| High recipes per 100 recommendations | 100.0 |
| False positives per 100 recommendations | 0.0 |
| Precision lift vs. base rate | 1.65× |
| Net value per 100 recommendations | 100.0 |
| Passes business rule | Yes |

Recommended monitoring metric: **daily precision of recommended recipes**, reported with a Wilson confidence interval so the team does not overreact to small sample sizes.

## Repository structure

```text
.
├── assets/
│   └── readme-banner.svg
├── food-prediction-model.ipynb
├── project-instructions.md
└── README.md
```

## Run locally

1. Place `recipe_site_traffic_2212.csv` in the repository root.
2. Create an environment and install the notebook dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install jupyter numpy pandas matplotlib scikit-learn
```

3. Start Jupyter and open the analysis:

```bash
jupyter notebook food-prediction-model.ipynb
```

## Main recommendation

Deploy the Logistic Regression approach as a conservative first version, keep `recipe` only for traceability, and recalibrate the decision threshold periodically to maintain precision ≥80% without unnecessarily shrinking recommendation coverage.
