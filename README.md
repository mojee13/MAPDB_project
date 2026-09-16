# Distributed Analysis of COVID-19 Research Literature (CORD-19)

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Dask](https://img.shields.io/badge/Dask-Distributed%20Bag%20%26%20Client-orange.svg)](https://dask.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![Cloud-Veneto](https://img.shields.io/badge/Infrastructure-Cloud--Veneto%20SSH%20Cluster-purple.svg)](http://cloudveneto.it/)
[![University](https://img.shields.io/badge/University-Padova%20%7C%20Physics%20of%20Data-navy.svg)](https://www.unipd.it/)

A high-performance distributed data science project analyzing full-text scientific papers from the **COVID-19 Open Research Dataset (CORD-19)**, conducted for the **Management and Analysis of Physics Data (MAPD - Part B)** course in the Physics of Data Master's degree at the **University of Padua**.

---

## 📋 Table of Contents
- [Architecture & Infrastructure](#-architecture--infrastructure)
- [Project Tasks & Technical Implementation](#-project-tasks--technical-implementation)
  - [Task 1: Distributed Bag Word Counter & NLP Pipeline](#task-1-distributed-bag-word-counter--nlp-pipeline)
  - [Task 2: Country Representation & Scalability Benchmarking](#task-2-country-representation--scalability-benchmarking)
  - [Task 3: Title Embeddings & Cosine Similarity Matrix](#task-3-title-embeddings--cosine-similarity-matrix)
- [Performance & Scaling Mathematics](#-performance--scaling-mathematics)
- [JSON Document Schema](#-json-document-schema)
- [Repository Structure](#-repository-structure)
- [Setup & Deployment Guide](#-setup--deployment-guide)
- [Authors & Acknowledgments](#-authors--acknowledgments)

---

## ☁️ Architecture & Infrastructure

The project utilizes a **multi-node Dask distributed cluster** deployed on **Cloud-Veneto** infrastructure, linking 3 Virtual Machines via passwordless SSH protocol:

```
                          ┌─────────────────────────────────────────┐
                          │          Dask Scheduler Client          │
                          └────────────────────┬────────────────────┘
                                               │
                                     (SSH Cluster Network)
                                               │
               ┌───────────────────────────────┼───────────────────────────────┐
               │                               │                               │
               ▼                               ▼                               ▼
    ┌─────────────────────┐         ┌─────────────────────┐         ┌─────────────────────┐
    │  VM Worker 1        │         │  VM Worker 2        │         │  VM Worker 3        │
    │  (192.168.x.x)      │         │  (192.168.x.y)      │         │  (192.168.x.z)      │
    │  Dask Worker Exec   │         │  Dask Worker Exec   │         │  Dask Worker Exec   │
    └─────────────────────┘         └─────────────────────┘         └─────────────────────┘
```

---

## 🔬 Project Tasks & Technical Implementation

### Task 1: Distributed Bag Word Counter & NLP Pipeline
- **Map Phase**: Extracts text blocks from abstracts and body text across 1,000 JSON documents.
- **NLP Text Normalization**:
  - Lowercasing, punctuation stripping, and special character filtering using regular expressions.
  - Stop-word removal (NLTK English stop-words) and token filtering to retain meaningful scientific vocabulary.
- **Reduce Phase**: Aggregates token counts across Dask partitions using `dask.bag.frequencies()`.
- **Top Vocabulary Analysis**: Identifies dominant epidemic terminology (`covid`, `virus`, `patients`, `infection`, `cells`, `protein`).

```python
# Example Dask Bag Word Counter Execution
import dask.bag as db

b = db.read_text('data/*.json').map(json.loads)
words = b.map(extract_and_clean_text).flatten()
word_counts = words.frequencies().topk(20, key=lambda x: x[1]).compute()
```

---

### Task 2: Country Representation & Scalability Benchmarking
- **Geographic Affiliation Analysis**: Extracts author affiliations from document metadata to determine the **most and least represented countries** in COVID-19 research literature.
- **Cluster Scalability & Partition Optimization**:
  - Benchmark execution runtimes across varying partition counts ($P \in [1, 2, 4, 8, 16, 32, 64]$) and worker counts ($W \in [1, 2, 3]$ VMs).
  - Identifies the trade-off between task parallelism and Dask task-scheduling / network serialization overhead.

---

### Task 3: Title Embeddings & Cosine Similarity Matrix
- **Semantic Title Vectorization**:
  - Converts paper titles into dense vector representations ($n \times m$ matrices where $n$ is word count and $m$ is embedding dimension).
- **Pairwise Cosine Similarity**:
  $$\text{Cosine Similarity}(u, v) = \frac{u \cdot v}{\|u\| \|v\|}$$
- **Similarity Matrix Visualization**: Computes and plots $N \times N$ heatmap matrices comparing semantic proximity between scientific paper titles.
- **Partition Optimization for NLP Matrix Operations**: Measures performance trade-offs when distributing dense vector matrix computations across Dask partitions.

---

## 📐 Performance & Scaling Mathematics

We evaluate the distributed execution efficiency using standard parallel computing metrics:

1. **Speedup $S(W)$**:
   $$S(W) = \frac{T(1)}{T(W)}$$
   where $T(1)$ is sequential execution time on 1 worker VM, and $T(W)$ is execution time on $W$ worker VMs.

2. **Parallel Efficiency $E(W)$**:
   $$E(W) = \frac{S(W)}{W} = \frac{T(1)}{W \cdot T(W)}$$

3. **Amdahl's Law (Strong Scaling)**:
   $$S(W) = \frac{1}{(1 - s) + \frac{s}{W}}$$
   where $s$ is the parallelizable fraction of the word-counting and matrix computation algorithm.

> [!NOTE]
> **Key Finding**: Increasing partition count $P$ improves worker CPU utilization up to $P \approx 16-32$. Beyond $P = 64$, network IPC serialization overhead dominates the computation time.

---

## 📄 JSON Document Schema

The CORD-19 research papers follow a standardized JSON schema:

```json
{
  "paper_id": "<40-character sha1 hash of PDF>",
  "metadata": {
    "title": "<String>",
    "authors": [
      {
        "first": "<String>",
        "last": "<String>",
        "affiliation": {"laboratory": "...", "institution": "...", "location": {"country": "..."}},
        "email": "<String>"
      }
    ],
    "abstract": [{"text": "<Paragraph>", "cite_spans": [], "section": "Abstract"}],
    "body_text": [{"text": "<Paragraph>", "cite_spans": [], "section": "Introduction"}],
    "bib_entries": {"BIBREF0": {"title": "...", "year": 2020, "venue": "..."}},
    "ref_entries": {"FIGREF0": {"text": "Caption...", "type": "figure"}}
  }
}
```

---

## 📁 Repository Structure

```
MAPDB_project/
├── Final_MAPD-B_ Mojtaba_Roya.ipynb    # Main computational notebook with Dask code & visualizations
├── json_schema.txt                      # Official CORD-19 JSON schema definition
├── Project_Description                  # Course assignment specifications
├── Bibliography/                        # Project documentation & reference PDF syllabus
│   └── Latest revision 30-05-2022.pdf   # Official MAPD-B project syllabus
├── README.md                            # Comprehensive project documentation
└── .gitignore                           # Git ignore rules
```

---

## 🛠️ Setup & Deployment Guide

### 1. Prerequisites
- Python 3.8+
- Dask Distributed (`pip install dask[complete] distributed`)
- NLP & Visualization (`pip install nltk gensim spacy seaborn matplotlib scikit-learn`)
- Cloud-Veneto virtual machines with passwordless SSH key pair access.

### 2. Dask SSH Cluster Initialization
```python
from dask.distributed import Client, SSHCluster

cluster = SSHCluster(
    ["192.168.x.x", "192.168.x.y", "192.168.x.z"],
    connect_options={"username": "ubuntu"},
    worker_options={"nthreads": 2}
)
client = Client(cluster)
print("Dask Cluster Dashboard:", client.dashboard_link)
```

### 3. Execution
Launch Jupyter Notebook and run [`Final_MAPD-B_ Mojtaba_Roya.ipynb`](./Final_MAPD-B_%20Mojtaba_Roya.ipynb) cell-by-cell.

---

## 👥 Authors & Acknowledgments

* **Authors**: Mojtaba Roshana & Roya ...
* **Course**: Management and Analysis of Physics Data (MAPD - Part B)
* **Degree**: M.Sc. in Physics of Data, Department of Physics and Astronomy, University of Padua, Italy.
