# Reflection

Name: Winit Pongsripian

Answer each question in 2–3 sentences.

**1. Which antipatterns from the lecture did `train_all.py` show? Point to the lines.**

> train_all.py is a monolithic, notebook-style script that mixes data loading, cleaning, feature encoding, training, evaluation, and saving in one manually run file (lines 14–45), making individual stages impossible to test or reuse. It also contains a dead experimental code path (USE_LOG_PRICE on lines 27–31) and hardcoded configuration/magic values—such as the fixed file path "data/sales_2025.csv" (line 15), the rigid date format "%Y-%m-%d" (line 19), and max_depth=8 (line 35). Finally, it saves the model unconditionally without any automated tests or evaluation gate (lines 38–45).

**2. Suppose there were no test and no evaluation gate. What would have happened when the 2026 batch arrived? Think about both the crash and the missing `sqft` values.**

> First, the new ISO timestamp format in data/sales_2026_q1.csv would have immediately crashed pd.to_datetime when someone tried to retrain, halting model updates until manually debugged. Even after fixing the date parsing, a decision tree silently accepts NaN values without raising an error, so the rows with missing sqft would have quietly flowed into training. Without tests enforcing data-cleaning guarantees or an MAE evaluation gate to block bad models, a silently degraded model could be saved and shipped to the marketplace unnoticed.

**3. In the real marketplace, which non-ML parts of the system should trigger this pipeline or feed it data?**

> The marketplace's sales transaction database and property listing service (via a data warehouse or ETL export job) should feed newly closed sales records into the collect stage. The pipeline should be triggered automatically by a time-based scheduler (e.g., weekly or monthly cron/Airflow runs), an event from the data ingestion system when a new sales batch arrives, or CI/CD whenever engineers update pipeline code. Once a new model passes the evaluation gate, the deployment/serving infrastructure (model registry and fair-value prediction API) should pick up the new artifact to update live home estimates.
