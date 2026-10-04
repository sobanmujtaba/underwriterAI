# UnderwriteOS — AI Mortgage Underwriting Assistant

**UnderwriteOS** is an agentic decision-support system designed to reduce mortgage underwriting review time.

The platform processes multi-document borrower loan packages, extracts and reconciles financial facts, retrieves applicable underwriting guidelines using RAG, delegates financial calculations to a deterministic Python engine, and generates evidence-backed draft underwriting conditions.

> **Human-in-the-Loop Safeguard**
>
> UnderwriteOS provides **decision-support findings and actionable underwriting conditions**. It does **not** issue final loan approvals, binding denials, adverse action notices, or automated clear-to-close decisions. Final underwriting decisions remain with a qualified human underwriter.

---

## Architecture & Workflow

```text
┌─────────────────────────────────────────────────────────────┐
│              Borrower Loan Package Upload                  │
│                  ZIP / Multiple Files                      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              1. Document Review Processing                  │
│       Ingestion • Normalisation • Fact Extraction           │
└─────────────────────────────┬───────────────────────────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
┌────────────────────────────┐  ┌────────────────────────────┐
│ 2. Guideline RAG           │  │ 3. Calculation Engine     │
│                            │  │                            │
│ FAISS / Semantic Search    │  │ Deterministic Python      │
│ Policy Retrieval           │  │ Financial Calculations    │
└──────────────┬─────────────┘  └──────────────┬─────────────┘
               │                               │
               └───────────────┬───────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              4. Underwriting Assessment                    │
│       Policy Evaluation • Risk Findings • Conditions       │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              5. Evidence & Citation Review                 │
│       Source Validation • Traceability • Auditability      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│          Underwriting Transmittal Report & HUD             │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Modules

| Module                      | Responsibility                                                                                                                                      |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/document_processor.py` | Ingests borrower packages including PDF, TXT, MD and ZIP files; normalises text and extracts relevant financial facts.                              |
| `src/guideline_rag.py`      | Performs semantic underwriting guideline retrieval using FAISS vector search, with TF-IDF fallback and metadata-aware chunking.                     |
| `src/calculations.py`       | Deterministic financial calculation engine for DTI, base monthly income, housing expense ratios and required closing funds.                         |
| `src/discrepancy_engine.py` | Reconciles information across applications, W-2s, paystubs and bank statements to identify mismatches, unexplained deposits and documentation gaps. |
| `src/underwriting.py`       | Orchestrates the underwriting assessment and synthesises policy rules, extracted facts and discrepancies into draft conditions.                     |
| `src/llm.py`                | Integrates with the Groq API, using `llama-3.3-70b-versatile` by default for structured reasoning and assessment.                                   |
| `src/database.py`           | Provides SQLite persistence for underwriting runs and audit information.                                                                            |
| `src/models.py`             | Defines Pydantic models used for structured state management and auditability.                                                                      |

---

## Directory Structure

```text
.
├── app.py                         # Streamlit application and executive HUD
│
├── data/
│   ├── demo/                      # Synthetic test packages and borrower documents
│   │   ├── 01_application.txt
│   │   ├── ...
│   │   ├── 14_application_correction.txt
│   │   ├── initial_package/       # Package containing known discrepancies
│   │   └── revised_package/       # Package resolving identified conditions
│   │
│   ├── guidelines/                # Underwriting rules and policy documents
│   └── underwriter.db             # Persistent audit database
│
├── src/                           # Application modules and engines
│
├── tests/                         # Unit and integration tests
│
├── packages.txt                   # System-level dependencies
├── pytest.ini                     # Pytest configuration
├── requirements.txt               # Python dependencies
└── README.md
```

---

## Quickstart

### Prerequisites

* Python **3.12**
* A **Groq API key**
* Git

### 1. Clone the Repository

```bash
git clone https://github.com/abdur-rahman-tech/mortgage-underwriting-assistant.git
cd mortgage-underwriting-assistant
```

### 2. Create a Virtual Environment

```bash
python3.12 -m venv venv
source venv/bin/activate
```

On Windows:

```powershell
venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

For dense vector search, install the optional FAISS and Sentence Transformers dependencies:

```bash
pip install faiss-cpu sentence-transformers
```

### 4. Configure Environment Variables

Create a `.env` file in the project root:

```ini
GROQ_API_KEY=your_groq_api_key_here
GROQ_MODEL_NAME=llama-3.3-70b-versatile

POLICY_PROGRAM=synthetic_conventional_salaried
POLICY_EFFECTIVE_DATE=2026-10-01
```

> **Note:** The included policy data is synthetic and intended for demonstration and testing purposes.

### 5. Run the Test Suite

```bash
pytest tests/
```

### 6. Launch the Application

```bash
streamlit run app.py
```

The application will be available at:

```text
http://localhost:8501
```

---

## Demonstration Scenarios

The `data/demo/` directory contains synthetic borrower packages designed to demonstrate discrepancy detection and condition resolution.

### Scenario A — Discrepancy Detection

**Package:** `data/demo/initial_package/`

Upload:

```text
data/demo/initial_package/loan_file.txt
```

or the individual fixtures:

```text
01_application.txt
...
07_credit_report_summary.txt
```

#### Expected Behaviour

UnderwriteOS identifies several underwriting issues:

* **Base income mismatch**

  * Stated income: **$8,500/month**
  * Verified paystub income: **$8,000/month**

* **Bonus income documentation gap**

  * Required 24-month historical documentation is missing.

* **Unexplained deposit**

  * A **$15,000** bank deposit lacks an acceptable source.

The system converts these findings into evidence-backed draft underwriting conditions.

---

### Scenario B — Condition Resolution

**Package:** `data/demo/revised_package/`

Upload:

```text
data/demo/revised_package/loan_file.txt
```

or include:

```text
13_savings_statements.txt
14_application_correction.txt
```

#### Expected Behaviour

The revised package resolves the previously identified discrepancies:

* Corrected base income is validated at **$8,000.00/month**.
* Bonus income is excluded from qualifying income.
* The **$15,000 deposit** is sourced to verified personal savings.
* Total monthly debt obligations are verified at **$3,350.00**.
* Deterministic illustrative DTI is calculated as:

```text
$3,350 / $8,000 × 100 = 41.875%
```

**Illustrative DTI: 41.875%**

---

## Design Principles

### Deterministic Financial Calculations

Financial calculations are deliberately separated from LLM reasoning.

The calculation engine performs operations such as:

* Base income calculations
* Debt-to-income ratio calculations
* Housing expense calculations
* Closing-fund calculations

This reduces the risk of an LLM producing arithmetic errors.

### Evidence-Based Findings

Underwriting findings are intended to be traceable to source documents and retrieved policy guidance rather than being generated solely from model assumptions.

### Retrieval-Augmented Guideline Validation

The RAG layer retrieves relevant policy material before underwriting assessment, allowing the system to associate findings and conditions with the applicable synthetic guideline context.

### Human Oversight

The system is designed as **decision support**, not autonomous underwriting.

A human underwriter remains responsible for reviewing the evidence, validating the findings and making the final lending decision.

---

## Technology Stack

* **Python 3.12**
* **Streamlit** — Web application interface
* **Groq API** — LLM inference
* **Llama 3.3 70B** — Default reasoning model
* **FAISS** — Vector similarity search
* **Sentence Transformers** — Embeddings
* **TF-IDF** — Retrieval fallback
* **Pydantic** — Structured data models
* **SQLite** — Run and audit persistence
* **Pytest** — Automated testing

---

## Disclaimer

UnderwriteOS is a **demonstration and decision-support system**.

The included borrower documents, underwriting policies and financial scenarios are synthetic. The system is not intended to replace a qualified mortgage underwriter, lender compliance process, legal review or applicable regulatory requirements.

No output from UnderwriteOS should be interpreted as a final credit decision, approval, denial, adverse action notice or clear-to-close determination.

---

## License

Distributed under the **MIT License**.
