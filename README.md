
![Deprescribing CDSS](https://img.shields.io/badge/Status-Prototype-yellow)
![Python](https://img.shields.io/badge/Python-3.13+-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.104-green)
![React](https://img.shields.io/badge/React-19-blue)
![Tailwind](https://img.shields.io/badge/Tailwind-3-06B6D4)
![License](https://img.shields.io/badge/License-MIT-green)

# Deprescribing Clinical Decision Support System (CDSS)

A comprehensive, multi-module clinical decision support system designed to help healthcare providers identify potentially inappropriate medications (PIMs) in older adults and generate evidence-based deprescribing plans. The system integrates **9 clinical analysis modules**, **AI-powered prescription parsing**, **herb-drug interaction checking**, and **automated PDF report generation**.

> **Disclaimer:** This system is a clinical decision support tool only. All recommendations must be reviewed by a qualified healthcare professional before implementation.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
  - [9 Clinical Analysis Modules](#9-clinical-analysis-modules)
  - [AI-Powered Features](#ai-powered-features)
  - [Reporting & Interaction Checking](#reporting--interaction-checking)
- [Tech Stack](#tech-stack)
  - [Backend](#backend)
  - [Frontend](#frontend)
  - [Data & AI](#data--ai)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Setup](#environment-setup)
  - [Running the Application](#running-the-application)
- [API Reference](#api-reference)
- [Project Structure](#project-structure)
- [Datasets](#datasets)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Polypharmacy in older adults is a growing global health concern, leading to increased adverse drug reactions, hospitalizations, and healthcare costs. The **Deprescribing CDSS** addresses this by providing a systematic, evidence-based approach to medication review. It combines established clinical criteria (Beers, STOPP/START, ACB) with advanced AI capabilities to deliver personalized deprescribing recommendations.

### Who Is This For?

- **Geriatricians & Primary Care Physicians** — Rapid medication safety reviews
- **Clinical Pharmacists** — Detailed drug interaction and tapering analysis
- **Long-Term Care Facilities** — Routine medication regimen review
- **Medical Researchers** — Studying deprescribing patterns and polypharmacy

---

## Architecture

```
                   ┌─────────────────────────────────────┐
                   │         React SPA (Port 3000)        │
                   │  PatientForm → ResultsDashboard →    │
                   │  TaperModal / MonitoringModal / etc. │
                   └──────────────┬──────────────────────┘
                                  │ HTTP (Axios)
                                  ▼
                   ┌──────────────────────────────────────┐
                   │       FastAPI Backend (Port 8000)     │
                   │                                      │
                   │  ┌──────────────────────────────────┐ │
                   │  │      AnalysisService (Orchestrator)│ │
                   │  │  ┌─────┐ ┌──────┐ ┌──────────┐  │ │
                   │  │  │ ACB │ │Beers│ │STOPP/START│  │ │
                   │  │  ├─────┤ ├──────┤ ├──────────┤  │ │
                   │  │  │Taper│ │Frailty│ │ Gender   │  │ │
                   │  │  ├─────┤ ├──────┤ ├──────────┤  │ │
                   │  │  │ TTB │ │Ayurvedic│ │Priority │  │ │
                   │  │  └─────┘ └──────┘ └──────────┘  │ │
                   │  └──────────────────────────────────┘ │
                   │          │              │              │
                   │          ▼              ▼              │
                   │  ┌──────────┐  ┌──────────────┐      │
                   │  │ Gemini AI│  │  CSV Datasets │      │
                   │  │ (Fallback│  │ (Beers, ACB,  │      │
                   │  │  Taper   │  │  STOPP, TTB,  │      │
                   │  │  Plans)  │  │  etc.)        │      │
                   │  └──────────┘  └──────────────┘      │
                   └──────────────────────────────────────┘
```

The system follows a **modular engine** pattern where each clinical module is independently instantiated, tested, and orchestrated by a central `AnalysisService`. The frontend is a single-page React application communicating via RESTful endpoints.

---

## Features

### 9 Clinical Analysis Modules

| # | Module | Engine | What It Does |
|---|--------|--------|-------------|
| 1 | **ACB Score** | `acb_engine.py` | Calculates anticholinergic burden from a curated medication list. Flags scores ≥3 (high risk for cognitive decline). |
| 2 | **Beers Criteria** | `beers_engine.py` | Matches medications against the AGS Beers Criteria for Potentially Inappropriate Medication Use in Older Adults (age 65+). |
| 3 | **STOPP/START v2** | `stopp_engine.py` | Evaluates against STOPP (potentially inappropriate prescriptions) and START (potentially omitted beneficial therapies) criteria v2, with eGFR-aware matching. |
| 4 | **Tapering Engine** | `tapering_engine.py` | Generates evidence-based tapering schedules from a rules dataset covering 10+ drug classes. Supports Ashton protocol for benzodiazepines, hyperbolic tapering for SSRIs, and more. |
| 5 | **Frailty Risk** | `frailty_risk_engine.py` | Escalates risk scores for frail patients (CFS ≥ 6) on high-risk drug classes. Adjusts tapering timelines based on frailty status. |
| 6 | **Gender Risk** | `gender_risk_engine.py` | Identifies gender-specific medication risks (e.g., zolpidem in women, NSAIDs in men with CKD). Provides monitoring guidance. |
| 7 | **Time-to-Benefit** | `time_to_benefit_engine.py` | Compares medication benefit onset vs. patient life expectancy. Flags medications where deprescribing should be considered. |
| 8 | **Ayurvedic Interactions** | `ayurvedic_interaction_engine.py` | Checks herb-drug interactions using a known interactions database and AI-simulated pharmacological profiles for undocumented combinations. |
| 9 | **Priority Classifier** | `priority_classifier.py` | Assigns each medication a **RED** (high priority), **YELLOW** (monitor), or **GREEN** (continue) classification based on all module outputs. |

### AI-Powered Features

- **Prescription Parsing** — Upload PDFs or photos of prescriptions; the system extracts medication names, doses, and frequencies using Google Gemini Vision.
- **Brown Bag Review** — Upload photos of medication bottles; the system identifies products from label text.
- **AI Taper Plans** — For medications not in the local database, Gemini generates personalized tapering schedules based on drug class, half-life, and patient factors.
- **Monitoring Plans** — AI-generated monitoring parameter schedules with withdrawal symptom tracking.

### Reporting & Interaction Checking

- **Standalone Interaction Checker** — Check herb-drug interactions without a full patient analysis.
- **PDF Report Generation** — Download professional clinical PDF reports with formatted tables, risk summaries, and recommendations (powered by ReportLab).
- **Full API Documentation** — Auto-generated OpenAPI/Swagger docs at `/docs` and `/redoc`.

---

## Tech Stack

### Backend

| Technology | Version | Purpose |
|-----------|---------|---------|
| Python | 3.13+ | Runtime |
| FastAPI | 0.104.1 | Web framework (async, auto-docs, Pydantic validation) |
| Uvicorn | 0.24.0 | ASGI server |
| Pandas | 2.1.3 | CSV data processing |
| Pydantic | 2.5.0 | Request/response validation |
| ReportLab | — | PDF report generation |
| Pillow | — | Image processing for prescription OCR |
| PyPDF2 | — | PDF text extraction |
| google-generativeai | — | Google Gemini AI integration |
| python-dotenv | — | Environment variable management |

### Frontend

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 19.2.0 | UI framework |
| React Scripts | 5.0.1 | Build tooling (CRA) |
| Tailwind CSS | 3.4 | Utility-first styling |
| Axios | 1.13 | HTTP client |
| React Select | 5.10 | Autocomplete medication search |
| Headless UI | 2.2 | Accessible modal/dropdown primitives |
| React Icons | 5.5 | UI icons |
| jsPDF + html2canvas | — | Client-side PDF export |

### Data & AI

- **Google Gemini** (gemini-1.5-flash) — Prescription parsing, taper plan generation, monitoring plans
- **CSV Datasets** — Curated clinical rules stored in `datasets/` (easily extensible without code changes)
- **JSON Profiles** — Ayurvedic pharmacological profiles for AI-driven interaction simulation

---

## Getting Started

### Prerequisites

- Python 3.13+
- Node.js 18+ and npm
- Google Gemini API key ([get one free](https://aistudio.google.com/apikey))

### Installation

**1. Clone the repository**

```bash
git clone <repository-url>
cd Docathon
```

**2. Backend setup**

```bash
cd backend
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
pip install -r requirements.txt
```

> **Note:** `requirements.txt` only lists core dependencies. ReportLab, PyPDF2, Pillow, google-generativeai, and python-dotenv are imported but not listed — install them manually:
> ```bash
> pip install reportlab PyPDF2 Pillow google-generativeai python-dotenv
> ```

**3. Frontend setup**

```bash
cd frontend
npm install
```

### Environment Setup

Create a `.env` file in the `backend/` directory:

```bash
GEMINI_API_KEY=your_gemini_api_key_here
```

### Running the Application

**Start the backend** (from `backend/`):

```bash
python app/main.py
# OR
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Start the frontend** (from `frontend/`):

```bash
npm start
```

The application will be available at:
- **Frontend:** http://localhost:3000
- **API:** http://localhost:8000
- **Swagger Docs:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

---

## API Reference

| Method | Endpoint | Description | Tags |
|--------|----------|-------------|------|
| `POST` | `/analyze-patient` | Comprehensive 9-module patient medication analysis | Analysis |
| `POST` | `/get-taper-plan` | Detailed tapering schedule for a specific medication | Tapering |
| `POST` | `/interaction-checker` | Standalone herb-drug interaction check | Interactions |
| `POST` | `/extract-from-prescription` | Extract medications from PDF/image prescription | Medication Extraction |
| `POST` | `/extract-brown-bag` | Extract medications from brown bag review photo | Medication Extraction |
| `POST` | `/generate-report-pdf` | Generate downloadable clinical PDF report | Reports |
| `GET` | `/` | API root with system info | System |
| `GET` | `/health` | System health check with dataset counts | System |
| `GET` | `/supported-drugs` | List of drugs with tapering protocols | Reference |
| `GET` | `/supported-herbs` | List of supported Ayurvedic herbs | Reference |

### Example: Analyze Patient

```bash
curl -X POST "http://localhost:8000/analyze-patient" \
  -H "Content-Type: application/json" \
  -d '{
    "patient": {
      "age": 78,
      "gender": "female",
      "is_frail": true,
      "cfs_score": 6,
      "life_expectancy": "2-5_years",
      "comorbidities": ["dementia", "hypertension", "CKD stage 3"],
      "serum_creatinine_mg_dl": 1.4,
      "medications": [
        {"generic_name": "diphenhydramine", "dose": "25mg", "frequency": "qhs", "duration": "long_term"},
        {"generic_name": "omeprazole", "dose": "20mg", "frequency": "daily", "duration": "long_term"}
      ],
      "herbs": [
        {"generic_name": "Ashwagandha", "dose": "500mg", "frequency": "daily", "duration": "long_term"}
      ]
    }
  }'
```

---

## Project Structure

```
Docathon/
├── backend/
│   ├── app/
│   │   ├── main.py                     # FastAPI entry, endpoints, engine init
│   │   ├── models/
│   │   │   ├── patient.py              # Patient, Medication, HerbalProduct, enums
│   │   │   ├── api_models.py           # Request/Response schemas
│   │   │   └── responses.py            # Shared response types
│   │   ├── services/
│   │   │   ├── analysis_service.py     # Core orchestrator (runs all engines)
│   │   │   ├── acb_engine.py           # Anticholinergic Burden Score
│   │   │   ├── beers_engine.py         # AGS Beers Criteria
│   │   │   ├── stopp_engine.py         # STOPP/START v2 criteria
│   │   │   ├── tapering_engine.py      # Rule-based taper plans
│   │   │   ├── taper_plan_service.py   # AI-augmented taper plan service
│   │   │   ├── frailty_risk_engine.py  # CFS-based frailty escalation
│   │   │   ├── gender_risk_engine.py   # Gender-specific medication risks
│   │   │   ├── time_to_benefit_engine.py # Life expectancy vs. benefit
│   │   │   ├── ayurvedic_interaction_engine.py # Herb-drug interactions
│   │   │   ├── gemini_service.py       # Google Gemini integration
│   │   │   ├── clinical_calculators.py  # eGFR (CKD-EPI), MELD score
│   │   │   ├── organ_function_checker.py # Renal/hepatic safety checks
│   │   │   ├── risk_classifier.py       # Multi-factor risk scoring
│   │   │   ├── priority_classifier.py   # RED/YELLOW/GREEN assignment
│   │   │   ├── prescription_parser.py   # AI prescription image/PDF parsing
│   │   │   ├── pdf_generator.py         # ReportLab PDF reports
│   │   │   └── stopp_start_analyzer.py  # Enhanced STOPP/START analyzer
│   │   └── utils/
│   │       └── data_loader.py           # Dataset loader functions
│   ├── requirements.txt
│   ├── test.py
│   └── .gitignore
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── index.js / index.css        # Entry point + Tailwind
│   │   ├── App.jsx / App.css           # Root component
│   │   ├── components/
│   │   │   ├── PatientForm.jsx         # Patient data intake
│   │   │   ├── MedicationTable.jsx     # Input review table
│   │   │   ├── ResultsDashboard.jsx    # Main results display
│   │   │   ├── MedicationExtractor.jsx # AI prescription upload
│   │   │   ├── TaperModal.jsx          # Taper plan detail modal
│   │   │   ├── MonitoringModal.jsx     # Monitoring plan modal
│   │   │   ├── InteractionModal.jsx    # Interaction detail modal
│   │   │   └── LoadingSpinner.jsx      # Loading animation
│   │   ├── services/
│   │   │   └── api.js                  # Axios client
│   │   └── utils/
│   │       ├── constants.js            # Form options
│   │       └── medicationOptions.js    # Drug/herb databases
│   ├── package.json
│   └── tailwind.config.js
├── datasets/
│   ├── acb_brand_list.csv              # Anticholinergic burden medications
│   ├── beers.csv                       # Beers Criteria PIMs
│   ├── stopp_criteria_v2.csv           # STOPP v2 criteria
│   ├── start_criteria_v2.csv           # START v2 criteria
│   ├── tapering_rules_dataset.csv      # Tapering protocols
│   ├── cfs_risk_taper_map.csv          # Frailty risk mapping
│   ├── gender_risk_medications.csv     # Gender-specific risks
│   ├── time_to_benefit_dataset.csv     # Time-to-benefit data
│   ├── ayurvedic_known_interactions.csv # Known herb-drug interactions
│   ├── ayurvedic_herbs_summary.csv     # Herb summaries
│   └── ayurvedic_pharmacological_profiles.json # AI interaction profiles
├── Text.ipynb                          # Experimental notebook
└── README.md
```

---

## Datasets

All clinical rules live as CSV/JSON files in `datasets/`, making them easily auditable and extensible:

| Dataset | Records | Source |
|---------|---------|--------|
| `acb_brand_list.csv` | ~100+ | Published ACB literature |
| `beers.csv` | ~60+ | AGS Beers Criteria 2019/2023 |
| `stopp_criteria_v2.csv` | ~80+ | STOPP/START v2 (O'Mahony et al.) |
| `start_criteria_v2.csv` | ~30+ | STOPP/START v2 |
| `tapering_rules_dataset.csv` | ~10 drug classes | Clinical guidelines |
| `cfs_risk_taper_map.csv` | 9 levels | Rockwood Clinical Frailty Scale |
| `gender_risk_medications.csv` | ~30 | FDA/EMA safety communications |
| `time_to_benefit_dataset.csv` | ~50+ | Published RCTs and meta-analyses |
| `ayurvedic_known_interactions.csv` | ~100+ | Clinical literature |
| `ayurvedic_pharmacological_profiles.json` | ~100+ | Pharmacological data |

---

## Roadmap

See [FUTURE_IMPROVEMENTS.md](./FUTURE_IMPROVEMENTS.md) for a comprehensive list of planned enhancements to make this system production-ready.

### Near-Term

- [ ] Comprehensive test suite (pytest for backend, expanded Jest tests for frontend)
- [ ] Docker containerization (Dockerfile + docker-compose)
- [ ] User authentication and multi-tenant support
- [ ] Patient history tracking and longitudinal analysis
- [ ] CI/CD pipeline (GitHub Actions)

### Long-Term

- [ ] EHR integration (FHIR R4 API)
- [ ] NDC/RxNorm drug database with automated updates
- [ ] Advanced analytics dashboard for population health
- [ ] Offline capability with service worker + IndexedDB
- [ ] HIPAA compliance framework

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push: `git push origin feature/your-feature`
5. Open a Pull Request

Please ensure your code follows the existing style patterns and includes appropriate tests.

---

## License

This project is licensed under the MIT License — see the LICENSE file for details.

---

## Acknowledgments

- **AGS Beers Criteria** — American Geriatrics Society
- **STOPP/START Criteria** — O'Mahony et al., European Journal of Clinical Pharmacology
- **Clinical Frailty Scale** — Rockwood et al., Canadian Medical Association Journal
- **Google Gemini** — AI assistance for prescription parsing and taper plan generation
- **Ashton Protocol** — Prof. Heather Ashton, benzodiazepine tapering guidelines
