# UnderwriteOS — AI Mortgage Underwriting Assistant

**UnderwriteOS** is an agentic mortgage underwriting decision-support system designed to streamline the review of multi-document borrower loan packages.

The system combines document processing, guideline retrieval, deterministic financial calculations, discrepancy detection, and LLM-assisted underwriting analysis to produce evidence-backed findings and draft underwriting conditions.

> **Human-in-the-Loop Safeguard**
>
> UnderwriteOS is a **decision-support system**, not an autonomous lending decision-maker. It does not issue final loan approvals, binding denials, adverse action notices, or automated clear-to-close decisions. Final underwriting decisions remain with a qualified human underwriter.

---

## Overview

Mortgage underwriting often requires reviewing information spread across multiple documents, including applications, paystubs, W-2s, bank statements and credit information.

UnderwriteOS brings these tasks into a single workflow:

1. Ingest borrower documentation.
2. Extract and reconcile relevant financial facts.
3. Retrieve applicable underwriting guidelines using RAG.
4. Perform financial calculations using deterministic Python logic.
5. Identify discrepancies and documentation gaps.
6. Generate evidence-backed underwriting findings.
7. Produce draft conditions for human review.

The architecture intentionally separates **LLM reasoning from deterministic financial calculations** to improve reliability and traceability.

---

## Architecture

```text
                 ┌──────────────────────────────┐
                 │   Borrower Loan Package      │
                 │      ZIP / Multiple Files    │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │ 1. Document Processing       │
                 │                              │
                 │ Ingestion                    │
                 │ Normalisation                │
                 │ Fact Extraction              │
                 └──────────────┬───────────────┘
                                │
                   ┌────────────┴────────────┐
                   │                         │
                   ▼                         ▼
        ┌─────────────────────┐   ┌─────────────────────┐
        │ 2. Guideline RAG    │   │ 3. Calculation      │
        │                     │   │    Engine            │
        │ FAISS / Semantic    │   │                     │
        │ Search              │   │ Deterministic       │
        │                     │   │ Python Calculations │
        └──────────┬──────────┘   └──────────┬──────────┘
                   │                         │
                   └────────────┬────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │ 4. Underwriting Assessment   │
                 │                              │
                 │ Policy Evaluation            │
                 │ Risk Findings                │
                 │ Draft Conditions             │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │ 5. Evidence & Citation       │
                 │    Review                    │
                 │                              │
                 │ Source Validation             │
                 │ Traceability                  │
                 │ Auditability                  │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │ Underwriting Report / HUD    │
                 └──────────────────────────────┘
```

---

## Core Modules

| Module                      | Responsibility                                                                                                                   |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `src/document_processor.py` | Ingests borrower packages including PDF, TXT, MD and ZIP files; normalises text and extracts financial facts.                    |
| `src/guideline_rag.py`      | Retrieves relevant underwriting guidelines using FAISS semantic search with TF-IDF fallback.                                     |
| `src/calculations.py`       | Performs deterministic calculations including DTI, monthly income, housing expenses and closing funds.                           |
| `src/discrepancy_engine.py` | Reconciles information across applications, W-2s, paystubs and bank statements to identify discrepancies and documentation gaps. |
| `src/underwriting.py`       | Orchestrates the underwriting assessment and generates draft conditions from policy rules and findings.                          |
| `src/llm.py`                | Handles Groq API integration and structured LLM reasoning.                                                                       |
| `src/database.py`           | Provides SQLite persistence for underwriting runs and audit information.                                                         |
| `src/models.py`             | Defines Pydantic models for structured application state and auditability.                                                       |

---

## Project Structure

```text
underwriterAI/
│
├── app.py
│
├── data/
│   ├── demo/
│   │   ├── 01_application.txt
│   │   ├── ...
│   │   ├── 14_application_correction.txt
│   │   ├── initial_package/
│   │   └── revised_package/
│   │
│   ├── guidelines/
│   └── underwriter.db
│
├── src/
│   ├── document_processor.py
│   ├── guideline_rag.py
│   ├── calculations.py
│   ├── discrepancy_engine.py
│   ├── underwriting.py
│   ├── llm.py
│   ├── database.py
│   └── models.py
│
├── tests/
│
├── .env.example
├── .gitignore
├── LICENSE
├── packages.txt
├── pytest.ini
├── requirements.txt
└── README.md
```

---

## Quickstart

### Requirements

* Python 3.12
* Git
* Groq API key

### 1. Clone

```bash
git clone https://github.com/sobanmujtaba/underwriterAI.git
cd underwriterAI
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

For dense vector retrieval:

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

> The included policy data and borrower scenarios are synthetic and intended for demonstration and testing.

### 5. Run Tests

```bash
pytest tests/
```

### 6. Launch the Application

```bash
streamlit run app.py
```

The Streamlit application will be available at:

```text
http://localhost:8501
```

---

## Demonstration Scenarios

The repository includes synthetic borrower packages for demonstrating discrepancy detection and condition resolution.

### Scenario A — Discrepancy Detection

Package:

```text
data/demo/initial_package/
```

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

The system demonstrates detection of:

* **Base income mismatch**

  * Stated income: **$8,500/month**
  * Verified paystub income: **$8,000/month**

* **Bonus income documentation gap**

  * Required 24-month historical documentation is missing.

* **Unexplained deposit**

  * A **$15,000** bank deposit lacks an acceptable source.

These findings are converted into draft underwriting conditions for human review.

---

### Scenario B — Condition Resolution

Package:

```text
data/demo/revised_package/
```

Upload:

```text
data/demo/revised_package/loan_file.txt
```

or include:

```text
13_savings_statements.txt
14_application_correction.txt
```

The revised package demonstrates resolution of the previously identified issues:

* Corrected base income: **$8,000.00/month**
* Bonus income excluded from qualifying income
* $15,000 deposit sourced to verified personal savings
* Total monthly debt obligations: **$3,350.00**

The deterministic calculation engine produces:

```text
$3,350 / $8,000 × 100 = 41.875%
```

**Illustrative DTI: 41.875%**

---

## Key Design Principles

### Deterministic Financial Calculations

Financial arithmetic is intentionally handled outside the LLM.

The calculation engine is responsible for:

* Base income calculations
* Debt-to-income ratios
* Housing expense calculations
* Closing-fund calculations

This helps prevent arithmetic errors that can occur when financial calculations are delegated to an LLM.

### Retrieval-Augmented Guideline Validation

The RAG layer retrieves relevant policy material before the underwriting assessment, allowing findings to be evaluated against the applicable synthetic guideline context.

### Evidence-Based Findings

Underwriting findings are designed to remain traceable to the underlying borrower documents and retrieved policy guidance.

### Human Oversight

The system provides underwriting **decision support**, not autonomous underwriting.

A qualified human underwriter remains responsible for reviewing the evidence, validating findings and making the final lending decision.

---

## Technology Stack

| Technology                | Purpose                               |
| ------------------------- | ------------------------------------- |
| **Python 3.12**           | Core application                      |
| **Streamlit**             | Web interface                         |
| **Groq API**              | LLM inference                         |
| **Llama 3.3 70B**         | Default reasoning model               |
| **FAISS**                 | Vector similarity search              |
| **Sentence Transformers** | Embeddings                            |
| **TF-IDF**                | Retrieval fallback                    |
| **Pydantic**              | Data validation and structured models |
| **SQLite**                | Persistence and audit tracking        |
| **Pytest**                | Testing                               |

---

## Attribution

This repository is based on the original **Mortgage Underwriting Assistant / UnderwriteOS** project created by **Abdur Rehman**.

Original project:

**Abdur Rehman**
`https://github.com/abdur-rahman-tech/mortgage-underwriting-assistant`

This repository represents my own version, adaptation and development of the project for my portfolio and experimentation.

Credit and attribution are retained to acknowledge the original creator and source project.

---

## Disclaimer

UnderwriteOS is a **demonstration and decision-support system**.

The included borrower documents, underwriting policies and financial scenarios are synthetic. The system is not intended to replace a qualified mortgage underwriter, lender compliance process, legal review or applicable regulatory requirements.

No output from this application should be interpreted as a final credit decision, loan approval, denial, adverse action notice or clear-to-close determination.

---

## License

Distributed under the **MIT License**.

See [`LICENSE`](LICENSE) for details.
