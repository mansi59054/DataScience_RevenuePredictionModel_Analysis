# ABC Retail Revenue Prediction

Predicting ABC-Retail's quarterly revenue from sparse daily order counts and transaction spend indices, using a transparent Multiple Linear Regression pipeline built end-to-end in Python.

## Overview

This project was built as a data screening exercise. It takes three raw, messy signals — cumulative daily order numbers, a daily transaction spend index, and quarterly reported revenue — and turns them into a working forecasting model that explains ~92% of historical revenue variance and predicts an out-of-sample quarter within ~11% of the actual value.

## Objective

Estimate ABC-Retail's quarterly revenue using two leading indicators:
- Daily order volume (reconstructed from cumulative, gap-filled data)
- Daily transaction spend index (normalized for panel/user-base growth)

## Data

Input: `data_task.xlsx`, containing three sheets:
| Sheet | Contents | Rows |
|---|---|---|
| Orders | Cumulative daily order numbers (sparse, non-sequential) | 856 |
| Transactions | Daily total spend index + weekly active users index | 1,826 |
| Reported | Quarterly reported revenue index (2018 Q1 – 2022 Q4) | 20 |

## Methodology

### 1. Data Cleaning
- Standardized all column headers (lowercase, whitespace-stripped)
- Converted all date fields to datetime objects
- Deduplicated orders by date, keeping the last value per day

### 2. Feature Engineering
- **Cumulative max filter** (`cummax()`) to correct non-sequential/bad entries in cumulative order data
- **Linear interpolation** on a daily resample to fill gaps between known order data points
- **First-order differencing** (`.diff()`) to convert interpolated cumulative orders into estimated daily new orders
- **Spend normalization**: divided total spend index by weekly active users index, isolating true per-user spend intensity and removing growth-panel bias

### 3. Quarterly Aggregation
For each reported quarter:
- Summed daily estimated orders → total quarterly order volume
- Averaged normalized spend → quarterly consumer intensity
- Calculated days per period to account for variable quarter lengths (59–122 days)

### 4. Modeling
- **Model**: Multiple Linear Regression
- **Features**: `total_orders`, `avg_spend`, `days`
- **Target**: `revenue_index`
- **Train set**: 2018 Q1 – 2022 Q3 (19 quarters)
- **Test set (out-of-sample)**: 2022 Q4

### 5. Evaluation
Historical fit and predictive accuracy were assessed with R², MAE, RMSE, and out-of-sample percentage error, followed by visual diagnostics (actual vs. predicted chart, residual plot).

## Results

| Metric | Value |
|---|---|
| In-sample R² | 0.9189 |
| Historical MAE | 29.27 |
| Historical RMSE | 35.82 |
| 2022 Q4 Predicted Revenue | 454.43 |
| 2022 Q4 Actual Revenue | 512.08 |
| Out-of-sample % error | 11.26% |

**Model coefficients:**

| Feature | Weight |
|---|---|
| `total_orders` | -0.000008 |
| `avg_spend` | 374.831063 |
| `days` | 2.687150 |
| Intercept | -251.520539 |

## Visualizations

- **Actual vs. Predicted Revenue** (`actual_vs_predicted.png`) — tracks the model's fitted line against reported revenue across all 20 quarters, with the 2022 Q4 out-of-sample point highlighted
- **Residual Plot** (`residuals.png`) — checks that prediction errors are randomly scattered around zero, indicating no systematic bias across revenue levels

## Tech Stack

- Python 3
- pandas / numpy
- scikit-learn (`LinearRegression`, evaluation metrics)
- matplotlib

## Project Structure

```
.
├── data_task.xlsx              # Raw input data (3 sheets: orders, transactions, reported)
├── revenue_prediction.ipynb    # Full analysis notebook (cleaning → modeling → evaluation)
├── actual_vs_predicted.png     # Output chart
├── residuals.png               # Output chart
└── README.md
```

## How to Run

1. Clone the repo and open the notebook in Google Colab or Jupyter
2. Run cells sequentially — Step 1 will prompt for upload of `data_task.xlsx`
3. Outputs (metrics, coefficients, charts) print/save automatically at each step

```bash
pip install pandas numpy scikit-learn matplotlib openpyxl
```

## Key Assumptions & Limitations

- Linear interpolation assumes steady, constant growth between known order data points — it will smooth over any real short-term spikes or drops
- Spend normalization assumes user growth is the primary source of index inflation, not genuine demand growth
- A single blind quarter (2022 Q4) is a limited out-of-sample test; a rolling-window or k-fold time-series validation would give a more robust accuracy estimate
- Linear regression assumes a linear relationship between features and revenue; more complex dynamics (e.g. seasonality interactions) are not explicitly modeled

## Author

Mansi Od
