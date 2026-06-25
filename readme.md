# Agentic Travel Assistant with Multi-Tool Routing (RAG & Web Search)

An intelligent, context-aware travel assistant built using the modern **OpenAI Python SDK**, **Azure Identity**, and **Azure AI Foundry**. 

This project demonstrates the implementation of an agentic workflow where a Large Language Model (`gpt-4.1`) dynamically routes user queries between an on-demand vector store containing internal company assets (Retrieval-Augmented Generation) and a live web search tool to fetch real-time destination data.

---

## 🚀 Project Overview & Key Features

The application serves as a automated advisor for *Margie's Travel* clients, moving beyond static model knowledge by executing autonomous tool-calling:
* **Dynamic Multi-Tool Functionality:** Combines a local vector store search (`file_search`) with live internet queries (`web_search`).
* **On-the-Fly Vectorization:** Programmatically initializes a vector store, aggregates local unstructured documentation (PDF brochures), and uploads/polls the batch during runtime initialization.
* **Stateful Conversations:** Tracks execution states across user turns via sequential response ID pairing to ensure seamless multi-turn reasoning.
* **Enterprise-Grade Identity Layer:** Uses Microsoft Entra ID token-based credential scoping (`DefaultAzureCredential`) for passwordless access to Azure OpenAI service boundaries.

---

## 🛠️ Architecture Flow & Mechanics

When a user submits a prompt, the application coordinates multiple actions:
1. **Initialization:** The app spins up, scans the internal directory for product documentation, compiles a temporary vector store embedding layout, and hooks into Azure OpenAI endpoints.
2. **Analysis:** The model interprets the user's intent. 
3. **Execution Routing:** * If the query regards internal booking structures or specific company partnerships, it executes a `file_search` tool constraint over the vectorized asset store.
   * If the query asks for real-time local dynamics, flight status, or current events, it switches execution pathways to a concurrent `web_search`.

---

## 📂 Repository Structure

```text
├── brochures/            # Unstructured company assets (Internal Margie's Travel PDF brochures)
├── .env                  # Environment configurations (Service endpoints and deployment tags)
├── .gitignore            # Excludes local Python runtime dependencies (.venv)
├── requirements.txt      # Project library manifest (OpenAI, Azure Identity, Glob dependencies)
└── tools-app.py          # Main application engine handling file orchestration, tool setup, and execution loops
```

## 💻 Code Architecture Highlight
Here is the setup for the runtime initialization and agentic instruction structure:

```bash
# Create vector store and dynamically chunk/upload binary document matrices
print("Creating vector store and uploading files...")
vector_store = openai_client.vector_stores.create(name="travel-brochures")
file_streams = [open(f, "rb") for f in glob.glob("brochures/*.pdf")]

file_batch = openai_client.vector_stores.file_batches.upload_and_poll(
     vector_store_id=vector_store.id,
     files=file_streams
)

# Multi-Tool runtime instantiation framework inside the conversation loop
response = openai_client.responses.create(
     model=model_deployment,
     instructions="""
     You are a travel assistant that provides information on travel services available from Margie's Travel.
     Answer questions about services offered by Margie's Travel using the provided travel brochures.
     Search the web for general information about destinations or current travel advice.
     """,
     input=input_text,
     previous_response_id=last_response_id,
     tools=[
         {"type": "file_search", "vector_store_ids": [vector_store.id]},
         {"type": "web_search"}
     ]
)
```

## 🔧 Local Verification & Runtime Instructions
### 1. Project Instantiation
Ensure Python 3.13 is configured natively. Spin up your local testing sandbox:
```bash
# Setup local execution environment
python -m venv .venv
.venv\Scripts\Activate.ps1

# Install SDK requirements
pip install -r requirements.txt
```

### 2. Service Provisioning (.env configuration)
Create a .env file at the directory root containing your explicit Azure OpenAI endpoints:
```bash
AZURE_OPENAI_ENDPOINT="your_azure_openai_endpoint"
MODEL_DEPLOYMENT="model_name"
```

### 3. Active Session Authorization
Authenticate your terminal workspace session against your Azure Active Directory tenant:
```bash
az login
```
### 4. Running the Application Engine
Fire up the script from your terminal console:
```bash
python tools-app.py
```

## 📋 Interactive Verification Example
Once the engine initializes and builds the vector database, copy and paste the following sequential prompts to verify the tool routing capabilities:

### Step A: Testing the web_search Router
When prompted for input, paste the following real-time tracking query:

```text
What's happening in San Francisco next month?
```

**Expected Behavior:** The engine maps the request parameters against current timeline restrictions, fires up the Web Search agent, and reports on current live event schedules pulled from public indices.

### Step B: Testing the file_search (RAG) Router
Follow up inside the same session thread with this company-specific inquiry:
```text
What hotels does Margie's Travel offer there?
```

**Expected Behavior:** Maintaining dialogue state, the system acknowledges "there" as San Francisco. Recognizing a company-proprietary inquiry, it invokes the File Search engine over your vectorized internal brochures/ PDFs, extracting precise package configurations.

To exit, simply type quit.
