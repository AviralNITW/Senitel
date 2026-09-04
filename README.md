# SENTINEL - E-Commerce Review Integrity & Synthetic Manipulation Detection Engine

> Production-ready Machine Learning and NLP system that audits live e-commerce reviews (Amazon, Flipkart, Shopify) in real time, generates 100-dimensional semantic embeddings, and detects computer-generated and synthetic review manipulation using a high-speed Linear SVM decision boundary.

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Vercel%20Production-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://senitel-drab.vercel.app)
[![Python](https://img.shields.io/badge/Python-3.10%20%7C%203.11%20%7C%203.12-blue?style=flat-square&logo=python)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.1.0-lightgrey?style=flat-square&logo=flask)](https://flask.palletsprojects.com/)
[![NumPy](https://img.shields.io/badge/NumPy-Inference%20Kernel-blue?style=flat-square&logo=numpy)](https://numpy.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

Live Application: [https://senitel-drab.vercel.app](https://senitel-drab.vercel.app)

---

## Overview

SENTINEL is an engineering-first review verification platform built to combat the proliferation of AI-generated, bot-fabricated, and coordinated promotional consumer reviews.

Unlike conventional heuristic or keyword-based approaches, Sentinel couples **dynamic proxy scraping** with **100-dimensional continuous Word2Vec semantic embeddings** and a **Linear Support Vector Machine (SVM)** mathematical decision kernel. The inference layer is executed entirely in pure NumPy linear algebra, delivering sub-millisecond execution times and minimal memory footprint on serverless environments.

---

## UML System Architecture

### 1. Component Architecture Diagram

```mermaid
graph TD
    classDef client fill:#1e293b,stroke:#38bdf8,stroke-width:2px,color:#f8fafc;
    classDef server fill:#0f172a,stroke:#818cf8,stroke-width:2px,color:#f8fafc;
    classDef proxy fill:#334155,stroke:#34d399,stroke-width:2px,color:#f8fafc;
    classDef ml fill:#1e1b4b,stroke:#a855f7,stroke-width:2px,color:#f8fafc;
    classDef ui fill:#022c22,stroke:#10b981,stroke-width:2px,color:#f8fafc;

    subgraph ClientLayer["Client Interface Layer"]
        UI["Web Dashboard UI<br/>(Tailwind CSS + ES6 JavaScript)"]:::client
        URL_INPUT["Product URL Scanner Mode"]:::client
        TEXT_INPUT["Direct Review Text Mode"]:::client
    end

    subgraph ServerlessLayer["Backend API Layer (Vercel Serverless)"]
        APP["Flask Application Core<br/>(app.py / index.py)"]:::server
        ROUTER["API Router & Validator<br/>(/analyze, /api/health)"]:::server
    end

    subgraph ScrapingLayer["Data Acquisition Engine"]
        SCRAPER["Scraper Engine (scraper.py)"]:::proxy
        PROXY["ScraperAPI Residential Proxy<br/>(Anti-Bot & CAPTCHA Bypass)"]:::proxy
        AMZ["Amazon Review Extractor"]:::proxy
        FK["Flipkart Review Extractor"]:::proxy
    end

    subgraph MLLayer["NLP & Machine Learning Inference Engine"]
        PREPROC["NLP Text Normalizer (preprocessing.py)<br/>• Regex Clean • Lowercase • Stopword Removal"]:::ml
        W2V["Word2Vec Vectorizer (w2v_weights.npz)<br/>• 100-Dimensional Continuous Embeddings"]:::ml
        FEAT["Feature Assembler<br/>• Vector + Length + Rating (102-D Vector)"]:::ml
        SVM["Linear SVM Kernel (svm_weights.npz)<br/>• Pure NumPy Dot Product: W · x + b"]:::ml
    end

    subgraph AnalyticsLayer["Telemetry & Visualization Engine"]
        ANALYTICS["Trust Index Calculator<br/>• Risk Badges (Low / Mod / High)"]:::ui
        DASHBOARD["Dynamic Response Payload<br/>• Review Metrics • JSON/CSV Export"]:::ui
    end

    UI --> URL_INPUT
    UI --> TEXT_INPUT
    URL_INPUT --> ROUTER
    TEXT_INPUT --> ROUTER
    ROUTER --> APP

    APP --> SCRAPER
    SCRAPER --> PROXY
    PROXY --> AMZ
    PROXY --> FK
    AMZ --> PREPROC
    FK --> PREPROC
    TEXT_INPUT --> PREPROC

    PREPROC --> W2V
    W2V --> FEAT
    FEAT --> SVM
    SVM --> ANALYTICS
    ANALYTICS --> DASHBOARD
    DASHBOARD --> UI
```

---

### 2. Sequence Flow Diagram

```mermaid
sequenceDiagram
    autonumber
    actor User as Client Browser
    participant App as Flask Backend (Vercel)
    participant Scraper as Scraper Engine
    participant Proxy as ScraperAPI Gateway
    participant Ecom as Target Platform (Amazon / Flipkart)
    participant NLP as Preprocessor & Word2Vec
    participant SVM as Linear SVM Kernel (NumPy)

    User->>App: POST /analyze { url: "https://amazon.com/dp/..." }
    App->>Scraper: scrape_reviews(url)
    
    alt URL Scraping Mode
        Scraper->>Proxy: Fetch HTML via Residential Proxy
        Proxy->>Ecom: Request Product Review Page
        Ecom-->>Proxy: Return Clean Product HTML
        Proxy-->>Scraper: Forward Decoded Response
        Scraper->>Scraper: Parse Review Blocks, Extract Star Ratings & Clean Noise
    else Direct Text Mode
        User->>App: POST /analyze { review_text: "...", rating: 5 }
    end

    Scraper-->>App: Return Raw Reviews List [ { Rating, Review Text } ]
    
    loop For Each Review Item
        App->>NLP: preprocess_text(text)
        NLP-->>App: Return Clean Normalized Tokens
        App->>NLP: vectorize_tokens(tokens) -> 100D Vector
        NLP-->>App: Return Word2Vec Semantic Vector
        App->>NLP: append_metadata(vector, rating, text_length) -> 102D Vector
        App->>SVM: compute_decision(W · x + b)
        SVM-->>App: Output Binary Prediction (0: Synthetic / 1: Organic) + Confidence Score
    end

    App->>App: Compute Trust Index (%) & Risk Classification
    App-->>User: Return JSON Response with Review Diagnostics & Telemetry
    User->>User: Render Visual Metrics, Breakdown Charts & Search Filters
```

---

## Core Features

- **Live Multi-Platform Scraping:**
  - **Amazon Engine:** Direct ASIN extraction, canonical URL resolution, and automated review body parsing.
  - **Flipkart Engine:** Dynamic review extraction with structured fallback parsers.
  - **Residential Proxy Integration:** Compatible with ScraperAPI for automated WAF, CAPTCHA, and datacenter IP block resolution on cloud environments.
  - **Direct Review Text Auditor:** Audit custom or offline review passages with arbitrary star ratings.

- **High-Speed Pure NumPy Inference:**
  - Compact pre-computed weight matrices (`svm_weights.npz` and `w2v_weights.npz`).
  - Pure linear algebra decision kernel ($y = W \cdot x + b$) without heavy runtime frameworks.
  - Sub-millisecond inference time and serverless bundle footprint under 20MB.

- **Telemetry Dashboard:**
  - Real-time **Trust Index (%)** computation.
  - Classification into **Organic (OR)** vs. **Synthetic / Computer-Generated (CG)**.
  - Interactive search and filter controls.
  - Export capabilities for **JSON telemetry** and **CSV datasets**.

---

## Tech Stack

| Component | Technologies |
|---|---|
| **Live Deployment** | Vercel Serverless Functions |
| **Backend & API** | Python 3.12, Flask, Werkzeug |
| **Inference Kernel** | Pure NumPy Linear Algebra ($W \cdot x + b$) |
| **NLP & Embeddings** | Word2Vec Vector Space (100-D), NLTK |
| **Scraping & Proxy** | Requests, BeautifulSoup4, ScraperAPI Gateway |
| **Frontend UI** | HTML5, Tailwind CSS, ES6 JavaScript, Plus Jakarta Sans, JetBrains Mono |

---

## Local Development Setup

### 1. Clone the Repository
```bash
git clone https://github.com/AviralNITW/Senitel.git
cd Senitel
```

### 2. Create Virtual Environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS / Linux
python -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables (Optional)
Create a `.env` file or export your ScraperAPI key for cloud-level proxy scraping:
```bash
# Windows PowerShell
$env:SCRAPER_API_KEY="your_api_key_here"

# Linux / macOS
export SCRAPER_API_KEY="your_api_key_here"
```

### 5. Launch the Server
```bash
python app.py
```

Access the dashboard at: **`http://localhost:5001`**

---

## Production Deployment (Vercel)

1. Push your repository to GitHub.
2. Import the project in [Vercel Dashboard](https://vercel.com/dashboard).
3. In **Settings -> Environment Variables**, add:
   - `SCRAPER_API_KEY`: *(Your ScraperAPI Key)*
4. Deploy the project. The serverless configuration in `index.py` and `requirements.txt` runs out of the box.

---

## Author

**Aviral**  
- GitHub: [@AviralNITW](https://github.com/AviralNITW)  
- Live Application: [https://senitel-drab.vercel.app](https://senitel-drab.vercel.app)

---

## License

This project is licensed under the [MIT License](LICENSE).
