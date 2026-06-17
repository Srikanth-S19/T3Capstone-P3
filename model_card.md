# Model Card: Customer Churn Prediction Engine

This model card provides technical, operational, and ethical documentation for the customer churn classification models engineered for the Churn Project. It outlines the scope, data constraints, performance targets, and deployment guardrails required to run the predictive system safely in production.

### 1. Model Details

- **Developed By:** Customer Data Science & Analytics Division

- **Model Date:** June 2026

- **Model Version:** $v1.0.0$

- **Model Type:** Supervised Binary Classification (Ensemble & Parametric)

- **Primary Architectures:** XGBoost Classifier (Champion Framework), Random Forest Classifier (Precision-Constrained Variant), and Logistic Regression (Baseline Benchmark).

### 2. Intended Use

- **Primary Objective:** To predict the probability that an individual active customer will churn (i.e., fail to complete a transaction) within a $60$-day lookforward window following a fixed operational snapshot date.

- **Target Audience:** Marketing Ops, Customer Experience (CX) Leads, Customer Support Concierge Desks, and Financial Planning teams.

- **Downstream Action:** Automating real-time, low-cost marketing triggers (e.g., cart-recovery emails, product replenishment offers) and flagging high-value, high-friction accounts for manual support intervention via specialized VIP concierge queues.

### 3. Data Used

The training matrix represents a multi-source fusion of transactional, operational, demographic, and digital lookback behaviors aggregated into a point-in-time snapshot.

- **Raw Components:** `customers.csv`, `orders.csv`, `support_tickets.csv`, `web_events_snapshot.csv`, and `intervention_history.csv`.

- **Lookback Temporal Boundaries:** All feature variables represent historical aggregations compiled exclusively up to **September 30, 2025**.

- **Data Leakage Control Strategy:** To prevent predictive invalidation, a strict temporal wall was applied at the snapshot date. Transactions occurring in the future target window (comprising $1,872$ post-snapshot orders between October 1, 2025, and November 29, 2025) were explicitly excluded from feature generation and used solely to construct the ground-truth churn labels.

- **Dataset Partitions & Sample Dimensions:**
  
  - **Training Set ($72.0\%$):** $1,728$ rows (balanced stratification: $916$ retained, $812$ churned)
  
  - **Validation Set ($14.0\%$):** $336$ rows ($189$ retained, $147$ churned)
  
  - **Test Set ($14.0\%$):** $336$ rows (exactly balanced: $168$ retained, $168$ churned)

### 4. Model Approach

- **Data Preprocessing Pipeline:** Categorical attributes (such as `city_tier`, `age_group`, `acquisition_channel`, `loyalty_tier`, and `preferred_category`) were transformed using scikit-learn’s `OneHotEncoder(handle_unknown='ignore')`, expanding the feature space to $44$ sparse dimensions. Missing values in the loyalty column were mapped to an explicit `'None'` category to represent standard non-loyalty members. Numerical variables were standardized to zero mean and unit variance ($\mu = 0, \sigma = 1$) via `StandardScaler`.

- **Training Execution:** Pipeline scaling metrics were computed **exclusively from the training partition** to eliminate out-of-sample data leakage.

- **Decision Optimization:** Rather than defaulting to a standard mathematical cutoff of $0.50$, the production deployment implements an operationally optimized decision threshold of **$\pi = 0.35$**. This custom probability boundary aligns the statistical models with the company’s strict $\$15,000$ retention campaign budget, capturing maximum churn risk while minimizing opportunity loss.

### 5. Performance Metrics

Models were evaluated on an out-of-sample test set ($N = 336$ customer profiles). Because the test partition is perfectly balanced ($50/50$ class split), accuracy provides an un-skewed indicator of baseline classification performance.

#### Out-of-Sample Performance Table (Test Split)

| **Evaluation Metric**    | **Logistic Regression (Baseline)** | **Random Forest (Precision Variant)** | **XGBoost Classifier (Champion Frame)** |
| ------------------------ | ---------------------------------- | ------------------------------------- | --------------------------------------- |
| **Accuracy**             | $79.17\%$                          | $79.17\%$                             | **$80.06\%$**                           |
| **Precision**            | $79.88\%$                          | **$82.24\%$**                         | $79.88\%$                               |
| **Recall (Sensitivity)** | $77.98\%$                          | $74.40\%$                             | **$80.36\%$**                           |
| **F1-Score**             | $78.92\%$                          | $78.13\%$                             | **$80.12\%$**                           |
| **ROC-AUC**              | $0.8848$                           | **$0.8865$**                          | $0.8686$                                |

```
  Receiver Operating Characteristic (ROC) Metrics Overview
  ├── Random Forest   : [██████████████████████████████] AUC = 0.887
  ├── Logistic Reg.   : [████████████████████████████]   AUC = 0.885
  └── XGBoost Engine  : [██████████████████████████]     AUC = 0.869
```

#### Selection Justification

The **XGBoost Classifier** is designated as the production deployment framework because it achieves the highest overall **Recall ($80.36\%$)** and **F1-Score ($80.12\%$)**. It successfully flags the highest absolute surface area of at-risk users, preventing catastrophic retention slips. However, under high capital constraints, the **Random Forest** model can be substituted to capitalize on its superior **Precision ($82.24\%$)**, ensuring minimal incentive spend waste on false positives.

### 6. Limitations

- **Static Horizon View:** The model is bounded by a fixed $60$-day prediction timeline. It cannot accurately forecast rapid consumer drop-offs that occur inside a shorter $14$-day window, nor can it model long-tail macro attrition beyond $60$ days.

- **Sensitivity to "Shock Churn":** As uncovered during manual error analysis (e.g., `CUST00184`), the framework suffers from structural blind spots when high-value, highly frequent "Core Champions" defect suddenly without showing previous digital drop-off signals. This typically stems from external, unobserved shocks like competitive pricing attacks or major un-logged service failures.

- **Cold-Start Vulnerability:** The model requires a minimum historical transaction baseline to extract valid RFM attributes. It cannot generate reliable predictions for fresh onboards whose customer lifespan is under $7$ days.

### 7. Ethical Risks & Fairness Considerations

- **Algorithmic Price Discrimination:** If probability predictions are used to dynamically alter retail pricing or selectively issue discount vouchers, high-loyalty groups may be systemically forced to pay a premium ("loyalty penalty"), while highly volatile shoppers receive continuous discounts. This risks alienating your most profitable customer base.

- **Demographic Bias:** One-hot encodings track location metadata (`city_tier`) and age classifications (`age_group`). The model must be continuously audited using demographic parity checks to ensure it does not limit access to premium tiers or restrict customer service support levels based on protected personal attributes.

- **Consent and Privacy:** Features rely on web and application activity snapshots (`web_events_snapshot.csv`). The data pipeline must strictly comply with regional data privacy laws, automatically stripping behavior tracking variables for any customer profile that revokes their marketing consent (`marketing_consent == 'No'`).

### 8. Monitoring Needs

- **Data Drift Auditing:** Monthly validation routines must track statistical drift in key numerical features—specifically `recency_days` and `product_views_30d`—using Population Stability Index ($PSI$) metrics. Sudden shifts in baseline digital interactions will degrade model performance over time.

- **Concept Drift and Macro Dynamics:** The relationship between digital engagement and churn behavior will evolve alongside changes to the application UI or shifting economic conditions. The F1-score performance must be monitored continuously against real-world outcomes; any sustained drop below **$75\%$** should automatically trigger a comprehensive model retraining pipeline.

- **Operational Friction Overlap:** Support ticket characteristics must be monitored weekly. If the resolution times for delayed refunds (`refund_delay`) or cosmetic issues (`product_reaction`) spike past the current $36$-hour bottleneck, feature weights must be recalibrated to account for the increased impact of support friction on customer churn.

### 9. Out-of-Scope / When the Model Should Not Be Used

- **Product Quality Auditing:** This system is an engagement and transaction predictor. It should never be used as a primary metric to evaluate raw product safety or cosmetic formula performance, which requires specialized quality control testing.

- **Broader Corporate Financial Projections:** While the model provides valuable operational risk alerts, its output probabilities should not be used as direct cash flow projections or formal corporate revenue guidance.

- **Cross-Brand Extrapolation:** The behavioral relationships captured in this model are highly specific to the cosmetics, skin, and hair care categories present in this dataset. Running this model against external product lines or entirely different retail brands will produce invalid risk classifications and lead to suboptimal marketing spend.
