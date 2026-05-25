# 🚨 Healthcare Claims Fraud Detection

> AI-driven fraud risk assessment for healthcare providers using XGBoost and provider-level claims aggregation.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.28+-FF4B4B?logo=streamlit&logoColor=white)](https://streamlit.io)
[![XGBoost](https://img.shields.io/badge/XGBoost-ML-blue)](https://xgboost.readthedocs.io)

---

## 📋 Overview

This project detects potentially fraudulent healthcare providers by analyzing aggregated claims data at the provider level. It uses an **XGBoost classifier** inside a **scikit-learn Pipeline** (with imputation + scaling) trained on the [Healthcare Provider Fraud Detection](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis) dataset from Kaggle.

The system provides two interfaces:
- **FastAPI REST API** — programmatic single and batch prediction endpoints
- **Streamlit Dashboard** — interactive UI with CSV upload and manual claim entry

---

## ✨ Features

- 🔍 **Single Prediction** — submit provider features via API and get fraud probability
- 📂 **Batch Prediction** — upload CSV files for bulk fraud screening
- 📊 **Interactive Dashboard** — Streamlit-based UI with KPI cards, donut charts, and fraud heat bars
- 📝 **Manual Claim Entry** — form-based input for ad-hoc claim checking
- ⬇️ **Export Results** — download batch predictions as CSV
- ✅ **Input Validation** — feature count validation on API, field validation on dashboard
- 🎯 **67-Feature Model** — comprehensive provider-level feature engineering

---

## 🏗️ Architecture

```
┌─────────────────┐     ┌──────────────────┐
│   Streamlit UI  │     │   FastAPI API     │
│   (app.py)      │     │   (main.py)       │
│   Port 8501     │     │   Port 8000       │
└────────┬────────┘     └────────┬──────────┘
         │                       │
         └───────┬───────────────┘
                 │
         ┌───────▼───────┐
         │  ML Pipeline  │
         │  (joblib)     │
         │               │
         │ ColumnTransf. │
         │ → Imputer     │
         │ → Scaler      │
         │ → XGBClassif. │
         └───────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| ML Model | XGBoost, scikit-learn Pipeline |
| API | FastAPI, Uvicorn, Pydantic |
| Dashboard | Streamlit, Plotly |
| Data | pandas, NumPy |
| Serialization | joblib |
| Training | Jupyter Notebook (MODEL.ipynb) |

---

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- pip

### Installation

```bash
# Clone the repository
git clone https://github.com/Karthik-333/Claims-fraud-detection.git
cd Claims-fraud-detection

# Install dependencies
pip install -r requirements.txt
```

### Run the FastAPI API

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at:
- Swagger docs: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

### Run the Streamlit Dashboard

```bash
streamlit run app.py
```

The dashboard will open at http://localhost:8501.

---

## 📡 API Endpoints

### `GET /`
Health check endpoint.

**Response:**
```json
{
  "message": "✅ Fraud Detection API is running (no Kafka/DB)"
}
```

### `POST /predict/`
Single provider fraud prediction.

**Request:**
```json
{
  "provider_id": "PRV001",
  "features": [1.0, 5000.0, 2500.0, ... ]  // 67 float values
}
```

**Response:**
```json
{
  "provider_id": "PRV001",
  "prediction": "Fraudulent",
  "probability": 0.8723,
  "risk_factors": ["Unusual billing patterns"]
}
```

### `POST /predict/batch/`
Batch prediction via CSV upload. The CSV must contain columns matching the model's 67 expected features.

**Response:**
```json
{
  "predictions": [
    {
      "provider_id": "0",
      "prediction": "Fraud",
      "probability": 0.9112,
      "threshold": 0.5,
      "risk_factors": []
    }
  ]
}
```

---

## 📁 Project Structure

```
Claims-fraud-detection/
├── model/
│   └── trained_model.joblib      # Trained sklearn Pipeline (ColumnTransformer + XGBClassifier)
├── app.py                        # Streamlit dashboard
├── main.py                       # FastAPI REST API
├── schemas.py                    # Pydantic request/response schemas
├── models.py                     # SQLAlchemy model (placeholder for future DB integration)
├── feature_columns.json          # Feature names reference (64-col notebook model)
├── decision_threshold.json       # Threshold config reference
├── xgb_fraud_model.pkl           # Raw XGBClassifier from notebook (64 features)
├── xgb_fraud_model_calibrated.pkl# Calibrated model from notebook (64 features)
├── MODEL.ipynb                   # Model training notebook (Kaggle dataset)
├── requirements.txt              # Python dependencies
├── .gitignore                    # Git ignore rules
└── README.md                     # This file
```

> **Note:** The project contains model artifacts from two training pipelines. The active model used by both `app.py` and `main.py` is `model/trained_model.joblib` (67-feature Pipeline). The `.pkl` files and `feature_columns.json` are artifacts from the notebook's training pipeline (64 features, different naming).

---

## 🧠 Model Details

- **Algorithm:** XGBoost (`XGBClassifier`) wrapped in a scikit-learn `Pipeline`
- **Preprocessing:** `ColumnTransformer` → `SimpleImputer` (median) + `StandardScaler`
- **Features:** 67 provider-level aggregated features including:
  - Inpatient/outpatient claim counts and reimbursement statistics
  - Patient demographics (age, gender, chronic conditions)
  - Length-of-stay statistics
  - Coverage and deductible amounts
  - Claim ratios and totals
- **Threshold:** 0.5 (configurable)
- **Training Data:** [Healthcare Provider Fraud Detection Analysis](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis) (Kaggle)

---

## 🔧 Configuration

| Parameter | Location | Default | Description |
|-----------|----------|---------|-------------|
| Model path | `main.py`, `app.py` | `model/trained_model.joblib` | Path to the trained model |
| Threshold | `main.py`, `app.py` | `0.5` | Fraud classification threshold |
| CORS origins | `main.py` | `http://localhost:8080` | Allowed frontend origins |
| API port | CLI argument | `8000` | FastAPI server port |
| Dashboard port | CLI argument | `8501` | Streamlit server port |

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|---------|
| `ModuleNotFoundError` | Run `pip install -r requirements.txt` |
| `InconsistentVersionWarning` (sklearn) | Cosmetic warning — model was saved with sklearn 1.7.0. Safe to ignore. |
| `WARNING: xgboost serialization` | Cosmetic warning — model was saved with older XGBoost. Safe to ignore. |
| Model load fails | Ensure `model/trained_model.joblib` exists |
| `models.py` import error | Expected — `models.py` is a placeholder for future DB integration and is not imported at runtime |

---

## 🔮 Future Improvements

- [ ] Load threshold from `decision_threshold.json` instead of hardcoding
- [ ] Add database integration for persisting prediction results
- [ ] Implement real-time KPI cards based on actual prediction history
- [ ] Add authentication to API endpoints
- [ ] Create Docker / docker-compose setup
- [ ] Add CI/CD pipeline with automated testing
- [ ] Implement SHAP-based model explainability
- [ ] Add more comprehensive input validation on manual entry form
- [ ] Support for the calibrated model (`xgb_fraud_model_calibrated.pkl`)

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👤 Author

**Karthik** — [GitHub](https://github.com/Karthik-333)
