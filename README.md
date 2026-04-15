# CrocoIT RAG System

An English-only Retrieval-Augmented Generation (RAG) chatbot built over the public **CrocoIT** website. This project crawls website content, cleans and structures it, builds a hybrid retrieval pipeline, reranks the retrieved passages, and generates grounded answers using an LLM. It also includes a custom evaluation pipeline to measure retrieval and response quality.

---

## Overview

This project creates a domain-specific chatbot that answers questions about **CrocoIT** using only information retrieved from the company’s public website. Instead of relying purely on the language model’s internal knowledge, the system first retrieves relevant website content and then uses that evidence to generate a grounded response.

The system is designed to:

- crawl selected CrocoIT website sections
- clean and structure raw website text
- split content into section-aware chunks
- index the chunks using dense embeddings and BM25
- perform hybrid retrieval with scoring improvements
- rerank top candidates using a cross-encoder reranker
- apply abstention logic when evidence is weak
- generate concise answers strictly from retrieved context
- evaluate performance using several custom RAG metrics

---

## Features

### Core Features
- **Website crawling** from selected CrocoIT pages
- **Boilerplate removal** for cleaner text extraction
- **Section-aware chunking** for better retrieval granularity
- **Dense vector retrieval** using `BAAI/bge-m3`
- **Keyword retrieval** using BM25
- **Hybrid retrieval scoring**
- **Entity-aware query expansion**
- **Page-type-aware filtering and boosting**
- **Cross-encoder reranking** using `BAAI/bge-reranker-v2-m3`
- **LLM answer generation** using `Qwen/Qwen2.5-3B-Instruct`
- **Abstention mechanism** to reduce hallucinations
- **Interactive chatbot mode**
- **Evaluation pipeline** for relevance, faithfulness, robustness, and rejection

---

## Project Goal

The goal of this project is to build a **grounded question-answering system** for the CrocoIT website. The chatbot is restricted to answering only from retrieved evidence, which makes it more reliable than a standard chatbot for domain-specific use.

This is especially useful for:
- company information assistants
- product and services Q&A systems
- website-based support bots
- internal experiments in retrieval optimization
- RAG evaluation research and benchmarking

---

## Website Scope

The crawler focuses on selected CrocoIT website sections such as:

- Home
- Solutions
- About Us
- Our Products
- Articles
- Case Studies

The crawler avoids irrelevant or noisy pages such as:
- login/register pages
- policy pages
- feed/tag/category pages
- checkout/cart pages
- media files and static assets

This controlled crawling improves retrieval precision and reduces noise in the indexed knowledge base.

---

## Tech Stack

### Languages
- Python

### Main Libraries
- `requests`
- `trafilatura`
- `beautifulsoup4`
- `rank_bm25`
- `sentence-transformers`
- `transformers`
- `accelerate`
- `bitsandbytes`
- `faiss-cpu`
- `langchain`
- `langchain-community`
- `langchain-huggingface`
- `pandas`
- `datasets`

### Models Used
- **Embedding model:** `BAAI/bge-m3`
- **Reranker model:** `BAAI/bge-reranker-v2-m3`
- **Generator LLM:** `Qwen/Qwen2.5-3B-Instruct`

---

## System Architecture

The system follows a multi-stage RAG pipeline:

### 1. Web Crawling
The crawler starts from a list of seed URLs from the CrocoIT website and recursively visits valid pages within the allowed domain and path rules.

### 2. Content Extraction and Cleaning
HTML content is processed using `trafilatura` and `BeautifulSoup`. Boilerplate lines such as navigation items, footer text, contact-only lines, and repeated UI text are removed.

### 3. Section-Aware Chunking
Each page is divided into logical sections using headings (`h1`, `h2`, `h3`). Sections are then chunked into smaller overlapping passages to preserve local context.

### 4. Dense Indexing
Chunk embeddings are generated using `BAAI/bge-m3` and stored in a FAISS vector index.

### 5. BM25 Indexing
The same chunks are tokenized and indexed with BM25 for keyword-based retrieval.

### 6. Hybrid Retrieval
For each query, the system combines:
- dense retrieval using Max Marginal Relevance (MMR)
- BM25 retrieval
- page-type boosting
- section/title/URL term boosting
- entity-based boosting
- query-type-specific scoring

### 7. Reranking
Top retrieved candidates are reranked using a cross-encoder model to improve semantic relevance.

### 8. Abstention Logic
If the retrieved evidence is too weak or uncertain, the system returns:

`I do not have enough information from the available content.`

This helps reduce hallucinated answers.

### 9. Answer Generation
The final top passages are inserted into a constrained prompt, and the LLM generates an answer using only the provided context.

---

## Retrieval Strategy

One of the strongest parts of this project is the retrieval pipeline.

### Query Expansion
The system expands the user’s query based on:
- known entities such as `CrocoIT`, `2Loyal`, `ARTRIXX`, `ERP`
- predefined query hints
- query type detection

### Query Type Detection
The system classifies the question into categories such as:
- definition
- service listing
- product listing
- about company
- case study
- blog
- general

This classification helps guide page filtering and scoring.

### Page-Type Preference Detection
Each question type gets preference scores for page categories like:
- home
- service
- about
- product
- blog
- case study

This improves relevance by favoring the most likely source page type.

### Hybrid Scoring
Candidate passages are scored using a weighted combination of:
- dense retrieval score
- BM25 score
- page-type boost
- heading match boost
- title match boost
- URL match boost
- exact term boost
- entity boost
- query-type boost

This produces a more reliable ranking than using one retrieval method alone.

---

## Prompting Strategy

The model is prompted with strict instructions:

- answer only in English
- answer only from retrieved context
- do not hallucinate
- if unsupported, abstain
- keep the response concise and accurate

This grounded prompting makes the generation stage safer and more aligned with the retrieved evidence.

---

## Evaluation Pipeline

This project does not stop at building the RAG system — it also evaluates it using several custom metrics.

### Included Evaluation Metrics

#### 1. Answer Relevance
Measures how related the generated answer is to the question.

#### 2. Faithfulness / Fidelity
Measures how much the answer overlaps with the retrieved contexts.

#### 3. Context Relevance
Measures how relevant the retrieved contexts are to the question.

#### 4. Negative Rejection Accuracy
Checks whether the model correctly refuses to answer unsupported questions.

#### 5. Noise Robustness
Tests whether irrelevant noisy text can affect the generated answer.

#### 6. Counterfactual Robustness
Adds false statements to the context and checks whether the model incorrectly uses them.

#### 7. Information Integration
Measures whether multi-source questions are answered using more than one retrieved source.

### Evaluation Outputs
The project saves:
- prediction JSON files
- per-metric CSV reports
- summary JSON results

This makes the project useful not only as an application, but also as an experimental RAG benchmark.

---

## Folder Structure

A typical structure for the project is:

```bash
crocoit_rag_final/
│
├── data/
│   ├── scraped_pages.json
│   ├── advanced_eval_predictions.json
│   ├── eval_core_metrics.csv
│   ├── eval_negative_rejection.csv
│   ├── eval_noise_robustness.csv
│   ├── eval_counterfactual.csv
│   ├── eval_information_integration.csv
│   └── eval_summary.json
│
├── faiss_index/
│   └── ... saved FAISS files ...
│
└── README.md
