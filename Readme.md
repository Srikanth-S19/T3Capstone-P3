# Capstone Project: D2C Customer Churn Intelligence & Retention API

## Part 3 — Churn Prediction Model & Model Card

### Objective

The company wants a churn-prediction model that can identify customers likely to churn in the next 60 days. Your task is to build, evaluate, interpret, and document a model that can support retention decisions.

### Tasks

1. Prepare the modeling data using either:
   o    the provided modeling snapshot, or
   o    your own feature table created from the raw datasets.
2. Clearly separate train, validation, and test data using the provided split or a justified alternative.
3. Train at least two models:
   o    one simple baseline model,
   o    one stronger model such as Random Forest, Gradient Boosting, XGBoost, LightGBM, or another justified classifier.
4. Evaluate the model using metrics suitable for churn classification. Accuracy alone is not sufficient.
5. Select a decision threshold and justify it from a business perspective.
6. Perform error analysis using false positives and false negatives. Include at least 10 specific customer examples with your interpretation.
7. Explain the top features driving predictions.
8. Write a model card covering:
   o    intended use,
   o    data used,
   o    model approach,
   o    performance,
   o    limitations,
   o    ethical risks,
   o    monitoring needs,
   o    when the model should not be used.

### Execution Pre-Requisites :

1. Install Python along with Jupyter Notebook / VS Code or other software to read & execute ".ipynb" files, or use cloud-based programs such as Google Colab

2. Install additional python modules given in "requirements.txt"

3. Create sub-folder structure as below :
   
   ```
       ├───data
       └───images
   ```

4. Copy the files listed in "Input Data Files" section to "data" folder

### Input Data Files (Under "data" folder)

| File Name                     | Rows   | Description                                                   |
| ----------------------------- | ------ | ------------------------------------------------------------- |
| `customers.csv`               | 2,400  | Static customer profile and acquisition attributes            |
| `orders.csv`                  | 10,009 | Full order-level transaction history (pre- and post-snapshot) |
| `support_tickets.csv`         | 1,921  | Customer-service interactions                                 |
| `web_events_snapshot.csv`     | 2,400  | 30-day web/app activity as of snapshot date                   |
| `churn_labels.csv`            | 2,400  | Target variable and train/val/test split assignment           |
| `rfm_modeling_snapshot.csv`   | 2,400  | Pre-built, feature-engineered modeling table                  |
| `intervention_history.csv`    | 2,400  | Most recent campaign/intervention per customer                |
| `engineered_rfm_features.csv` | 2,400  | Generated in Part 2 - contains the new engineered features    |

### Program Execution

```
Run Program "churn_model.ipynb"
```

### Output Data Files (Under "data" folder)

| File Name      | Description                                                                                                                                                                            |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model.pkl`    | Trained model, saved for Part 4                                                                                                                                                        |
| `metrics.json` | file with <br/>a) key model metrics - accuracy, precision, recall, F1-score, ROC-AUC<br/>b) Confusion matrix values - at 0.5 (default) and threshold value<br/>c) Threshold value<br/> |

### Output Images (Under "images" folder)

| File Name                                   | Description                                                                                             |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `model_evaluation_metrics_comparison.png`   | ROC curves, F1 Score and ROC-AUC for the 3 models used (Logistic Regression, Random Forest and XGBoost) |
| `business_threshold_optimization_curve.png` | Graph showing the net value vs probability decision for threshold                                       |

### Documents

| File Name           | Description                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| `error_analysis.md` | False positive, false negative analysis. Also specific analysis of 10 customers |
| `model_card.md`     | A structured model card, explaining the model                                   |
