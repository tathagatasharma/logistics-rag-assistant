# Logistics Knowledge Assistant — RAG on Databricks

A fully **Databricks-native** Retrieval-Augmented Generation (RAG) pipeline that lets you ask natural language questions against a corpus of logistics documents (shipping guides, GS1 standards, supply chain research papers) and receive cited, grounded answers.

---

## Architecture

```
PDF Documents (Unity Catalog Volume)
        ↓
Text Extraction + Chunking (PyMuPDF + tiktoken)
        ↓
Delta Table  ──► Vector Search Index (auto-embeddings via BGE-Large)
                        ↓
              Semantic Retrieval (top-k chunks)
                        ↓
           LLM Generation (Meta Llama 3.3 70B)
                        ↓
          Grounded Answer with Citations
```

Every component runs on the Databricks native stack — no external vector databases, no third-party embedding services.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Data governance | Unity Catalog |
| File storage | UC Volumes |
| Chunk storage | Delta Tables |
| Embeddings | Databricks BGE-Large-EN (Foundation Model API) |
| Vector search | Databricks Vector Search (Delta Sync) |
| LLM | Meta Llama 3.3 70B Instruct (Foundation Model API) |
| Orchestration | Databricks Notebook |

---

## Notebook Sections

| Section | What it does |
|---|---|
| 1 — UC Setup | Creates catalog, schema, and volume for PDF storage |
| 2 — PDF Ingestion | Extracts text, chunks at ~900 tokens with 120-token overlap, writes to Delta |
| 3 — Vector Search | Provisions endpoint and Delta Sync index with managed embeddings |
| 4 — Retrieval | Semantic similarity search returning top-k chunks |
| 5 — Generation | Grounded answer synthesis with strict citation instructions |
| 6 — Testing | End-to-end Q&A examples including out-of-scope boundary tests |

---

## Prerequisites

- Databricks workspace with **Unity Catalog** enabled
- Access to **Foundation Model APIs** (check your Serving tab)
- PDF documents uploaded to the UC Volume (path configured in Section 1)

---

## Quick Start

1. Clone or import `Logistics Knowledge Assistant RAG.ipynb` into your Databricks workspace
2. Upload your logistics PDFs to the configured Volume path
3. Verify the embedding and LLM endpoint names in Sections 3 and 5
4. Run sections **1 → 2 → 3 → 4 → 5 → 6** in order

---

## Knowledge Base

The notebook was built and tested against 16 logistics documents spanning:

- **Carrier guides**: FedEx Freight Shipping Guide, UPS Packaging Guidelines
- **GS1 standards**: Logistic Label Guideline, Global Traceability Standard, Traceability Checklist
- **Research papers**: AI/ML in supply chain, demand forecasting, last-mile delivery, warehouse automation, LLM agents in supply chain, and more

---

## Key Design Decisions

**Chunking**: 900-token chunks with 120-token overlap balances retrieval precision against context richness. Token-based (not character-based) sizing aligns with how LLMs process text.

**Delta Sync**: Databricks auto-embeds the chunk_text column whenever the source Delta table updates — no manual embedding pipeline to maintain.

**Strict grounding**: The generation prompt explicitly instructs the LLM to answer only from retrieved context and say "I don't have that information" when context is insufficient, minimising hallucination.

---

## Customisation

- Adjust chunk_size and overlap in Section 2 for different document types
- Change k (number of retrieved chunks) in Sections 4–6
- Switch pipeline_type to CONTINUOUS in Section 3 for real-time index updates
- Add metadata fields (doc_date, doc_type) to the Delta table for filtered retrieval
- Wrap generate_grounded_answer() in a Model Serving endpoint for production use
