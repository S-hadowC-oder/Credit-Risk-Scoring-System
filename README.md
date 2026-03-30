# 📊 Credit Risk Scoring System

> A **finance-grade** end-to-end machine learning system for predicting loan default risk — built with production-level architecture including temporal train/test splits, SHAP explainability, sklearn Pipeline design, and a fully containerized REST API.

---

## 🧠 Project Overview

This project simulates a real-world credit risk assessment pipeline as deployed by financial institutions. Given a loan applicant's profile, the system outputs a **risk score**, a **default probability**, and the **top contributing risk factors** — making it interpretable for both technical teams and business stakeholders.

Built as a flagship portfolio project targeting **senior analyst / junior ML engineer** roles in fintech and data-driven finance.

---

## 🎯 Problem Statement

**Can we predict whether a borrower will default on their loan — before the loan is issued?**

Using historical LendingClub loan data, we train a model that:
- Assigns a **probability of default** to each applicant
- Produces a **credit risk score** (0–1000 scale, similar to FICO logic)
- Explains *why* the score is what it is using SHAP values
- Serves predictions in real-time via a REST API

---

## 📦 Dataset

| Attribute | Details |
|-----------|---------|
| **Source** | [LendingClub Loan Data](https://www.kaggle.com/datasets/wordsforthewise/lending-club) |
| **Size** | ~2.2M loan records (2007–2018) |
| **Target Variable** | `loan_status` → Binary: `Default (1)` / `Fully Paid (0)` |
| **Key Features** | FICO score, loan amount, DTI ratio, annual income, employment length, grade, purpose, home ownership |

> ⚠️ Raw data files are excluded from version control (see `.gitignore`). Download the dataset from Kaggle and place CSVs in `data/raw/`.

---

## 🏗️ System Architecture

```
LendingClub CSV
      │
      ▼
┌─────────────────────┐
│  PostgreSQL Database │  ← Raw + processed data stored here
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  Feature Engineering │  ← Pandas + NumPy transformations
│  + Sklearn Pipeline  │  ← Imputation → Encoding → Scaling
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  XGBoost Classifier  │  ← Temporal train/test split
│  + Class Imbalance   │  ← scale_pos_weight / SMOTE
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│   SHAP Explainer     │  ← Global + local feature importance
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│   FastAPI REST API   │  ← Serves predictions + risk factors
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│   Docker Container   │  ← Fully portable deployment
└─────────────────────┘
```

---

## 📁 Project Structure

```
credit-risk-scoring/
│
├── data/
│   ├── raw/                  # LendingClub CSVs (gitignored)
│   └── processed/            # Cleaned, feature-engineered data
│
├── notebooks/                # EDA, prototyping, SHAP analysis
│
├── sql/                      # PostgreSQL DDL + analytical queries
│
├── src/
│   ├── ingestion/            # Data loading + DB connection
│   │   └── db.py
│   ├── features/             # Feature engineering scripts
│   ├── models/               # Sklearn Pipeline + XGBoost training
│   ├── explainability/       # SHAP value computation + plots
│   └── api/                  # FastAPI application
│       └── main.py
│
├── tests/                    # Unit + integration tests (pytest)
│
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── reports/                  # Power BI / Tableau exports
├── requirements.txt
├── .env.example              # Template for environment variables
├── .gitignore
└── README.md
```

---

## 🔬 ML Pipeline — Key Design Decisions

### ⏱️ Temporal Train/Test Split
Rather than a random split, loans from earlier years train the model and later years test it — mirroring how a real lending institution would validate a model before deployment.

```
Train: 2007 – 2015  |  Test: 2016 – 2018
```

### ⚙️ Sklearn Pipeline Architecture
The entire preprocessing + modeling flow is encapsulated in a single `sklearn.Pipeline` — preventing data leakage and making the model trivially serializable for the API.

```python
pipeline = Pipeline([
    ('preprocessor', ColumnTransformer([...])),
    ('classifier', XGBClassifier(...))
])
```

### ⚖️ Class Imbalance Handling
Default events are rare (~15–20% of the dataset). Handled via:
- `scale_pos_weight` in XGBoost
- Precision-Recall AUC as the **primary evaluation metric** (not just ROC-AUC)

### 🔍 SHAP Explainability
Every prediction comes with a human-readable explanation:

```json
{
  "applicant_id": "LC_00234",
  "default_probability": 0.73,
  "risk_score": 312,
  "risk_tier": "HIGH",
  "top_risk_factors": [
    {"feature": "dti_ratio", "impact": "+0.21", "direction": "increases_risk"},
    {"feature": "fico_score", "impact": "-0.18", "direction": "decreases_risk"},
    {"feature": "loan_amount", "impact": "+0.14", "direction": "increases_risk"}
  ]
}
```

---

## 📊 Evaluation Metrics

| Metric | Why It Matters Here |
|--------|-------------------|
| **PR-AUC** | Primary metric — robust to class imbalance |
| **ROC-AUC** | Secondary — overall discrimination ability |
| **KS Statistic** | Industry-standard for credit scoring |
| **F1 Score** | Balance between precision and recall |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- PostgreSQL 14+
- Docker (for Phase 4 deployment)

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/credit-risk-scoring.git
cd credit-risk-scoring
```

### 2. Set Up Python Environment
```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure Environment Variables
```bash
cp .env.example .env
# Edit .env with your PostgreSQL credentials
```

### 4. Load Data into PostgreSQL
```bash
python src/ingestion/load_data.py
```

### 5. Run the API Locally
```bash
uvicorn src.api.main:app --reload
# Visit: http://localhost:8000/docs
```

### 6. Run with Docker
```bash
docker-compose up --build
```

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check |
| `POST` | `/predict` | Score a single applicant |
| `POST` | `/predict/batch` | Score multiple applicants |
| `GET` | `/model/info` | Model version + metrics |

---

## 🗺️ Project Roadmap

| Phase | Focus | Status |
|-------|-------|--------|
| Phase 1 (Days 1–7) | Data Ingestion + PostgreSQL | 🔄 In Progress |
| Phase 2 (Days 8–14) | EDA + Power BI Dashboard | ⏳ Upcoming |
| Phase 3 (Days 15–25) | ML Pipeline + SHAP | ⏳ Upcoming |
| Phase 4 (Days 26–35) | FastAPI + Docker | ⏳ Upcoming |
| Phase 5 (Days 36–40) | Polish + Resume Prep | ⏳ Upcoming |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.10 |
| Data Processing | Pandas, NumPy |
| ML / Modeling | Scikit-learn, XGBoost |
| Explainability | SHAP |
| Database | PostgreSQL + SQLAlchemy |
| API | FastAPI + Uvicorn |
| Containerization | Docker, Docker Compose |
| Visualization | Power BI / Tableau |
| Version Control | Git / GitHub |
| IDE | Antigravity |

---

## 🙋 Author

**Zoro**
Built as a finance-grade portfolio project for senior analyst / junior ML engineer roles.

> *"Don't just predict default — explain it."*
