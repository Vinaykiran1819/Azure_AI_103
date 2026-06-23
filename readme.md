# Generative AI Chat Application with Azure AI Foundry

An implementation of a multi-turn, contextual Generative AI chat application built using the modern **OpenAI Python SDK**, **Azure Identity**, and **Azure AI Foundry**. 

This repository documents my completion of a comprehensive exercise to build, optimize, and scale an AI-driven chat client using state-of-the-art cloud endpoints, transitioning from legacy paradigms to the modern, context-aware Responses API.

---

## 🚀 Project Overview & Key Learning Objectives

The goal of this project was to move beyond basic API consumption and develop a robust, production-grade CLI chat interface capable of:
* **Secure Authentication:** Leveraging Azure Entra ID via `DefaultAzureCredential` instead of hardcoded API keys.
* **Modern API Integration:** Migrating from the standard `ChatCompletions` endpoint to the streamlined, stateful `Responses` API.
* **State & Context Retention:** Building explicit conversational tracking to enable multi-turn dialogue.
* **User Experience Optimization:** Implementing token streaming (`stream=True`) to resolve terminal UI latency.
* **Asynchronous Scaling:** Building a concurrent, non-blocking client version using Python's `asyncio` framework.

---

## 🛠️ Architecture & Core Mechanics

The client application hooks into a custom model deployment hosted within my Microsoft Foundry Project ecosystem:

* **Host Environment:** Azure AI Foundry Portal (`https://ai.azure.com`)
* **Model Backbone:** `gpt-4.1` (Deployed securely under a dedicated Azure Resource Group)
* **Authentication Flow:** Token-based provider querying Azure Entra ID via the local Azure CLI session credentials (`az login`).

---

## 📂 Repository Structure

```text
├── .env                  # Project Configuration (Endpoints & Deployments)
├── .gitignore            # Ignores local Python virtual environment (.venv)
├── requirements.txt      # Project Dependencies (OpenAI, Azure-Identity)
├── chat-app.py           # Synchronous implementation (ChatCompletions -> Responses + Streaming)
└── chat-async.py         # Asynchronous implementation using AsyncOpenAI & asyncio
```

## 💻 Code Evolution & Implementation Journey
### Phase 1: Authentication & Standard ChatCompletions
Initially, I set up secure Entra ID integration and implemented the classic message-array paradigm.

```bash

# Initializing the credential securely
token_provider = get_bearer_token_provider(
     DefaultAzureCredential(), "[https://ai.azure.com/.default](https://ai.azure.com/.default)"
)
    
openai_client = OpenAI(
     base_url=azure_openai_endpoint,
     api_key=token_provider
)

# Fetching response via legacy ChatCompletions JSON structure
completion = openai_client.chat.completions.create(
     model=model_deployment,
     messages=[
         {"role": "system", "content": "You are a helpful AI assistant."},
         {"role": "user", "content": input_text}
     ]
)

```

## Phase 2: Upgrading to the Modern Responses API & State Tracking
To clean up syntax and optimize model interactions, I refactored the pipeline to use the newer Responses API, while introducing a state tracker (last_response_id) to retain chat history context between inputs.

```bash
# Track response history outside the runtime loop
last_response_id = None

# Context-aware request mapping using explicit ID injection
response = openai_client.responses.create(
             model=model_deployment,
             instructions="You are a helpful AI assistant.",
             input=input_text,
             previous_response_id=last_response_id,
)
last_response_id = response.id  # Cache the latest turn state

```

## Phase 3: Eliminating Latency with Token Streaming
To prevent the application from appearing unresponsive during long generation cycles, I integrated streaming deltas to print text to the terminal in real-time.

```bash
stream = openai_client.responses.create(
             model=model_deployment,
             instructions="You are a helpful AI assistant.",
             input=input_text,
             previous_response_id=last_response_id,
             stream=True
)
for event in stream:
     if event.type == "response.output_text.delta":
         print(event.delta, end="") # Direct real-time terminal output
     elif event.type == "response.completed":
         last_response_id = event.response.id

```

## Phase 4: High-Performance Asynchronous Scaling
Finally, I built a non-blocking variant (chat-async.py) utilizing AsyncOpenAI and azure.identity.aio to release the event loop during network I/O operations.

```bash
# Asynchronous model resolution
response = await async_client.responses.create(
             model=model_deployment,
             instructions="You are a helpful AI assistant.",
             input=input_text,
             previous_response_id=last_response_id
)

```

## 🔧 Installation & Verification Playbook
### 1. Prerequisites
Ensure you have Python 3.13.xx, Git, and the Azure CLI configured on your environment.

### 2. Local Environment Setup
Clone the codebase, initialize your virtual environment, and pull down the packages:

```bash
# Setup environment
python -m venv .venv
.venv\Scripts\Activate.ps1

# Install required dependencies
pip install -r requirements.txt

```

### 3. Application Configuration
Create or configure your .env file at the root of the application directory with your Azure AI Foundry resource information:

```bash
AZURE_OPENAI_ENDPOINT="https://<your-foundry-resource-name>[.openai.azure.com/](https://.openai.azure.com/)"
MODEL_DEPLOYMENT="gpt-4.1"

```

### 4. Cloud Authentication
Authenticate your local device securely against your Azure Active Directory tenant:

```bash
az login
```
### 5. Running the Applications
To run the standard/streamed chat client:

```bash
python chat-app.py

```

To run the highly-performant asynchronous chat client:

```bash
python chat-async.py

```

## 📈 Summary
Through this development handson, I successfully mastered token-based cloud provider architectures, modern state-management configurations within AI clients, and decoupled asynchronous processing design patterns.