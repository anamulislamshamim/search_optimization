![NDCG Search Metrics](images/embedding_model_comparison_ndcg_fixed.png)

# 🧠 Embedding Model Comparison for Query–Webpage Relevance

### *Optimization Task for Sisu Clinic Content Relevance Scoring*

This project evaluates **which embedding model** performs best for computing *semantic relevance* between user queries and webpages from **Sisu Clinic** (`https://www.sisuclinic.com/en-ie/...`).

The goal is to find an embedding model that produces the most *interpretable and meaningful relevancy scores* for business stakeholders.

---

## 🎯 Project Objectives

### **1. Crawl Sisu Clinic Pages**

* Extract all URLs matching pattern:

  ```
  https://www.sisuclinic.com/en-ie/*
  ```
* Parse pages listed under the sitemap:

  ```
  https://www.sisuclinic.com/sitemap.xml
  ```
* A **custom Python crawler** is used to replicate the behaviour of *Elastic Open Crawler* because:

  * Elastic crawler requires heavy resources.
  * Running it in Docker locally is expensive.
  * Custom crawler helps understand crawling, extraction, parsing, and indexing logic more deeply.

### **2. Index Webpages into Elasticsearch**

* Elasticsearch runs locally using Docker.
* The repository includes a `docker-compose.yml`:

  ```bash
  docker-compose up -d
  ```
* Crawled webpages are indexed with fields like:

  * `url`
  * `title`
  * `content`
  * `embedding` (from each tested model)

### **3. Compute Semantic Relevance**

For each query in the dataset:

* Generate embeddings for:

  * OpenAI → `text-embedding-3-small`
  * Gemini → `text-embedding-005`
  * HuggingFace → `all-MiniLM-L6-v2`
* Compare query embeddings with webpage embeddings using cosine similarity.
* Store the "relevancy score" per model.

> Note: OpenAI Free Trial **does not allow embedding models**, so evaluation includes HuggingFace and Gemini only.

### **4. Compare Model Performance**

To evaluate how well each embedding model retrieves relevant pages, **Mean NDCG (MNDCG)** is used.

* For each query, top 10 search results are ranked.
* NDCG is computed using:

  * Model ranking results
  * Ground-truth relevance labels
* Final MNDCG across 50 queries:

```
HuggingFace MiniLM      → 1.0  
Gemini 005              → 1.0
```

Both models achieved **perfect MNDCG** based on the dataset and relevance labels.

---

# 📦 Technologies Used

| Component     | Technology                                                                       |
| ------------- | -------------------------------------------------------------------------------- |
| Web Crawling  | Python, BeautifulSoup                                                            |
| Embeddings    | HuggingFace, Gemini, OpenAI API (key created but model unavailable in free tier) |
| Search Engine | Elasticsearch (Docker)                                                           |
| Evaluation    | MNDCG (Mean Normalized Discounted Cumulative Gain)                               |
| Storage       | Indexed webpages + embeddings in Elasticsearch                                   |

---

# 🧰 System Architecture Overview

```
            ┌─────────────────────┐
            │  Sitemap (XML)      │
            └─────────┬───────────┘
                      │
             Crawl & Extract Pages
                      │
     ┌────────────────▼────────────────┐
     │      Custom Python Crawler      │
     └───────────────┬─────────────────┘
                     │ Parsed Content
             ┌───────▼────────┐
             │ Elasticsearch   │  (Docker)
             └───────┬────────┘
                     │
   Generate Embeddings from 3 Models
                     │
     ┌───────────────▼────────────────┐
     │   Relevance Score Computation   │
     └───────────────┬────────────────┘
                     │
            ┌────────▼────────┐
            │  MNDCG Report    │
            └──────────────────┘
```

---

# 📊 Evaluation Approach (Subtask 3)

### Why **NDCG**?

* It is the industry-standard ranking evaluation metric.
* Rewards correct ordering of highly relevant pages.
* Penalizes wrong ordering more than wrong items.
* Ideal for search relevance tasks.

### Factors in the evaluation:

1. **Semantic similarity accuracy**
2. **Stability and consistency of rankings**
3. **Handling of long vs short queries**
4. **Embedding dimensionality**
5. **Latency + model cost**
6. **Interpretability of relevance scores**

### Result Summary

Both **HuggingFace MiniLM** and **Gemini 001** achieved **MNDCG = 1.0**, meaning:

* Both models produced the *same ranking order* as ground truth for the tested dataset.
* Both are suitable for semantic page–query relevance tasks.

### Observations

* **HuggingFace MiniLM**

  * Fast, no cost, low latency
  * Works offline
  * Excellent for large-scale indexing
* **Gemini 001**

  * Slightly richer semantic understanding
  * Good for complex/longer queries
  * Cloud-dependent

---

# 📁 Project Structure

```
project/
│
├── docker-compose.yml
|__ .env
|__ config.py
|__ embedding_search_performance_evaluation.ipynb
|__ README.md
```

---

# 🚀 How to Run Locally

### 1. Clone the repository: 
```bash
git clone https://github.com/anamulislamshamim/search_optimization.git
```

### 2. Go to Project Directory
```bash
cd search_optimization
```

### 3. Create Virtual Environment and Active the it
```bash
python -m venv denv
# windows
.\denv\Scripts\activate
```

## 4. Install Dependencies
```bash
pip install -r requirements.txt
```

### 5. Start Elasticsearch

```bash
docker-compose up -d
```

### 6. Run embedding_search_performance_evaluation.ipynb
```bash
   click run all of Jupyter Note book or run each section of code.
```

---

# 🏁 Final Conclusion

| Model                             | MNDCG Score     | Summary                                              |
| --------------------------------- | --------------- | ---------------------------------------------------- |
| **HuggingFace MiniLM**            | **1.0**         | Best trade-off between speed, cost, and accuracy     |
| **Gemini 005**                    | **1.0**         | Strong semantic accuracy; ideal for complex language |
| **OpenAI text-embedding-3-small** | *Not evaluated* | Free tier does not allow embedding API               |

### **Recommended Model (Business Perspective):**

➡️ If cost & self-hosting matter → **HuggingFace MiniLM**
➡️ If richer semantics preferred → **Gemini 001**

Both models meet the business requirement of generating **interpretable, stable relevance scores**.
