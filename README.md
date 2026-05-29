# Evaluating Advanced RAG Architectures for Cybersecurity Incident Response

## Overview

This repository contains the complete implementation, automation scripts, and evaluation suites for a secure, localized Retrieval-Augmented Generation (RAG) pipeline optimized for cybersecurity Incident Response (IR).

The architecture is designed to allow a Security Operations Center (SOC) to securely ingest authoritative documentation (e.g., NIST, CISA, SANS) and generate real-time remediation paths without relying on third-party cloud APIs. The evaluation suite programmatically benchmarks three retrieval strategies (**Naive RAG**, **Hybrid Search**, and **Advanced Cross-Encoder Reranking**) using a reference-free LLM-as-a-Judge framework powered by RAGAS and DeepEval.

---

## Repository Structure

```text
├── Data/
│   └── Incident_Response_Knowledge_Base/  # Structured IR playbooks (PDFs/JSONs)
├── Notebooks/
│   └── RAG_Design_almost_Final.ipynb      # Main execution and evaluation suite
├── evaluation_results/                    # Auto-generated CSVs and performance graphs
├── requirements.txt                       # Python dependencies
└── README.md                              # This documentation file
```

## Prerequisites & System Requirements

This pipeline strictly utilizes localized, open-source models to simulate an air-gapped environment. Ensure your hardware and environment meet the following specifications:

- Python Runtime: Version 3.10 or higher.

- Ollama Local Server: Required to host the generation and evaluation model locally.

- Compute Minimums: A100 Colab GPU to ensure consistent Ollama running without session expiration.

## Step-by-Step Reproduction Guide

### 1. Data

Consolidate your raw cybersecurity playbook documents into a single folder (e.g., Incident_Response_Knowledge_Base). Update the folder_path variable in the first data ingestion cell of the notebook to point to this directory.

### 2. Environment Installation

Install the required package dependencies. If running locally, execute this in your terminal. If running in Colab, this is handled by the first cell in the notebook.

```bash
pip install langchain langchain-community langchain-core pypdf chromadb sentence-transformers deepeval pandas ragas datasets streamlit rank_bm25
```

### 3. Initialize the Local Inference Engine (Ollama)

Because this project avoids cloud APIs, you must have the Ollama daemon running in the background before executing the evaluation loops.

If running locally on your machine:

- Download and install Ollama.

- Open a terminal and pull the model:

```Bash
ollama pull llama3:8b
Keep the Ollama application running in the background.
```

If running in Google Colab/Linux Notebook:

- Execute the designated setup cells at the top of the notebook to download the Linux binary, start the background nohup server, and pull the model into the environment.

### 4. Running the Evaluation Suite

Open RAG_Design.ipynb and execute the cells sequentially from top to bottom. The notebook is structured to automatically transition through the following phases:

- Document Ingestion & Vector Mapping: Parses PDFs, applies overlapping chunking (15%), and builds the ChromaDB dense index alongside the BM25 sparse index.

- Strategy Evaluation: Benchmarks Naive, Hybrid, and Hybrid+Reranker architectures across 100 queries (90 IR, 10 Out-of-Domain) using DeepEval and RAGAS.

- Hyperparameter Ablation Studies:
  - Chunk Size Optimization: Evaluates performance across 500, 1000, and 2000 token boundaries to measure context dilution.

  - Top-K Optimization: Evaluates Context Starvation vs. Dilution by testing retrieval limits of K=2, K=3, and K=5.

All metrics are automatically saved as .csv files and rendered as .png bar charts inside the evaluation_results/ directory.

## Critical Troubleshooting Notes

### Handling LLM Connection Disruptions (ConnectionError)

Processing the full 100-query batch evaluation suite puts significant strain on the local Ollama server's VRAM caching.

DeepEval: The evaluation loops are explicitly set to run_async=False to prevent overwhelming the local port with concurrent HTTP requests. Do not change this to True unless you are using a cloud API.

Timeouts: If a cell fails with an Ollama API Timeout or ConnectionError, simply re-run the Ollama initialization block to restart the daemon, then resume execution from the failed cell.

### Streamlit UI Deployment Timing

If you are deploying the interactive Streamlit UI via a local Cloudflare tunnel (using the final block in the notebook), allow a buffer of 5–10 seconds after the cell outputs the local URL before clicking the public Cloudflare link. This allows the secure routing tables to resolve fully.
