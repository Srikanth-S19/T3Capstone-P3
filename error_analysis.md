

By examining **False Positives (FPs)** - where budget was wasted by flagging loyal customers - and **False Negatives (FNs)** - where critical churners were missed, systemic model blind spots can be identified.



The analysis below uses a **0.35 probability threshold**, which prioritizes capturing churners (Recall) over perfect precision.

### 1. False Positive Analysis (Model Predicted Churn, User Stayed)

These users were flagged as "at-risk" but remained active. These errors represent **"wasted" retention incentives**, though they often act as successful surprise-and-delight brand touchpoints.

| **Customer ID** | **Pred. Prob** | **Recency** | **Freq (180d)** | **Sessions (30d)** | **Analysis**                                                                 |
| --------------- | -------------- | ----------- | --------------- | ------------------ | ---------------------------------------------------------------------------- |
| **CUST00007**   | $0.41$         | $3$ days    | $1$             | $11$               | High recent activity ($11$ sessions) masked by low historical frequency.     |
| **CUST00008**   | $0.43$         | $47$ days   | $3$             | $2$                | Just crossed the $45$-day recency threshold, triggering a false alarm.       |
| **CUST00017**   | $0.38$         | $286$ days  | $0$             | $4$                | Historically dormant user; model is sensitive to their sudden $4$ re-visits. |
| **CUST00018**   | $0.52$         | $111$ days  | $1$             | $3$                | Low engagement history skewed the model toward a high-risk prediction.       |
| **CUST00038**   | $0.51$         | $51$ days   | $3$             | $5$                | Frequent buyer who simply had a slightly longer inter-purchase interval.     |

**Common Root Cause:** The model is highly sensitive to **Recency thresholds** (specifically crossing the $45$-day mark). Many FPs are high-frequency, loyal users who simply experienced a natural "breather" in their purchase cycle but remained digitally engaged.



### 2. False Negative Analysis (Model Predicted Stay, User Churned)

These users were classified as "secure," yet they left the platform. These errors represent **lost lifetime value (LTV)** and missed revenue opportunities.

| **Customer ID** | **Pred. Prob** | **Recency** | **Freq (180d)** | **Sessions (30d)** | **Analysis**                                                                   |
| --------------- | -------------- | ----------- | --------------- | ------------------ | ------------------------------------------------------------------------------ |
| **CUST00033**   | $0.07$         | $12$ days   | $3$             | $7$                | **"The Sudden Defector"**: Extremely high engagement until abrupt exit.        |
| **CUST00037**   | $0.34$         | $102$ days  | $1$             | $5$                | Just missed the $0.35$ threshold; marginal error in probability calibration.   |
| **CUST00079**   | $0.25$         | $191$ days  | $0$             | $2$                | Low-value, long-term dormant; model failed to detect intent to finalize churn. |
| **CUST00087**   | $0.30$         | $44$ days   | $1$             | $1$                | Almost exactly at the threshold; failed to account for lack of intent.         |
| **CUST00117**   | $0.32$         | $123$ days  | $2$             | $2$                | Stagnant buyer with no recent digital recovery signals.                        |

**Common Root Cause:** The model struggles with **"Sudden Defectors"** (e.g., `CUST00033`), who exhibit high engagement right up until a catastrophic event—such as a bad product reaction or a failed delivery—causes them to exit permanently.



### 3. Recommendations for Model Refinement

Based on this analysis, we recommend the following three adjustments to the production pipeline:

- **Introduce "Velocity" Features:** The model currently uses static snapshots (e.g., sessions in last $30$ days). We must introduce **$\Delta$ (delta) features**—such as the change in session count week-over-week—to catch "Sudden Defectors" whose engagement is trending downward rapidly, even if the absolute value is still high.

- **Segment-Specific Thresholds:** Instead of a global $0.35$ threshold, implement **tiered probability cutoffs**. For high-frequency "Champions" (like `CUST00033`), lower the intervention threshold to $0.20$ to ensure even small dips in behavior trigger a "check-in" interaction.

- **Integrate Sentiment Features:** False Negatives like `CUST00033` are often driven by unresolved negative experiences. We must weight `avg_resolution_hours_90d` and `negative_ticket_rate` more heavily in the XGBoost gain criteria to override positive historical transactional patterns.



### **Summary of Error Impact:**

> The model is excellent at identifying "low-hanging fruit" (habitual users with slipping engagement) but is "blind" to high-value users who experience a single, decisive negative shock. 
> 
> By adding velocity-based features and lowering thresholds for most valuable segments, we can catch these high-impact False Negatives without significantly increasing our False Positive budget waste.




