# Distributed Analysis of COVID-19 Research Literature (CORD-19)

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Dask](https://img.shields.io/badge/Dask-Distributed%20Computing-orange.svg)](https://dask.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![University](https://img.shields.io/badge/University-Padova%20%7C%20Physics%20of%20Data-navy.svg)](https://www.unipd.it/)

A high-performance distributed computing project analyzing 1,000 full-text scientific papers from the **COVID-19 Open Research Dataset (CORD-19)**, conducted for the **Management and Analysis of Physics Data (MAPD - Part B)** course in the Physics of Data Master's degree at the University of Padua.

---

## 📌 Overview & Distributed Architecture

The CORD-19 dataset (*Allen Institute for AI / Kaggle*) comprises over 75,000 scientific publications on COVID-19, SARS-CoV-2, and historical coronaviruses. 

This project deploys a **distributed Dask cluster** connecting **3 Virtual Machines via SSH on Cloud-Veneto** to perform parallel text mining, scalability benchmarking, and Natural Language Processing (NLP) embedding generation over structured JSON document collections.

```
Cloud-Veneto Cluster Infrastructure:
┌──────────────────────────────────────────────────────────┐
│                   Dask Scheduler Client                  │
└────────────────────────────┬─────────────────────────────┘
                             │ (SSH Protocol)
      ┌──────────────────────┼──────────────────────┐
      ▼                      ▼                      ▼
┌───────────┐          ┌───────────┐          ┌───────────┐
│  Worker 1 │          │  Worker 2 │          │  Worker 3 │
│  (VM #1)  │          │  (VM #2)  │          │  (VM #3)  │
└───────────┘          └───────────┘          └───────────┘
```

---

## 🔬 Computational Tasks & Pipeline

### Task 1: Distributed Bag Word Counter
- Implements a parallel MapReduce-style word frequency algorithm utilizing Dask `Bag` data structures.
- Parses full-text JSON documents, cleans punctuation/stop-words, and aggregates token frequencies across distributed cluster workers.
- Encapsulated into a parameterized benchmark function to measure execution runtime against partition granularity.

### Task 2: Scalability & Performance Benchmarking
- Conducts experimental runs altering the **number of data partitions** ($P \in [1, 64]$) and **active worker nodes** (1 to 3 VMs).
- Evaluates strong scaling (fixed data volume, increasing workers) and weak scaling performance laws to identify communication bottlenecks.

### Task 3: Paper Title Vector Embedding Generation
- Transforms paper titles into dense vector representations ($n \times m$ matrices where $n$ is the word count and $m$ is the embedding dimension).
- Applies text pre-processing (tokenization, lemmatization, stop-word filtering) and vector space modeling for downstream semantic clustering and literature categorization.

---

## 📄 JSON Document Schema

The CORD-19 full-text documents are structured according to the following JSON schema:

```json
{
  "paper_id": "<40-character sha1 hash>",
  "metadata": {
    "title": "<Paper Title>",
    "authors": [{"first": "...", "last": "...", "affiliation": {...}}],
    "abstract": [{"text": "...", "cite_spans": [], "section": "Abstract"}],
    "body_text": [{"text": "...", "cite_spans": [], "section": "Introduction"}],
    "bib_entries": {"BIBREF0": {"title": "...", "year": 2020, "DOI": ["..."]}},
    "ref_entries": {"FIGREF0": {"text": "Caption...", "type": "figure"}}
  }
}
```

---

## 📁 Repository Structure

```
MAPDB_project/
├── Final_MAPD-B_ Mojtaba_Roya.ipynb    # Main distributed computation notebook
├── json_schema.txt                      # Detailed CORD-19 JSON document schema
├── Project_Description                  # Course assignment specifications
├── Bibliography/                        # Reference literature & project guidelines
│   └── Latest revision 30-05-2022.pdf   # Official MAPD-B project syllabus
├── README.md                            # Project documentation
└── .gitignore                           # Git ignore rules
```

---

## 🛠️ Infrastructure & Setup

### Prerequisites
- Python 3.8+
- Dask & Distributed (`pip install dask[complete] distributed`)
- NLTK / spaCy / Gensim (`pip install nltk gensim spacy`)
- Cloud-Veneto virtual machines with SSH key authentication.

### Cluster Execution
1. Configure SSH passwordless access across Cloud-Veneto VM nodes.
2. Initialize Dask `SSHCluster` or `Client`:
   ```python
   from dask.distributed import Client, SSHCluster
   
   cluster = SSHCluster(
       ["192.168.x.x", "192.168.x.y", "192.168.x.z"],
       connect_options={"username": "ubuntu"},
       worker_options={"nthreads": 2}
   )
   client = Client(cluster)
   ```
3. Open and run [`Final_MAPD-B_ Mojtaba_Roya.ipynb`](./Final_MAPD-B_%20Mojtaba_Roya.ipynb) in Jupyter.

---

## 👥 Authors & Course Information

* **Authors**: Mojtaba Roshana & Roya ...
* **Course**: Management and Analysis of Physics Data (MAPD - Part B)
* **Degree**: M.Sc. in Physics of Data, Department of Physics and Astronomy, University of Padua, Italy.
