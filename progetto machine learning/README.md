# Stock Price Direction Prediction

Final Machine Learning project by **Riccardo Striano**.

This project compares classification algorithms to predict whether the closing price of **Apple (AAPL)** will increase in the next trading session. The notebook covers data collection, feature engineering, model training, and evaluation.

## Project Files

| File | Description |
| --- | --- |
| [Progetto Finale Machine Learning.ipynb](Progetto%20Finale%20Machine%20Learning.ipynb) | Python notebook containing code, explanations, charts, and saved execution results. |
| [Presentazione Progetto Machine Learning.pptx](Presentazione%20Progetto%20Machine%20Learning.pptx) | Slides used for the oral presentation. |
| README.md | Project description and instructions for running the analysis. |

## Objective and Dataset

The task is **binary classification**:

- **1**: the next trading session's closing price is higher than the current closing price.
- **0**: the next trading session's closing price is lower than or equal to the current closing price.

The notebook downloads daily data from **Yahoo Finance** using the `yfinance` library:

```python
yf.download("AAPL", start="2018-01-01", end="2024-01-01")
```

The saved execution contains **1,509 observations**, from **January 2, 2018, to December 29, 2023**. The download's end date is January 1, 2024, so the dataset does not include any trading sessions from 2024.

The original variables are the opening, high, low, and closing prices, along with trading volume.

## Data Preparation

The notebook calculates the following indicators:

| Indicator | Definition | Usage |
| --- | --- | --- |
| Return | Daily percentage change in the closing price. | Used to calculate volatility. |
| MA10 | Moving average of the closing price over 10 trading sessions. | Model input. |
| MA50 | Moving average of the closing price over 50 trading sessions. | Model input. |
| Volatility | Standard deviation of daily returns over 10 trading sessions. | Model input. |
| Momentum | Difference between the current closing price and the closing price 5 trading sessions earlier. | Model input. |

The **9 features used as model inputs** are `Open`, `High`, `Low`, `Close`, `Volume`, `MA10`, `MA50`, `Volatility`, and `Momentum`. Daily return is not included directly in the input matrix.

After removing missing values, the code splits the observations chronologically:

- The **first 80%** is used for training.
- The **last 20%** is used for testing, corresponding to **292 observations** in the saved execution.

For Logistic Regression and SVM, `StandardScaler` is fitted only on the training set and then applied to both the training and test sets.

## Models Compared

| Approach | Configuration |
| --- | --- |
| Baseline | Always predicts a price increase, corresponding to class 1. |
| Logistic Regression | Standardized features and `max_iter=1000`. |
| Random Forest | 100 decision trees and `random_state=42`. |
| Support Vector Machine | RBF kernel, standardized features, and `random_state=42`. |

## Results

The table reports the **outputs saved in the notebook**, without rerunning the analysis.

| Approach | Test Accuracy | Macro F1 |
| --- | ---: | ---: |
| Baseline | 53.77% | 0.35 |
| **Logistic Regression** | **55.14%** | **0.55** |
| Random Forest | 52.05% | 0.50 |
| SVM | 50.00% | 0.47 |

Macro F1 scores are rounded to two decimal places, as in the notebook's classification reports.

**Logistic Regression achieves the highest accuracy**, with **161 correct predictions out of 292**, exceeding the baseline by approximately **1.37 percentage points**. This is a small improvement and, on its own, does not establish consistent predictive performance across different periods.

The test set contains 157 class 1 observations and 135 class 0 observations. The baseline benefits from the higher frequency of price increases but fails to identify any class 0 observations. Logistic Regression performs more evenly across the two classes. Random Forest and SVM identify a larger proportion of class 0 observations but miss many price increases.

The notebook also includes:

- A chart comparing model accuracy.
- Confusion matrices and classification reports.
- Random Forest predictions plotted alongside the stock price.
- A comparison of actual and predicted price direction.
- Random Forest feature importance.

In the saved output, the features with the highest Random Forest importance scores are **Momentum**, **Volatility**, and **Volume**. These scores describe how the model uses the variables and do not establish causal relationships.

## Running the Notebook

A Python environment with JupyterLab and an Internet connection are required to download the data.

1. Download the notebook or clone the repository.
2. Install JupyterLab and the required libraries:

   ```bash
   pip install jupyterlab yfinance pandas numpy matplotlib scikit-learn
   ```

3. Open a terminal in the project folder and launch:

   ```bash
   jupyter lab
   ```

4. Open `Progetto Finale Machine Learning.ipynb` and select a Python kernel with the required libraries installed.
5. Run the cells in their original order.

The notebook's first code cell also contains a command to install **TensorFlow**, which is not used by the models in this analysis. That installation line can be skipped when running this project.

For JupyterLab installation instructions, see the [official Jupyter documentation](https://jupyter.org/install).

## Limitations and Future Work

The analysis covers a single stock and a single chronological train/test split. The code does not include systematic hyperparameter tuning, walk-forward validation, or a trading strategy backtest. Downloaded data and library versions may change, so rerunning the notebook may produce different results.

**Target construction note:** in the attached version, the comparison using `shift(-1)` also assigns 0 to the final row, for which the next session's price is unavailable. This row should be excluded before a new evaluation. The results above describe the saved outputs of the current version.

Future work could include validation across multiple time windows, model tuning using only training and validation data, evaluation on additional stocks, and the inclusion of external variables.

