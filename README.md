# Product Recommender Chatbot

A conversational RAG (Retrieval-Augmented Generation) chatbot that answers product-related queries using Flipkart customer reviews. Built with LangChain, AstraDB, Groq, and Flask — deployed on Kubernetes with Prometheus and Grafana monitoring.

---

## Architecture

```
User Query
    ↓
Flask App (app.py)
    ↓
RAGChainBuilder — rewrites query using chat history
    ↓
AstraDB Vector Store — retrieves top 3 relevant reviews
    ↓
Groq LLM (LLaMA 3.1) — generates answer from retrieved context
    ↓
Response returned to user
```

**Key components:**
- `flipkart/data_converter.py` — Converts CSV reviews into LangChain Documents
- `flipkart/data_ingestion.py` — Embeds and stores documents in AstraDB
- `flipkart/rag_chain.py` — Builds the conversational retrieval chain
- `app.py` — Flask server exposing the chat UI and API endpoints

---

## Tech Stack

| Layer | Tool |
|---|---|
| LLM | Groq (LLaMA 3.1 8B Instant) |
| Embeddings | BAAI/bge-base-en-v1.5 (sentence-transformers) |
| Vector Store | AstraDB |
| Framework | LangChain |
| Web Server | Flask |
| Monitoring | Prometheus + Grafana |
| Deployment | Docker + Kubernetes (Minikube) |

---

## Local Setup

### 1. Clone the repo

```bash
git clone <your-repo-url>
cd FLIPKART
```

### 2. Create and activate virtual environment

```bash
python -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Create `.env` file

```env
ASTRA_DB_API_ENDPOINT=your_astra_db_api_endpoint
ASTRA_DB_APPLICATION_TOKEN=your_astra_db_token
ASTRA_DB_KEYSPACE=default_keyspace
GROQ_API_KEY=your_groq_api_key
HF_TOKEN=your_huggingface_token
```

### 5. Run the app

```bash
python app.py
```

Open `http://localhost:8080` in your browser.

> **Note:** On first run with `load_existing=False`, the app will embed and upload all reviews to AstraDB. Switch to `load_existing=True` in `app.py` for subsequent runs to skip re-ingestion.

---

## Deployment on Kubernetes (GCP VM)

### 1. Build Docker image

```bash
eval $(minikube docker-env)
docker build -t flask-app:latest .
```

### 2. Create Kubernetes secrets

```bash
kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY="" \
  --from-literal=ASTRA_DB_APPLICATION_TOKEN="" \
  --from-literal=ASTRA_DB_KEYSPACE="default_keyspace" \
  --from-literal=ASTRA_DB_API_ENDPOINT="" \
  --from-literal=HF_TOKEN="" \
  --from-literal=HUGGINGFACEHUB_API_TOKEN=""
```

### 3. Deploy the app

```bash
kubectl apply -f flask-deployment.yaml
kubectl get pods
kubectl port-forward svc/flask-service 5000:80 --address 0.0.0.0
```

---

## Monitoring

### Prometheus

```bash
kubectl create namespace monitoring
kubectl apply -f prometheus/prometheus-configmap.yaml
kubectl apply -f prometheus/prometheus-deployment.yaml
kubectl port-forward --address 0.0.0.0 svc/prometheus-service -n monitoring 9090:9090
```

Access at `http://<vm-external-ip>:9090`

### Grafana

```bash
kubectl apply -f grafana/grafana-deployment.yaml
kubectl port-forward --address 0.0.0.0 svc/grafana-service -n monitoring 3000:3000
```

Access at `http://<vm-external-ip>:3000` — default login is `admin / admin`

**Configure data source:**
- Go to Settings → Data Sources → Add Data Source → Prometheus
- URL: `http://prometheus-service.monitoring.svc.cluster.local:9090`
- Click Save & Test

---

## API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/` | GET | Chat UI |
| `/get` | POST | Send a message, receive chatbot response |
| `/metrics` | GET | Prometheus metrics |

---

## Project Structure

```
FLIPKART/
├── flipkart/
│   ├── config.py           # Environment variable loader
│   ├── data_converter.py   # CSV → LangChain Documents
│   ├── data_ingestion.py   # Embed + store in AstraDB
│   └── rag_chain.py        # Conversational RAG chain
├── prometheus/
│   ├── prometheus-configmap.yaml
│   └── prometheus-deployment.yaml
├── grafana/
│   └── grafana-deployment.yaml
├── utils/
│   ├── logger.py
│   └── custom_exception.py
├── templates/
│   └── index.html
├── static/
│   └── style.css
├── data/                   # CSV data (not tracked in git)
├── app.py
├── Dockerfile
├── flask-deployment.yaml
├── setup.py
├── requirements.txt
└── .env                    # Not tracked in git
```
