# 🛒 Instacart Reorder Prediction (Snowflake + dbt + ML + Streamlit)

This project builds a complete **AI-powered reorder prediction pipeline** on **Snowflake**, transforming raw transaction data into modeled reorder scores and surfacing them through a rich Snowsight dashboard.

It uses **dbt** for modular feature engineering, trains a **logistic regression model** locally using `scikit-learn`, and exports real predictions into Snowflake. The results are visualized interactively using native Snowflake tools — demonstrating what a productized ML workflow can look like inside the modern data stack. A local Streamlit app is also included for browsing predictions by user and product, adjusting thresholds, and uploading CSVs for batch inference.

---

## 💪🏼 What This Model Can Do
The logistic regression model predicts the probability that a user will reorder a given product in their next Instacart order. This enables a variety of high-impact use cases:

- **Personalized Recommendations**

Rank products by reorder probability to suggest top items for each user’s next basket.
- **Marketing Targeting**

Identify users with high reorder intent and send timely nudges or promotions.
- **Operational Forecasting**

Use aggregate reorder probabilities to improve inventory planning and fulfillment accuracy.
- **Analytics & Dashboards**

Visualize reorder likelihood across users, products, or departments to uncover actionable patterns.

---

## 📊 Live Snowflake Dashboard

[View Dashboard (Snowsight)](https://app.snowflake.com/kctrqzo/wdb83228/#/instacart-reorder-prediction-dLPdne9M0)  
_(Snowflake login required)_

This dashboard includes:

- Model-wide KPIs (volume, avg score, % above threshold)
- Score distribution for all predictions
- Predicted reorder rate by hour of day
- Top users, products, and aisles by predicted reorder probability
- Labeled risk bands (🟥🟨🟩) for easily surfacing low-confidence product categories

---

## ⚙️ Model Summary

| Metric                  | Value      |
|-------------------------|------------|
| Total Predictions       | 7,922,443  |
| Avg Predicted Score     | 0.102      |
| % Above 0.5 Threshold   | 0.6%       |

> The model was trained locally in Jupyter using `scikit-learn`, exported via `.pkl`, and used to generate predictions for all user-product pairs. These predictions were then uploaded to Snowflake and visualized through Snowsight.

---

## 🧱 Project Stack & Tools

| Component        | Tool Used                                |
|------------------|--------------------------------------------|
| Data Warehouse   | **Snowflake** (`INSTACART_DB`)             |
| Modeling         | **dbt** (SQL transformations)              |
| ML Training      | **scikit-learn** (logistic regression)     |
| Inference        | Local batch scoring (`.pkl` model)         |
| Dashboard        | **Snowsight** (Snowflake-native UI)        |
| App Interface    | **Streamlit** (local UI for predictions & exploration) |

---

## 🧠 Full AI Pipeline

- **Feature Engineering (dbt):**  
  Raw order and product data transformed into ML-ready features using reusable models

- **Model Training (Jupyter):**  
  Trained logistic regression locally on 7.9M user-product pairs  
  → **Notebook**: `model_training/instacart_train_model.ipynb`

- **Model Export:**  
  Exported via `joblib` into `instacart_model.pkl`

- **Inference Results:**  
  Predictions uploaded into Snowflake as `RAW.PREDICTED_REORDERS2`

- **Dashboard View:**  
  `RAW.instacart_predictions_output` used to unify scoring logic and power dashboard

- **Streamlit App:**  
  Local UI for browsing predictions, visualizing features, and running batch inference

---

## 💡 Key Insights
Most predictions score well below 0.5 — a calibrated model that's cautious by design

Only 0.6% of user-product pairs exceed the reorder threshold

Hour-of-day predictions peak between 10AM–4PM

"At-risk" aisles like frozen foods and soft drinks receive high volume but low reorder confidence

Milk and yogurt categories show consistently high predicted reorders

---

## 📌 How to Run This Project
Step 1: Run dbt transformations

Step 2: Train model locally using sklearn + Jupyter
         → export as instacart_model.pkl

Step 3: Upload predictions to Snowflake via UI (Snowsight stage)

Step 4: Create instacart_predictions_output view

Step 5: Explore live metrics in Snowsight dashboard

Step 6: Launch the local Streamlit app to browse predictions by user
```bash
streamlit run streamlit_app.py
```

---

## 🗂️ Raw Dataset Source

This project uses the open-source **Instacart Online Grocery Shopping Dataset 2017**, which includes:

- Over **3 million grocery orders** from ~200,000 users  
- **50,000+ unique products** across **130 aisles**  
- Complete **user–product order history**  
- Metadata for products, departments, and aisles  

The data was ingested into **Snowflake** via a user-created stage (`instacart_stage`) using standard `COPY INTO` commands.

📦 **Dataset URL**  
[Kaggle: Instacart Online Grocery Shopping Dataset](https://www.kaggle.com/datasets/yasserh/instacart-online-grocery-basket-analysis-dataset)

---

### ⚠️ Large Files Not Included in This Repo

GitHub limits file uploads to 100MB, so the following large files are **excluded**:

| File                         | Approx. Size |
|------------------------------|---------------|
| `predicted_reorders.csv`     | ~250MB        |
| `order_products__prior.csv`  | ~180MB        |
| `instacart_training_input.csv` | ~260MB       | Exported from Snowflake after running dbt |

### 📥 Setup Instructions

1. Download and unzip the dataset from Kaggle.

2. Place the following files in your local `original_files/` directory:

```plaintext
original_files/
├── aisles.csv
├── departments.csv
├── order_products__prior.csv
├── order_products__train.csv
├── orders.csv
├── products.csv
├── instacart_training_input.csv
```

### 📥 How to Generate `instacart_training_input.csv`

1. Ensure your dbt environment is configured and connected to Snowflake

2. Run the model using:
```bash
dbt run --select instacart_training_input
```
3. In [Snowsight](https://app.snowflake.com), run:
```sql
SELECT * FROM RAW.instacart_training_input;
```
4. Use the download icon in Snowsight to export the result as CSV

5. Save it to:
```
original_files/instacart_training_input.csv
```
Once saved, the Streamlit app will work as expected.

---

## 🔮 Future Product Ideas Inspired by Project Experience

While building this project, I identified an opportunity to further enhance the Snowflake developer experience:

### [Context-Aware SQL Copilot for Feature Engineering](https://docs.google.com/document/d/1Z-HQx90oDcu1zacMwie6zvGnZomQS-slF4m3_k7wrlI)
A built-in assistant within SQL worksheets that suggests and auto-generates feature engineering queries based on a project’s schema.  
Example prompts:
- *"Would you like to add a rolling 3-order average?"*
- *"Would you like to calculate reorder rates per user?"*

**Impact:**  
Accelerates ML preparation, lowers SQL barriers, and bridges data warehousing with AI workflows.

---

## 📁 Project Structure

```plaintext
instacart-reorder-prediction/
├── assets/
│   └── dashboard_preview.png                  # Snowsight dashboard screenshot
│
├── docs/                                      # Documentation and product design artifacts
│   └── one-pager.md                           # One-pager: Context-Aware SQL Copilot for Snowsight
│
├── models/                                    # dbt models for feature engineering & ML prep
│   ├── dim_orders.sql
│   ├── dim_products.sql
│   ├── fct_orders.sql
│   ├── fct_reorder_training_data.sql
│   ├── fct_user_product_features.sql
│   ├── instacart_orders.sql
│   ├── instacart_predictions_output.sql
│   ├── instacart_training_input.sql
│   └── schema.yml
│
├── notebooks/
│   └── Instacart.ipynb                        # Jupyter notebook for local model training
│
├── original_files/                            # Raw dataset CSVs from Kaggle
│   ├── aisles.csv
│   ├── departments.csv
│   ├── order_products__train.csv
│   ├── predicted_reorders.csv
│   └── products.csv
│
├── snowflake_sql/                             # Snowflake SQL scripts for pipeline setup
│   ├── 01_ingest_instacart_data.sql           # Stage and load raw CSVs
│   ├── 02_dbt_model_run.sql                   # Run dbt transformations
│   ├── 03_model_upload_and_udf.sql            # (Optional) Create UDFs from model
│   ├── 04_local_predictions_to_table.sql      # Upload predictions to Snowflake
│   └── 05_model_features_and_dummy_output.sql # Final view logic and risk labeling
│
├── snowpark/                                  # Local ML logic (Snowpark-ready structure)
│   ├── local_train.py
│   └── model_training.py
│
├── streamlit_app.py                           # Public-facing UI for prediction browsing
├── instacart_model.pkl                        # Exported logistic regression model
├── dbt_project.yml                            # dbt configuration
├── .env.example                               # Sample env config for Snowflake/Streamlit
├── .gitignore
└── README.md                                  # Project documentation (this file)
```
---

## 🙏 Supporting Materials

- [One-Pager: Context-Aware SQL Copilot for Snowsight](docs/one-pager.md)

---

## 📌 GitHub Metadata

- 🧑‍💻 Author: [Justin Borenstein-Lawee](https://www.linkedin.com/in/justin-borenstein-lawee/)  
- 🕓 Last Updated: April 2025  
