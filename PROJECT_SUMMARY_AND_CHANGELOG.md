# DocuBrix: Complete Project Evolution & Collaborative Development Guide
**From Phase 1 MVP to Production Cloud Release & Academic Evaluation**  
*Project Code:* **ARP 455 (B.Tech 7th Semester — Industrial Internet of Things)*  
*Institution:* **University School of Automation and Robotics (USAR), GGSIPU, East Delhi Campus*  
*Team Members:* **Dhruv Grover (00619051723)** & **Pranav Bisht (02719051723)**  
*Project Supervisor:* **Dr. Bhanu Prakash Joshi (Associate Professor, USAR)*  
*Document Version:* **v2.0 — Final Comprehensive Release (November 2026)*  

---

## 📌 Purpose of this Guide
This document serves as an exhaustive, developer-ready summary of all engineering developments, architectural refactorings, bug fixes, test suites, cloud deployments, and academic deliverables produced since our initial Internal Evaluation 1.

It is specifically structured for **Dhruv Grover** and collaborators to:
1. Understand every code and architectural change made between the initial prototype and the final production system.
2. Spin up, run, test, and debug the entire application locally without friction.
3. Understand the deployed production cloud infrastructure on Render.
4. Review the final academic report and presentation slides aligned with GGSIPU guidelines.

---

## 1. Executive Summary: What is DocuBrix Today?

DocuBrix has transitioned from a basic, single-domain invoice parser script into an **enterprise-grade, multi-domain Document Intelligence, Autonomous Workflow Automation, and Grounded RAG Platform**.

### Core Platform Capabilities
- **Dual-Engine Ingestion & OCR**: High-speed native digital text extraction via **PyMuPDF** (`fitz`) and fallback computer-vision OCR for scanned/handwritten documents via **Tesseract OCR** and **OpenCV**.
- **14-Class Multi-Domain Machine Learning Classification**: Statistical TF-IDF vectorization + Cosine Similarity against domain centroid vectors with deterministic keyword heuristic tiebreaking across 4 operational domains:
  1. *Financial*: Invoices, Receipts, Purchase Orders, Tax Statements, Salary Slips.
  2. *Healthcare / Medical*: Prescriptions, Clinical Diagnostics, Medical Invoices.
  3. *Legal & Compliance*: Non-Disclosure Agreements (NDAs), Employment Agreements, Service Level Agreements (SLAs).
  4. *Academic & Administrative*: Official Circulars, Grade Transcripts, KYC/Identity Cards.
- **Deterministic Regex Schema Extractors**: Auditable, schema-compliant entity extraction with zero risk of generative hallucinations.
- **Calibrated Multi-Signal Confidence Scoring ($\mu \pm 1.5\sigma$)**: A rigorous 4-signal weighted scoring model combining regex match density, mandatory field completeness, relational cross-field mathematical validation (e.g., $\text{Subtotal} + \text{Tax} = \text{Total}$), and format compliance.
- **Dynamic Workflow Policy Rule Engine**: Autonomous event-driven routing:
  - $\ge \mu + 1.5\sigma \implies$ **AUTO_APPROVE**
  - Between $\mu - \sigma$ and $\mu + 1.5\sigma \implies$ **FLAG_FOR_HUMAN_REVIEW**
  - $< \mu - \sigma$ or missing critical compliance fields $\implies$ **REJECT**
  - High-risk monetary thresholds (e.g., Total $> \$5,000$ or $\text{INR } 50,000$) automatically route to human review.
  - Complete state transitions preserved in an audit trail with **1-Click CSV Export**.
- **Grounded Semantic RAG Assistant**: Multi-turn conversational copilot powered by chunk-level semantic vector embeddings (**Google Gemini text-embedding-004** / 768-dimension vectors) + generative Q&A (**Gemini 1.5 Flash**), strictly bound to retrieved chunk evidence with source citation. Includes a **deterministic extractive fallback engine** if API keys or quotas are unavailable.
- **Enterprise SaaS Dashboard**: Built with **React 18**, **Vite**, **Tailwind CSS**, and **Lucide Icons**, featuring live KPI metric counters, interactive notification menus, review queues, audit logs, and demo presets.

---

## 2. Repository & Workspace Layout

```
Pranav_minor_project/
├── DocuBrix/                              <-- THE ACTIVE PRODUCTION REPOSITORY
│   ├── backend/
│   │   ├── app/
│   │   │   ├── api/routes/               <-- Modular FastAPI endpoint routers
│   │   │   │   ├── auth.py               <-- JWT login, registration, user profile
│   │   │   │   ├── documents.py          <-- Document CRUD, metadata, filtering
│   │   │   │   ├── ingestion.py          <-- File upload, PyMuPDF, OCR pipeline
│   │   │   │   ├── workflows.py          <-- Policy rule engine & review queue
│   │   │   │   ├── knowledge.py          <-- Chunking, vector indexing & RAG chat
│   │   │   │   └── analytics.py          <-- Aggregated KPIs & system statistics
│   │   │   ├── core/                     <-- Configuration, security, JWT helpers
│   │   │   ├── db/                       <-- SQLAlchemy 2.0 engine & session maker
│   │   │   ├── models/                   <-- Declarative DB models (User, Doc, Chunk, Workflow)
│   │   │   ├── schemas/                  <-- Pydantic v2 validation models
│   │   │   ├── services/                 <-- Core intelligence engines:
│   │   │   │   ├── document_processor.py <-- Dual-engine OCR & universal regex schemas
│   │   │   │   ├── classifier.py         <-- 14-class TF-IDF + heuristic classifier
│   │   │   │   ├── confidence.py         <-- 4-signal calibrated confidence scoring
│   │   │   │   ├── workflow.py           <-- Policy rule engine & CSV audit logger
│   │   │   │   └── knowledge.py          <-- RAG vector chunking & Gemini assistant
│   │   │   └── main.py                   <-- FastAPI app root, CORS, router inclusion
│   │   ├── tests/                        <-- Comprehensive PyTest test suite (90 tests)
│   │   ├── requirements.txt              <-- Python 3.11 pinned dependencies
│   │   └── Dockerfile                    <-- Container definition for Render backend
│   ├── frontend/
│   │   ├── src/
│   │   │   ├── components/               <-- Reusable UI: Header, Sidebar, Notifications, Review
│   │   │   ├── pages/                    <-- Dashboard, Upload, Library, RAG Chat, Review Queue
│   │   │   ├── services/api.js           <-- Axios HTTP client with Bearer JWT interceptors
│   │   │   ├── App.jsx                   <-- Master routing & state management
│   │   │   └── index.css                 <-- Tailwind CSS utility directives
│   │   ├── package.json                  <-- React 18, Vite, Lucide dependencies
│   │   ├── vite.config.js                <-- Proxy & build configurations
│   │   └── tailwind.config.js            <-- Styling & theme configuration
│   ├── render.yaml                       <-- Production cloud deployment blueprint
│   ├── start_local.bat                   <-- 1-click Windows development launcher
│   └── docubrix.db                       <-- Zero-config local SQLite development database
│
├── Final_Evaluation/                     <-- ACADEMIC EVALUATION & SUBMISSIONS
│   ├── Reports/
│   │   ├── DocuBrix_ARP455_Final_Report.pdf   <-- 32-Page Minor Project Report (USAR format)
│   │   ├── DocuBrix_Final_Presentation_ARP455_v2.pptx <-- 12-Slide Final PPT (Evaluation 1 format)
│   │   ├── report_frontmatter.html            <-- HTML source for preliminary pages
│   │   └── report_main.html                   <-- HTML source for chapters 1-8
│   ├── Assets/                                <-- Official college logo, architectural diagrams, ERD, DFD
│   └── Screenshots/Website/                   <-- High-res UI screenshots of the live system
│
└── compile_final_arp455.py               <-- Automated Chrome headless + PyMuPDF report builder
```

---

## 3. Detailed Version Comparison (Previous vs. Current)

| Component | Previous Version (Evaluation 1 Prototype) | Current Production Version (Evaluation 2 / Final) |
|---|---|---|
| **Document Support** | Single-domain: financial invoices only. | **14 Document Types across 4 Domains**: Financial, Medical, Legal, Academic/Administrative. |
| **Classification** | Basic string keyword matching in raw text. | **Hybrid TF-IDF Statistical Classifier + Cosine Centroids + Keyword Heuristics** with fallback score. |
| **Extraction Reliability** | Brittle regex tokens; dummy fallback mock dictionaries when regex failed. | **Hardened Universal Schema Extractors** with typed field casting, zero dummy mocks, and validation flags. |
| **Confidence Scoring** | Arbitrary static score assigned to mock outputs. | **Mathematical Multi-Signal Scoring Engine** ($\mu \pm 1.5\sigma$ distribution calibration across 4 orthogonal signals). |
| **Workflow Engine** | Non-existent; status hardcoded as `processed`. | **Autonomous Policy Rule Engine** (threshold rules, compliance rules, approval/flagging/rejection, CSV audit logs). |
| **Conversational AI** | Unconnected UI mock; generic LLM responses. | **Grounded RAG Assistant** (PyMuPDF chunking $\to$ Gemini text-embedding-004 $\to$ cosine retrieval $\to$ Gemini 1.5 Flash cited answers + extractive fallback). |
| **Database & Tenancy** | Single flat SQLite table with no ownership checks. | **Dual-Engine SQLAlchemy 2.0 ORM** (SQLite locally, PostgreSQL on cloud) with strict tenant ownership (`owner_id`) on all queries. |
| **Authentication** | None or temporary frontend mockup. | **Production JWT Bearer Authentication** with PBKDF2 password hashing, secure token storage, and auto-refresh on reload. |
| **Frontend UI/UX** | Basic developer debugging layout with placeholder links. | **Complete SaaS Application** (interactive notifications, filterable library, live confidence gauges, review queue, demo presets). |
| **Deployment** | Local only (`localhost`). | **Live Multi-Service Cloud Deployment on Render** (`docubrix-api-bywf` and `docubrix-frontend-bywf` in Singapore region). |
| **Automated Testing** | ~2 basic unit checks. | **90 Automated PyTest Suites** covering auth, ingestion, classification, schemas, scoring, workflows, and RAG (100% pass rate). |

---

## 4. Key Engineering Implementations & Code Commits

### Commit `016620f` — Multi-Domain Classification & Workflow Engine
- Implemented `backend/app/services/classifier.py`: Vectorizes extracted document tokens using scikit-learn TF-IDF, calculates cosine distance to pre-computed domain centroid vectors, and selects the optimal document schema.
- Implemented `backend/app/services/workflow.py`: Evaluates dynamic condition rules against extracted payload parameters (e.g., flagging total amounts $> \$5,000$, detecting missing confidentiality clauses in NDAs, catching abnormal blood pressure in clinical reports).
- Created database model `WorkflowRecord` storing `document_id`, `rule_triggered`, `decision`, `reviewer_notes`, and timestamps.

### Commit `e2f4803` & `f6bdfef` — Grounded RAG Assistant & Gemini Integration
- Implemented `backend/app/services/knowledge.py`:
  - Splits documents into 500-token chunks with 50-token overlaps.
  - Generates 768-dimensional dense vector embeddings via `models/text-embedding-004`.
  - Performs vector cosine similarity retrieval merged with BM25 keyword matching.
  - Queries `models/gemini-1.5-flash` with a strict system prompt forbidding external hallucinations: answers must cite chunk indices.
  - Added a deterministic extractive fallback engine to guarantee conversational responses even when API quotas or internet connectivity drop.

### Commit `134059d` — Confidence Score Calibration & Auth Persistence
- Implemented mathematical calibration in `backend/app/services/confidence.py`:
  $$\text{Score} = \omega_1 \cdot S_{\text{regex}} + \omega_2 \cdot S_{\text{completeness}} + \omega_3 \cdot S_{\text{validation}} + \omega_4 \cdot S_{\text{format}}$$
  - Prevents artificial 1.0 or 0.0 scores by calibrating against realistic Gaussian distributions.
- Fixed React client JWT session persistence: tokens stored in `localStorage` are automatically attached to Axios headers via request interceptors, preventing unwanted logouts on browser page refreshes.

### Commit `09353fe` — Notification Center & UI Enhancements
- Added an interactive Notification Dropdown component to the top navigation bar.
- Tracks real-time system events: new document uploads, workflow policy triggers, high-risk reviews, and system notices with unread badges, timestamps, and one-click dismissal.

### Commit `eec3fd8` — 14-Class Expansion & Upsert Deduplication
- Expanded classifier and extraction schemas to handle 14 discrete sub-document types:
  - Financial: Invoice, Tax Invoice, Receipt, Purchase Order, Salary Slip.
  - Medical: Prescription, Lab Report, Medical Bill.
  - Legal: NDA, Service Agreement, Employment Contract.
  - Academic: Marksheet/Transcript, Official Circular, Government Identity (Aadhaar/PAN/Passport).
- Added hash-based file deduplication: uploading the same document updates existing metadata rather than duplicating database records.

### Commit `2b902ee` & `fe62d2e` — Cloud Deployment & Regional Localization
- Configured `render.yaml` for zero-downtime containerized builds in the **Singapore region** (lowest latency for India).
- Corrected CORS origins and set `VITE_API_URL` to communicate seamlessly with `https://docubrix-api-bywf.onrender.com`.

---

## 5. How to Run & Test the Project Locally (Guide for Dhruv)

### Prerequisites
- **Python 3.11+** installed and available in your PATH.
- **Node.js (v18 or v20)** & **npm** installed.
- **Git** installed.

---

### Method A: 1-Click Launch (Recommended for Windows)
Inside the `DocuBrix` root folder, simply double-click or run:
```bat
start_local.bat
```
This automated launcher script will:
1. Verify Python and Node environments.
2. Check and initialize dependencies if missing.
3. Start the FastAPI backend server on `http://localhost:8000`.
4. Start the Vite React development server on `http://localhost:5173`.
5. Open your default web browser directly to the DocuBrix dashboard.

---

### Method B: Manual Startup

#### Step 1: Clone or Navigate to the Repository
```bash
cd C:\Users\bisht\Pranav-Work\Pranav_minor_project\DocuBrix
```

#### Step 2: Set Up & Run the Backend
```bash
cd backend

# Create virtual environment (if not already present)
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run database migrations / initialize SQLite
python -c "from app.db.database import init_db; init_db()"

# Start the FastAPI server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
*Backend API Docs will be available at:* `http://localhost:8000/docs`

#### Step 3: Set Up & Run the Frontend
In a separate terminal window:
```bash
cd frontend

# Install Node dependencies
npm install

# Start Vite dev server
npm run dev
```
*Frontend will be running at:* `http://localhost:5173`

---

### Running the Test Suite
To execute the comprehensive automated test suite:
```bash
cd backend
pytest -v
```
*Expected Result:* **90 passed (100% pass rate)**.

---

## 6. Live Production Cloud Deployment Reference

The project is continuously deployed on Render cloud infrastructure:

| Component | Service Name | Live Production URL |
|---|---|---|
| **Frontend Web App** | `docubrix-frontend-bywf` | **https://docubrix-frontend-bywf.onrender.com** |
| **Backend REST API** | `docubrix-api-bywf` | **https://docubrix-api-bywf.onrender.com** |
| **Interactive API Docs** | Swagger UI | **https://docubrix-api-bywf.onrender.com/docs** |
| **Database** | PostgreSQL 16 (Neon) | Connected via cloud connection string in Render environment |

---

## 7. Academic Deliverables Aligned with USAR / GGSIPU Guidelines

All academic submission files have been generated, fully formatted, and archived in the repository:

### 1. Minor Project Report (ARP 455)
- **Path:** `Final_Evaluation/Reports/DocuBrix_ARP455_Final_Report.pdf`
- **Length:** **32 Pages** (Strictly satisfies the $\ge 30$ pages university requirement).
- **Format Compliance:**
  - Font: 12pt Times New Roman, 1.5 line spacing.
  - Margins: 3.5 cm (Left), 2.5 cm (Top), 1.25 cm (Right & Bottom).
  - Front Matter Numbering: Small Roman numerals (`i` to `x`).
  - Chapter Numbering: Arabic numerals (`1` to `21`).
  - Chapters Covered: 1 Introduction, 2 Literature Survey, 3 Problem Statement, 4 Description of Modules, 5 Methodology Adopted (Flowcharts, Hardware/Software, DFD Level 0/1, ERD, Algorithms), 6 Results & Discussions (Snapshots, Tables, Comparative Analysis), 7 Conclusion & Future Scope, 8 References (APA format).

### 2. Minor Project Presentation (PPTX)
- **Path:** `Final_Evaluation/Reports/DocuBrix_Final_Presentation_ARP455_v2.pptx`
- **Format:** 12 Widescreen (16:9) slides matching the exact design system of our Evaluation 1 deck (`Aptos Display` + `Aptos`, purple `#4F43C3` header, navy `#1F2358` footer, custom accent cards).
- **Authors:** Features both **Dhruv Grover (00619051723)** and **Pranav Bisht (02719051723)** on the title slide, closing slide, and bottom running footer.

---

## 8. Collaborative Next Steps & Roadmap for Dhruv

If Dhruv or the team wishes to expand the project further:
1. **Vision-Language Transformers**: Explore integrating fine-tuned lightweight open-source models (such as `microsoft/Florence-2` or `naver-clova-ix/donut-base`) for complex multi-column table extraction.
2. **Real-Time WebSockets**: Upgrade the review queue to stream live approval events across connected administrative clients.
3. **Kafka / RabbitMQ Event Queue**: Introduce message queueing between file ingestion and vector embedding generation to handle concurrent high-volume batch enterprise uploads.
4. **Enhanced Analytics**: Add automated expenditure forecasting and anomaly detection on extracted financial datasets using ARIMA or Prophet.

---
*Prepared with precision for the DocuBrix Engineering Team.*  
*Dhruv Grover & Pranav Bisht — USAR, GGSIPU — November 2026*
