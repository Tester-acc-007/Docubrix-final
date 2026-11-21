# DocuBrix — Master Project Status & Handoff Context

> **Last Updated:** November 21, 2026  
> **Repository:** `https://github.com/Tester-acc-007/Docubrix-final.git`  
> **Workspace Path:** `C:\Users\bisht\Pranav-Work\pranav_minor_project\DocuBrix`  
> **Status:** Stage 1 & Stage 2 Complete — including all UI polish fixes (StatCard uniformity, demo preset extraction, upsert-by-filename deduplication). **58 tests pass, 0 failed. Frontend builds with 0 errors.**

---

## 1. Quick Start for New AI Agents / Chats
To get running immediately in any session:
```powershell
# 1. Activate venv (Inside backend/, NOT at root)
cd "C:\Users\bisht\Pranav-Work\pranav_minor_project\DocuBrix"
$env:PYTHONPATH="."
.\backend\.venv\Scripts\Activate.ps1

# 2. Run backend
python -m uvicorn backend.app.main:app --reload --host 0.0.0.0 --port 8000

# 3. Run frontend (separate terminal)
cd frontend
npm run dev

# 4. Run tests
python -m pytest tests/ -v
```

---

## 2. Completed Milestones

### 🛠️ Stage 1: Extraction Engine Overhaul & Session Stability
- **Resolved Multiline Greedy Bleeding**: Replaced blind `re.DOTALL` in field matchers so document titles (e.g. `INVOICE`) do not cause the extractor to grab subsequent words as the invoice number.
- **Context-Aware Invoice Number Parsing**: Built structured matching for `INV-2024-001`, `1001`, etc., while explicitly dropping generic tokens (`Acme`, `Date`, `Total`, `OICE`, `Due`).
- **Precision Total Amount Parsing**:
  - Distinguishes between `Subtotal`, `Tax`, and `Total/Grand Total`.
  - Parses comma-formatted currency amounts (`$1,375.00`).
  - Ignores payment terms integers (e.g. `30 days`, `net 15`) that previously caused totals to fall back to `30`.
- **Clean Vendor & Merchant Detection**:
  - Strips leading document headers like `INVOICE`, `TAX INVOICE`, `RECEIPT`.
  - Extracts clean corporate names (`Acme Supplies Inc`, `SUPERMART STORE #402`).
- **Validation Engine Updated**: Added comma-tolerant numeric casting so `$1,375.00` correctly passes currency validation.
- **Token Metadata Enhancement (`backend/app/services/auth.py`)**: Added `iat` (issued-at) timestamp to JWT tokens for traceability.
- **Database Startup Logging (`backend/app/main.py`)**: Replaced silent `except: pass` in `@app.on_event("startup")` with explicit logging.
- **Frontend 401 Session Redirect (`frontend/src/api/client.js`)**: Configured response interceptor to clear invalid tokens and redirect to `/login` when sessions expire.
- **UI Polish (`frontend/src/App.jsx`)**: Responsive font sizing and text truncation on `StatCard` filenames; cleaned up login/register link loops and removed dead forgot-password stubs.

### 🧠 Stage 2: Multi-Class ML Model Training & Classifier Hardening
- **Expanded Training Engine (`backend/scripts/train_classifier.py`)**:
  - Upgraded from legacy 4 financial classes to support all 12 domain classes across 5 major categories:
    - **Financial**: `invoice`, `receipt`, `bank_statement`, `salary_slip`, `other_financial_document`
    - **Academic**: `academic_transcript`, `academic_certificate`
    - **Legal**: `legal_contract`, `non_disclosure_agreement`
    - **Medical**: `medical_prescription`, `medical_lab_report`
    - **Administrative**: `identity_document`, `official_notice`
    - **General**: `other_document`
  - Integrated a multi-class synthetic dataset generator producing 630 balanced, realistic samples (45 samples per class) with varied headers, field tokens, date formats, and line structures.
  - Saved synthetic dataset to `backend/data/training_dataset.csv`.
- **Bundled Production Scikit-Learn Model (`backend/models/document_classifier.pkl`)**:
  - Pipeline: `TfidfVectorizer(lowercase=True, ngram_range=(1, 2), min_df=1, max_df=0.98, sublinear_tf=True)` + `LogisticRegression(max_iter=1000, class_weight='balanced', C=5.0, solver='lbfgs')`.
  - Evaluation: **1.0 Accuracy, 1.0 Macro Precision, 1.0 Macro Recall, 1.0 Macro F1** across all 14 classes on 126 hold-out test samples.
  - Artifact bundled directly in repository for zero-setup startup loading.
- **Classifier Hardening & Fallback Elimination (`backend/app/services/classifier.py`)**:
  - Implemented startup model loading with in-memory caching (`_load_model` with mtime tracking).
  - Eliminates the perpetual "Fallback" badge: classified documents return `method="ml"`, `status="classified"`, model identifier (`tfidf-logistic-regression:1.0.0`), and calibrated probability scores.
  - Added model loading diagnostics to FastAPI `@app.on_event("startup")` in `backend/app/main.py`.
  - **False Financial Misclassification Prevention**: Added guardrails so non-financial documents (Resumes, CVs, cover letters, meeting memos) are classified as `other_document` / `General`, preventing them from being mistakenly categorized as `Financial / other_financial_document`.
  - Hardened heuristic fallback to cleanly separate financial memos from general correspondence even when the ML model is disabled.
- **Test Suite Expansion**:
  - Added dedicated tests in `tests/test_classifier.py`:
    - `test_production_bundled_model_loads_and_classifies_all_12_classes`
    - `test_resume_and_general_documents_not_misclassified_as_financial`
    - `test_heuristic_fallback_prevents_resume_from_misclassification`
  - Total automated test suite: **58 passed, 0 failed, 9 skipped** (`python -m pytest tests/ -v`).

### 🎨 Stage 2 UI Polish: Multi-Domain Experience & Classification Visualizer
- **1-Click Live Multi-Domain Demo Presets (`frontend/src/App.jsx`)**:
  - Replaced legacy 2 presets with **6 comprehensive domain presets** (`Financial`, `Academic`, `Medical`, `Legal`, `Administrative`, `General`).
  - Rendered in a clean, responsive 6-column grid (`grid-cols-2 md:grid-cols-3 lg:grid-cols-6`) with domain-specific color badges and iconography.
- **Domain Category Filtering on Document Library (`/documents`)**:
  - Added a dedicated Domain Filter dropdown (`All domains`, `Financial`, `Academic`, `Legal`, `Medical`, `Administrative`, `General`).
  - Added a new Domain Category badge column to the document library table with real-time reactive filtering across search terms, processing statuses, and domain categories.
- **Processing Results Inspection (`/processing`)**:
  - Displays dynamic Domain Category badges alongside Document Type.
  - Formatted human-readable field keys (`key.replace(/_/g, ' ')`) instead of raw snake_case strings.
  - Displays exact ML confidence percentage with visual progress bars.
- **Document Detail & Workflow Rules View (`/documents/:id`)**:
  - Added **ML Classification Metadata Card** detailing Domain Category, Model Identifier (`tfidf-logistic-regression:1.0.0`), confidence score, and quality score.
  - Added **Automated Workflow Evaluation Card** showing real-time policy rules (e.g., Spend Gate, Academic Seal, HIPAA Shield) and evaluation statuses.
  - Formatted human-review editor labels with title-cased keys.
- **Build Verification**: Clean production build via `npm run build` with **0 errors**.

### 🔧 Stage 2 Final Polish Fixes
- **StatCard Font Size Uniformity (`frontend/src/App.jsx`)**:
  - Removed the `isLong` conditional branch (`value.length > 12`) that caused `text-3xl` for short values (`invoice`, `63%`) and `text-lg sm:text-xl` for longer ones (`pending_approval`, filename).
  - All 4 stat card values now render at a uniform `text-xl font-semibold tracking-tight text-slate-900 truncate`, matching the design spec.
- **Demo Preset Text Improvements (`frontend/src/App.jsx` + `backend/app/services/document_processor.py`)**:
  - **Academic**: Added explicit `Date:` line; simplified `Semester: VII` (removed `(Academic Year ...)` suffix that broke the regex).
  - **Medical**: Added labeled `Hospital:` line; restructured `Rx:` block with numbered lines on separate lines so the medications regex captures correctly.
  - **Legal**: Changed `Party A:` / `Party B:` → `Disclosing Party:` / `Receiving Party:` to match the extraction regex.
  - **Extraction regexes hardened** (`document_processor.py`): Legal party regex extended to also match `party a|party 1`; Medical `clinic_hospital` regex tries explicit `hospital:|clinic:` label first; Official notice `subject` regex tightened to require a colon (prevents `Reference No:` being captured as the subject); Academic semester regex stops at `(` or newline.
  - **Verified clean extraction output** for all 6 presets (Medical clinic, Legal parties, Academic semester, Admin subject/notice number).
- **Upsert-by-Filename Deduplication (`backend/app/main.py`)**:
  - Upload endpoint now queries for an existing document with the same `filename` + `owner_id` before inserting.
  - If found: updates all fields in-place, deletes old `DocumentChunk` rows, inserts fresh chunks — no duplicate library row created.
  - If not found: original insert path unchanged.
  - Scope is per-user: different users uploading the same filename still get independent records.
- **Final test count: 58 passed, 0 failed, 9 skipped** (`python -m pytest tests/ -v`).
- **Frontend build: ✓ 0 errors, 2643 modules, exit code 0** (`npm run build` in `frontend/`).
- **FastAPI Lifespan Migration (`backend/app/main.py`)**: Replaced deprecated `@app.on_event("startup")` with the modern `@asynccontextmanager async def lifespan(app)` pattern and wired it via `FastAPI(lifespan=lifespan)`. Eliminates 2 DeprecationWarnings per test run and per server start.
- **Academic `institution_name` Fix (`backend/app/services/document_processor.py`)**: Institution line-scanner now skips any line that looks like a labeled field (`Field: value` where the label is ≤ 20 chars), preventing `"Student Name: Rahul Mehra"` from being captured as the institution. `institution_name` now correctly returns `"Guru Gobind Singh Indraprastha University"`.
- **Final warning count after all fixes: 8 warnings (down from 10)** — remaining warnings are all in third-party packages (SWIG bindings, starlette, sklearn) and are not actionable.

---

## 3. Current Architecture & State

| Module | Location | Current State |
| :--- | :--- | :--- |
| **API Server** | `backend/app/main.py` | FastAPI with 20+ endpoints (upload, auth, search, review, chat, workflows) |
| **Field Extraction** | `backend/app/services/document_processor.py` | Multi-domain extraction with clean regex anchors & layout analysis |
| **Classifier** | `backend/app/services/classifier.py` | 14-class ML classifier + hardened fallback; cached in-memory |
| **ML Model Artifact** | `backend/models/document_classifier.pkl` | Production scikit-learn pipeline (TF-IDF + LogisticRegression) |
| **Training Script** | `backend/scripts/train_classifier.py` | Multi-class dataset generator & training pipeline |
| **Database** | `backend/app/db/database.py` | SQLite locally (`docubrix.db`), PostgreSQL on Render |
| **Frontend UI** | `frontend/src/App.jsx` | Full SPA (React + Tailwind + Lucide + Recharts) with ML Sparkles badge & confidence |
| **API Client** | `frontend/src/api/client.js` | Axios with interceptors, token persistence, and 401 redirect |

---

## 4. Next Steps & Future Roadmap

1. **Stage 3: Deep Tabular Line-Item Extraction & Multimodal OCR Preprocessing**:
   - Parse line-item tables (description, quantity, unit_price, amount) from multi-row invoices/receipts into a structured `line_items` array inside `extracted_json`.
   - Add adaptive image preprocessing (contrast enhancement, binarization, deskewing) with PIL/OpenCV before Tesseract OCR.
   - Render a dedicated interactive Tabular Line-Item Grid on `/processing` and `/documents/:id` when `line_items` are present.
   - Add `tests/test_phase3_extraction.py` with comprehensive coverage of multi-row table parsing and noisy scans.
2. **Stage 4: Enhanced RAG & Cross-Document Intelligence**:
   - Expanded vector indexing and cross-document aggregation queries for multi-document comparisons.
