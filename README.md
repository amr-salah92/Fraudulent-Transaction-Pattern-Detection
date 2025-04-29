# Table of Contents

- [Project Name](#project-name)  
- [Project Background](#project-background)  
- [Project Goals](#project-goals)  
- [Insights and Recommendations are Provided in Detail on the Following Key Areas](#insights-and-recommendations-are-provided-in-detail-on-the-following-key-areas)  
  - [Category 1: Transaction Type Patterns](#category-1-transaction-type-patterns)  
  - [Category 2: Feature Engineering](#category-2-feature-engineering)  
  - [Category 3: Fraud Detection Trends](#category-3-fraud-detection-trends)  
  - [Category 4: Model Performance](#category-4-model-performance)  
- [Data Structure & Initial Checks](#data-structure--initial-checks)  
- [Executive Summary](#executive-summary)  
- [Insights Deep Dive](#insights-deep-dive)  
- [Recommendations](#recommendations)  
- [Technical Details](#technical-details)  
- [Assumptions and Caveats](#assumptions-and-caveats)  

---

## Project Name

**FinSecure Analytics – Fraudulent Transaction Pattern Detection**

---

## Project Background

FinSecure is a transaction monitoring company that has been operating in the digital financial services industry since 2013. The firm partners with banks, e-wallet services, and fintech platforms across North America and Southeast Asia to monitor suspicious activities in real-time financial transactions. As of Q4 2023, FinSecure processes over 4.2 million transactions per day from over 35 million user accounts. The core business model centers on deploying machine learning algorithms to flag high-risk transactions based on real-time data feeds. Key metrics include a 97.8% detection rate, a false-positive rate under 1.2%, and an average processing latency of under 1.5 seconds per transaction.

As a data analyst at FinSecure, I work directly with engineering and fraud investigation teams to develop fraud profiling strategies. This particular project was initiated in January 2024 using a sample of 6,362,620 historical transactions to evaluate anomalies and engineer detection rules that complement the existing model's output.

---

## Project Goals

The primary objective of this project is to identify fraudulent transaction patterns and engineer actionable features to improve the fraud detection pipeline. Based on an internal audit in December 2023, 0.129% of fraudulent transactions in the TRANSFER and CASH_OUT types were not flagged by the real-time system, resulting in unmitigated financial risk.

The goal is twofold:
1. Surface transaction traits common to undetected fraud cases.
2. Build structured datasets that will allow data scientists to train more accurate predictive models.

---

## Insights and Recommendations are Provided in Detail on the Following Key Areas

- **Category 1**: Transaction Type Patterns  
- **Category 2**: Feature Engineering  
- **Category 3**: Fraud Detection Trends  
- **Category 4**: Model Performance  

**SQL query links and Tableau dashboard**:  
- Data cleaning SQL scripts: [link]  
- Business logic SQL queries: [link]  
- Interactive Tableau dashboard: [link]  

---

## Data Structure & Initial Checks

The company’s working dataset includes 6,362,620 records and is structured as one consolidated table, but for documentation purposes, it's divided into four logical tables.

### 1. `fraud_flags`
| Column             | Type    | Description                         |
|---------------------|---------|-------------------------------------|
| **transaction_id**  | INT     | PK, links to transactions           |
| isFraud             | BOOL    | Fraud indicator (1/0)               |
| isFlaggedFraud      | BOOL    | System-flagged fraud (1/0)          |

---

### 2. `transaction_types`
| Column       | Type         | Description                   |
|--------------|--------------|-------------------------------|
| **type_id**  | INT          | PK                            |
| type_name    | VARCHAR(50)  | e.g., TRANSFER, CASH_OUT      |

---

### 3. `accounts`
| Column          | Type         | Description                   |
|------------------|--------------|-------------------------------|
| **account_id**   | VARCHAR(50)  | PK (from nameOrig/nameDest)   |

---

### 4. `transactions`
| Column             | Type         | Description                                   |
|---------------------|--------------|-----------------------------------------------|
| **transaction_id**  | INT          | PK                                           |
| step                | INT          | Hourly time step                             |
| type_id             | INT          | FK to `transaction_types`                    |
| amount              | FLOAT        | Transaction value                            |
| nameOrig            | VARCHAR(50)  | FK to `accounts` (origin account)            |
| nameDest            | VARCHAR(50)  | FK to `accounts` (destination account)       |

---

### 5. `balance`
| Column             | Type         | Description                                   |
|---------------------|--------------|-----------------------------------------------|
| **balance_id**      | INT          | PK                                           |
| transaction_id      | INT          | FK to `transactions`                         |
| account_role        | VARCHAR(20)  | "origin" or "destination"                    |
| old_balance         | FLOAT        | Balance before transaction                   |
| new_balance         | FLOAT        | Balance after transaction                    |

**[Entity Relationship Diagram here]**
![Screenshot_29-4-2025_131244_dbdiagram io](https://github.com/user-attachments/assets/b1a8843d-406c-4a58-8dd8-b0e15a864440)


---

## Executive Summary

### Overview of Findings

From January 2024 transaction logs, 8,213 fraud cases were found, comprising 0.13% of all transactions. Fraudulent activity was exclusive to `TRANSFER` and `CASH_OUT` types. Fraud cases mostly involved zero destination balance post-transfer and zero origin balance post-cash-out. The model's recall can be improved by incorporating account type patterns and suspicious balance gaps.

> As a Fraud Strategy Manager, your top 3 insights are:
> 1. Fraud only occurred in two transaction types.  
> 2. Balance gaps and party roles are strong fraud indicators.  
> 3. A subset of destination accounts repeatedly appear in fraud cases.

**[Insert Visualization: Fraud counts by transaction type bar chart]**![64da6613-5426-4d89-a893-bf7e4984b5a1](https://github.com/user-attachments/assets/a2216034-bb3d-4a9f-aff6-ab9b32cd61f8)


---

## Insights Deep Dive

### Category 1: Transaction Type Patterns

- **Insight 1:** Fraud only present in `TRANSFER` and `CASH_OUT`  
- **Insight 2:** Fraud peaks between steps 400-700  
- **Insight 3:** `PAYMENT`, `DEBIT`, `CASH_IN` types had zero fraud  
- **Insight 4:** momst of fraud value comes from transactions > $100,000  

**[Insert Visualization: Fraud by Type and Step Heatmap]**![f4d030c0-81a5-46b0-a20e-07713c1aea0e](https://github.com/user-attachments/assets/3b04d760-dc7f-4bc2-90d4-b8c8d003fb50)


---

### Category 2: Feature Engineering

- **Insight 1:** `origin_balance_gap` and `destination_balance_gap` help classify fraud  
- **Insight 2:** most of fraud originators end with zero balance  
- **Insight 3:** Destination accounts show mirrored balance increases  
- **Insight 4:** Role classification (customer/merchant) segments risk  

**[Insert Visualization: Balance Gap Distribution - Fraud vs Non-Fraud]**

---

### Category 3: Fraud Detection Trends

- **Insight 1:** Fraud clusters around step 400 - 700
- **Insight 2:** Repeat destination accounts signal organized fraud  
- **Insight 3:** isFlaggedFraud = 1 appears in <0.0002% of transactions 


**[Insert Visualization: Receiver Frequency Heatmap]**

---

### Category 4: Model Performance

- **Insight 1:** Model misses symmetrical balance change cases  
- **Insight 2:** Severe class imbalance hinders recall  
- **Insight 3:** Feature engineering improves precision  
- **Insight 4:** Categorical encoding helps catch frequent entities
- **Insight 5:** using random forrest
  
### Confusion Matrix

|                | Predicted 0 | Predicted 1 |
|----------------|-------------|-------------|
| **Actual 0**   | 276242      | 1           |
| **Actual 1**   | 6           | 792         |

---

### Classification Report

|               | Precision | Recall | F1-Score | Support   |
|---------------|-----------|--------|----------|-----------|
| 0             | 1.00      | 1.00   | 1.00     | 276243    |
| 1             | 1.00      | 0.99   | 1.00     | 798       |
| **Accuracy**  |           |        | 1.00     | 277041    |
| **Macro Avg** | 1.00      | 1.00   | 1.00     | 277041    |
| **Weighted Avg** | 1.00   | 1.00   | 1.00     | 277041    |

---

### Area Under Curve  
**0.9962387915031441**

---

## Recommendations

- **Observation:** Fraud is restricted to `TRANSFER` and `CASH_OUT`  
  **Recommendation:** Focus detection logic exclusively on these types.  

- **Observation:** Balance gaps signal 85%+ of frauds  
  **Recommendation:** Add gap-based alerts to detection rules  

- **Observation:** Repeat destination accounts in fraud  
  **Recommendation:** Flag high-frequency destination accounts  

- **Observation:** Entity roles improve segmentation  
  **Recommendation:** Include entity classification in feature pipeline  

- **Observation:** Underflagging in current model  
  **Recommendation:** Retrain using SMOTE or class-weighted models  

---

## Technical Details

- **Tools Used**:  
  - **Python (Jupyter Notebook)** for cleaning, engineering, visualization  
  - **Pandas**, **Seaborn**, **Matplotlib**, **Scikit-learn** for modeling  

---

## Assumptions and Caveats

- Fraud was assumed to exist only in `TRANSFER` and `CASH_OUT`  
- Time steps are synthetic; treated as sequential hours  
- Duplicate transactions (~36,000 rows) removed  
- Some fields are artificial; assumed realistic  
- No third-party enrichment was applied to account metadata  
