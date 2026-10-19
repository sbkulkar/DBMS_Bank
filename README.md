# GraphRAG-Based Banking Compliance & Risk Intelligence System

An M.Tech engineering project implementing an intelligent document ingestion, graph knowledge preparation, dynamic risk calculation, and hybrid GraphRAG retrieval pipeline for banking compliance and fraud/AML risk intelligence.

---

## 1. Project Overview & Architecture

The system connects:
1. **Official Banking Regulations & Directives** (RBI Master Directions on KYC, Cybersecurity, Credit Information, Interest Rates, Operational Guidelines, Supervisory Credit, FATF 40 Recommendations, Annual Reports, Research Working Papers).
2. **KYC and AML Compliance Obligations** (Customer Due Diligence, Politically Exposed Persons, Suspicious Transaction Reporting).
3. **Investigation Signals & Typologies** (Pass-through mule accounts, rapid fund drains, circular transaction loops, unauthorized UPI bursts, account takeover, expired KYC operations, loan diversion).
4. **Knowledge Graph Representation** (Nodes: `Document`, `RegulatorySection`, `EvidenceChunk`, `Customer`, `Account`, `Transaction`, `RiskIndicator`, `FraudPattern`).
5. **Dynamic Risk Calculation Engine** (Observable mathematical and temporal reasoning over graph records: fan-in count, drain velocity, round-tripping cycles, turnover multiples).
6. **Dual-Mode Answer Generation** (Live LLM synthesis via Gemini/OpenAI REST API, or deterministic grounded synthesis engine when operating offline).
7. **Exact Source Attribution** (Verifiable citations down to document filename, page number, section heading, and regulatory authority level).

```
DBms_project/
├── data/
│   ├── [10 Original PDF documents preserved]
│   ├── processed/            # Page-level JSON extractions & 1,220 master chunks.json
│   ├── metadata/             # Document inventory & extraction manifests
│   └── synthetic/            # 7 benchmark customer scenarios with accounts & txs
├── ingestion/
│   ├── inspect_documents.py  # Phase 1: Inspects PDFs, detects OCR status, computes hashes
│   ├── extract_text.py       # Phase 3: Page-level text extraction with PyMuPDF
│   ├── chunk_documents.py    # Phase 3: Semantic paragraph chunking with metadata
│   └── build_graph.py        # Phase 5: Neo4j idempotent knowledge graph builder
├── graph/
│   ├── constraints.cypher    # Uniqueness constraints & performance indexes
│   └── schema.cypher         # Complete Neo4j schema & sample queries
├── retrieval/
│   ├── document_retriever.py # Phase 6B: BM25 retriever with acronym expansion & authority mapping
│   ├── graph_retriever.py    # Phase 6A/C/D: Hybrid graph traversal & full pipeline orchestrator
│   ├── risk_analyzer.py      # Dynamic graph reasoning engine (fan-in, drain, circular, KYC)
│   └── answer_generator.py   # Grounded 8-section report generator (LLM API & deterministic)
├── evaluation/
│   └── test_questions.json   # Benchmark evaluation questions with citation targets
├── reports/
│   ├── document_inventory_report.md
│   ├── extraction_report.md
│   └── chunking_report.md
├── tests/
│   └── test_pipeline.py      # Automated unit & integration test suite (13 tests)
├── app.py                    # Streamlit interactive UI application
├── .env.example              # Environment variables template
├── requirements.txt          # Python dependencies
└── README.md                 # Project documentation
```

---

## 2. Prerequisites & Environment Setup

- **Python**: Python 3.10+ (tested on Python 3.13)
- **Neo4j**: Live Neo4j Sandbox, AuraDB, or local Neo4j Desktop / Docker instance. *(If offline, local index fallback ensures full functionality).*

### Step 1: Install Python Dependencies
```powershell
pip install -r requirements.txt
```

### Step 2: Configure Environment Variables
Copy `.env.example` to `.env` and fill in your Neo4j credentials:
```powershell
Copy-Item .env.example .env
```

Example `.env` configuration:
```env
NEO4J_URI=bolt://<sandbox-ip>:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=<sandbox-password>
NEO4J_DATABASE=neo4j

# Optional: Add external LLM key if desired (pipeline automatically defaults to deterministic synthesis if omitted)
GEMINI_API_KEY=
OPENAI_API_KEY=
```

---

## 3. Synthetic Benchmark Scenarios

The synthetic dataset located in `data/synthetic/` contains 7 realistic banking customer scenarios modeling key financial risk and compliance typologies:

| Customer ID | Customer Name | Customer Type | Primary Typology | Observed Anomaly |
| :--- | :--- | :--- | :--- | :--- |
| `CUST_SYNTH_101` | **Arjun Sharma** | INDIVIDUAL | Money Mule / Rapid Dispersal | 4 sub-threshold deposits totaling INR 1.96L rapidly drained (99.2%) via NEFT in 20 mins |
| `CUST_SYNTH_102` | **Priya Patel** | INDIVIDUAL | Unauthorized UPI Transactions | 4 rapid nighttime micro-transactions (INR 4.5k to 9.8k) within 3 minutes |
| `CUST_SYNTH_103` | **Vikram Mehta** | INDIVIDUAL | Account Takeover (ATO) | Credential change notification followed by INR 450k IMPS transfer outside normal profile |
| `CUST_SYNTH_104` | **Apex Global Consultancy** | CORPORATE | CDD Gaps & Excessive Turnover | Expired KYC status with INR 8.5M wire from high-risk overseas shell jurisdiction (7x declared income) |
| `CUST_SYNTH_105` | **Bharat Tech Innovations** | CORPORATE | Expired KYC Active Operations | KYC expired 9 months ago; ongoing active commercial credit facilities totaling INR 1.2M |
| `CUST_SYNTH_106` | **Zenith Global Exim** | CORPORATE | Circular Routing / Round-Tripping | Closed fund cycle: `ACC_A -> ACC_B -> ACC_C -> ACC_A` (INR 5.0M round-tripping) |
| `CUST_SYNTH_107` | **BlueStar Infrastructure** | CORPORATE | Loan Diversion / Documentation Gaps| Term loan disbursement of INR 15M diverted to third-party vendor rather than declared project capital |

---

## 4. Execution Commands

### A. Run Interactive Streamlit UI
Launch the interactive web application to explore scenarios, inspect graph connections, view regulatory citations, and generate reports:
```powershell
streamlit run app.py
```
*Access via browser at `http://localhost:8501`.*

### B. Command-Line Retrieval & Grounded Answer Generation
Run an end-to-end investigation combining Neo4j graph traversal, dynamic risk reasoning, BM25 retrieval, and answer generation:
```powershell
# Investigate by customer name (strips fictional suffix automatically)
python retrieval/graph_retriever.py --customer-name "Arjun Sharma"

# Investigate by customer ID
python retrieval/graph_retriever.py --customer-id CUST_SYNTH_104

# Print raw grounding prompt & context for inspection / debugging
python retrieval/graph_retriever.py --customer-name "Apex Global Consultancy" --prompt-only

# Filter regulatory documents to KYC direction only
python retrieval/graph_retriever.py --customer-id CUST_SYNTH_104 --document-filter kyc.pdf
```

### C. Standalone BM25 Document Retrieval
Search official regulatory PDFs with banking acronym expansion and authority level mappings:
```powershell
python retrieval/document_retriever.py --query "enhanced due diligence high risk third party" --top-k 3
python retrieval/document_retriever.py --query "unauthorized electronic banking transactions customer liability" --top-k 3
```

### D. Idempotent Neo4j Graph Ingestion
Verify constraints and load regulatory chunks and synthetic scenarios into Neo4j:
```powershell
# Ingest regulatory evidence chunks and synthetic scenarios idempotently
python ingestion/build_graph.py --include-synthetic

# Dry-run validation without writing to database
python ingestion/build_graph.py --dry-run
```

### E. Run Automated Test Suite
Execute the full 13-point automated test suite covering extraction, chunking, Cypher constraints, customer lookups, safe unknown handling, dynamic risk calculations, synthetic scenarios, and grounded report generation:
```powershell
python -m unittest tests/test_pipeline.py -v
```

---

## 5. Output Report Structure

Every generated answer adheres to a standardized, 8-section investigation report:
1. **Situation Summary**: Inquiry context, customer entity, risk classification, and evidence counts.
2. **Verified Facts from Knowledge Graph**: Hard graph facts (accounts, balances, transaction records, PEP status).
3. **Identified Risk Indicators & Typology Analysis**: Stored typology definitions and mathematically calculated risk signals (fan-in, drain velocity, circular loops).
4. **Relevant Regulatory Provisions & Citations**: Verifiable citations (Document, Page, Section heading, Authority level, BM25 heuristic score).
5. **Plain-Language Explanation of Financial Risk**: Executive explanation of potential commercial and compliance impact.
6. **Suggested Investigation Steps**: Actionable investigation steps (source of funds inquiry, EDD, velocity caps, STR filing).
7. **Missing Evidence & Alternative Explanations**: Honest assessment of unobserved external facts (beneficiary verification, commercial contracts, customer disputes).
8. **Limitations & Regulatory Disclaimers**: Explicit notice of synthetic benchmark data, non-criminal attribution, and absence of fabricated citations.

---

## 6. Verification Test Suite Status

The test suite in `tests/test_pipeline.py` verifies 13 critical components:
- [x] **PDF extraction & page tracking**: Tracks all 101 pages of `kyc.pdf` and 807 pages across all 10 PDFs.
- [x] **Chunk metadata preservation**: Asserts chunk IDs, filenames, page ranges, section headings, and character spans.
- [x] **Duplicate handling**: Validates SHA256 chunk normalization preventing duplicate chunks.
- [x] **Neo4j schema & constraints**: Confirms uniqueness constraints for `Document`, `EvidenceChunk`, `Customer`, `Account`, `Transaction`.
- [x] **Graph relationships & synthetic integrity**: Confirms accounts and transactions connect to valid customers.
- [x] **Source citation availability**: Confirms retrieval produces verifiable citations (filename + page number).
- [x] **OCR detection logic**: Confirms image-heavy zero-character pages trigger OCR recommendations.
- [x] **Customer lookup & suffix handling**: Tests exact match, suffix stripping (`(Fictional Entity)`), and ID lookups.
- [x] **Unknown customer safe handling**: Confirms non-existent customers return `None` without crashing or selecting arbitrary records.
- [x] **Dynamic risk calculations**: Validates high fan-in, rapid drain ratio, circular routing cycle detection, and KYC non-compliance flags.
- [x] **Extended synthetic scenarios**: Asserts all 7 customers and 7 typologies are populated and coherent.
- [x] **Answer generation structure**: Verifies all 8 sections and synthetic disclaimers are present in synthesized reports.
- [x] **Acronym expansion & authority mapping**: Confirms acronym expansion (PEP, AML, KYC, STR, EDD) and regulatory authority tagging.
