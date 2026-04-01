# NeuroHealth: AI-Powered Health Assistant

```markdown
#  NeuroHealth: AI-Powered Health Assistant

An intelligent conversational agent for accessible healthcare guidance, built with
Retrieval-Augmented Generation (RAG) architecture. NeuroHealth combines LLM-based
medical reasoning with structured clinical knowledge to provide personalized health
guidance, symptom interpretation, urgency assessment, and appointment recommendations.

Built entirely with open-source tools — runs locally with zero paid APIs or cloud dependencies.

---

##  What NeuroHealth Does

NeuroHealth is a conversational health assistant specializing in cardiovascular health that can:

- **Understand intent** — Dynamically classifies user queries (symptom inquiry, medication
  question, appointment guidance, etc.) using LLM-based reasoning, not hardcoded keywords
- **Extract and match symptoms** — Retrieves validated medical content matching user-described
  symptoms through semantic search
- **Assess urgency** — Classifies medical urgency into 5 tiers (Emergency → Informational)
  with conservative safety overrides for life-threatening situations
- **Recommend specialists** — Suggests appropriate doctor types based on retrieved medical
  content metadata, not hardcoded condition-to-specialist mappings
- **Generate grounded responses** — Produces health guidance anchored in validated knowledge
  base content, reducing hallucination risk
- **Maintain conversation context** — Supports natural multi-turn dialogues without requiring
  users to repeat information
- **Enforce safety boundaries** — Detects out-of-scope queries, refuses to diagnose or prescribe,
  and directs emergencies to 911

---

## Architecture

```
The system follows a modular pipeline approach. When a user query is received, it first passes through the Conversation Manager, which stores the message and provides multi-turn context. Next, the Intent Recognition Module uses the LLM to determine the intent, scope, and any clarification needs. The Urgency Assessment Module then evaluates the urgency level with built-in safety overrides for critical situations.

For retrieving relevant medical knowledge, the Symptom Extraction Module leverages RAG by embedding the query, searching the vector database, filtering results, and assembling context. The Appointment Recommendation Module extracts specialist recommendations from the retrieval metadata. A Cross-Validation step ensures that both triage and retrieval signals confirm the scope of the response.

Finally, the Response Formatting Module selects the appropriate strategy, assembles the prompt, and generates the final response, which includes health guidance, appointment recommendations, and necessary safety disclaimers.
```

### Tech Stack

| Component | Technology |
|---|---|
| LLM | Llama 3.1 via Ollama |
| Embeddings | BGE-large-en-v1.5 (sentence-transformers) |
| Vector Database | ChromaDB |
| Language | Python 3.11+ |

---

## Project Structure

```
The project is organized under the neurohealth/ directory. The knowledge_base/ folder contains all the medical knowledge stored as Markdown files, covering topics such as coronary artery disease, heart failure, arrhythmias, hypertension, heart valve disease, cardiac medications, heart-healthy lifestyle, cardiovascular screening, cardiac procedures, and special topics. The neurohealth_chroma_db/ directory is automatically created at runtime and serves as the persistent storage for ChromaDB. The main application logic resides in neurohealth.ipynb, which contains all the code cells for running the assistant.
```

---

##  Quick Start

### Prerequisites

**1. Install Ollama**

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows: download from https://ollama.com/download
```

**2. Pull Llama 3.1**

```bash
ollama pull llama3.1:latest
```

**3. Start Ollama server**

```bash
ollama serve
# Keep this terminal open
```

**4. Install Python dependencies**

```bash
pip install sentence-transformers chromadb pyyaml requests
```

### Running NeuroHealth

**1. Clone the repository**

```bash
git clone https://github.com/molathotisaiteja1267-ctrl/neurohealth-prototype.git
cd neurohealth-prototype
```

**2. Open the notebook**

```bash
jupyter notebook neurohealth.ipynb
# or
jupyter lab neurohealth.ipynb
```

**3. Run all cells sequentially (Cell 1 through Cell 12)**

| Cell | What it does |
|---|---|
| Cell 1 | Loads configuration |
| Cell 2 | Creates knowledge base Markdown files in `knowledge_base/` |
| Cell 3 | Parses and chunks the knowledge base |
| Cell 4 | Embeds chunks and indexes into ChromaDB (~1-2 min first run) |
| Cell 5 | Connects to Ollama / Llama 3.1 |
| Cell 6 | Initializes the triage engine (intent + urgency) |
| Cell 7 | Initializes the RAG pipeline |
| Cell 8 | Initializes conversation manager |
| Cell 9 | Builds the core orchestrator |
| Cell 10 | Prepares the chat interface |
| Cell 11 | Prepares the evaluation framework |
| Cell 12 | **Launch** — start interactive chat or run evaluation |

**4. Start chatting**

In Cell 12, the interactive chat starts automatically. Type your health question and press Enter.

```
🧑 You: I've been feeling chest tightness when I climb stairs

⏳ Thinking...

🟡 🤖 NeuroHealth (4.2s):

Based on your description, the chest tightness during physical exertion that
resolves with rest is consistent with what's known as **stable angina**...

I'd recommend scheduling an appointment with a **Cardiologist** within the
next few days for proper evaluation...
```

### Chat Commands

| Command | Action |
|---|---|
| `/new` | Start a new conversation |
| `/info` | Show session info |
| `/debug` | Toggle debug mode (shows triage/retrieval details) |
| `/quit` | Exit |

---

## 📊 Evaluation

NeuroHealth includes a built-in evaluation framework. In Cell 12, comment out `chat.run()` and
uncomment the evaluation commands:

```python
# Run full automated test suite
eval_results = evaluator.run_evaluation(verbose=True)

# Test multi-turn conversation coherence
conv_results = evaluator.run_conversation_test(verbose=True)

# Test adversarial edge cases
adv_results = evaluator.run_adversarial_test(verbose=True)
```

### Test Categories

- **Emergency detection** — Validates urgency assessment for life-threatening scenarios
- **Scope validation** — Tests intent recognition boundaries (rejects out-of-scope queries)
- **Symptom interpretation** — Evaluates RAG retrieval quality for symptom descriptions
- **Medication queries** — Tests knowledge grounding accuracy
- **Appointment routing** — Validates specialist recommendation appropriateness
- **Multi-turn coherence** — Tests conversation context across realistic dialogue flows
- **Adversarial robustness** — Probes prompt injection, prescription requests, empty inputs

---

## 📚 Adding New Topics to the Knowledge Base

NeuroHealth's knowledge base is designed for easy extension. To add a new medical topic:

**1. Create a new `.md` file in `knowledge_base/`**

```markdown
---
topic: Your Topic Category
subtopic: Specific Condition Name
aliases: other names, abbreviations
specialist: Relevant Specialist Type
---

# Condition Name

## Overview
Description of the condition...

## Symptoms
Common symptoms include...

## Causes and Risk Factors
The condition is caused by...

## Diagnosis
Diagnosis involves...

## Treatment
Treatment options include...

## When to See a Doctor
Schedule an appointment if...

## Emergency Signs
Call 911 immediately if...
```

**2. Re-run Cell 3 and Cell 4** to re-chunk and re-index the updated knowledge base.

That's it. No code changes needed. The system automatically picks up the new content, chunks it
with heading-aware processing, indexes it with the specialist metadata, and uses it in future queries.

---

##  Safety Design

NeuroHealth implements multiple layers of safety:

- **LLM-driven urgency assessment** with conservative emergency override — whenever the
  emergency flag is true, urgency is forced to EMERGENCY regardless of other signals
- **Dual-signal scope cross-validation** — triage classification is confirmed against retrieval
  similarity scores to reduce both false rejections and false acceptances
- **No diagnoses or prescriptions** — the system prompt and instruction templates explicitly
  prevent diagnostic or prescriptive responses
- **Consistent disclaimers** — every response includes appropriate safety disclaimers recommending
  professional medical consultation
- **Local-first architecture** — all processing runs locally; user queries never leave the machine
- **No data retention** — conversation history exists only in memory and is discarded on session end

---

##  Important Disclaimer

**NeuroHealth is an informational health assistant, NOT a medical device or diagnostic tool.**

- It does not provide medical diagnoses or prescriptions
- It should not be used as a substitute for professional medical advice
- In any medical emergency, call 911 (or your local emergency number) immediately
- Always consult a qualified healthcare professional for medical decisions

The system is designed for educational and research purposes. It is currently specialized in
cardiovascular health topics.

---

##  Technical Details

### Key Design Decisions

- **No hardcoded keywords or rules** — All intent recognition, scope detection, urgency
  assessment, and emergency detection is handled dynamically by the LLM
- **Markdown-aware chunking** — Preserves heading hierarchy so each chunk carries its full
  context (e.g., "Heart Failure > Symptoms")
- **Metadata-driven specialist routing** — Specialist recommendations come from knowledge
  base YAML frontmatter, not code-level mappings
- **BGE instruction-based retrieval** — Queries use a retrieval instruction prefix for improved
  embedding quality while documents are embedded as-is
- **Configurable thresholds** — Relevance filtering, similarity thresholds, and context window
  sizes are all configurable in the `NeuroHealthConfig` class

### Configuration

Key settings in Cell 1 (`NeuroHealthConfig`):

```python
EMBEDDING_MODEL = "BAAI/bge-large-en-v1.5"
LLM_MODEL = "llama3.1:latest"
OLLAMA_BASE_URL = "http://localhost:11434"
CHUNK_SIZE = 500          # max words per chunk
CHUNK_OVERLAP = 50        # overlap between chunks
TOP_K = 6                 # retrieval results per query
RELEVANCE_THRESHOLD = 0.35  # minimum similarity score
TEMPERATURE = 0.15        # LLM temperature (low for factual consistency)
MAX_HISTORY_TURNS = 10    # conversation context window
```

### System Requirements

- **Python** 3.11+
- **RAM:** 16GB minimum (for embedding model + ChromaDB)
- **Disk:** ~2GB (embedding model download) + knowledge base index
- **GPU:** Not required (runs on CPU). GPU accelerates embedding generation.
- **Ollama:** Running with `llama3.1:latest` pulled

---

##  License

This project is open-source and available under the [MIT License](LICENSE).

---

##  Acknowledgments

This project is developed as part of the **Google Summer of Code 2026** program with
**UC Santa Cruz Open Source Program Office (UCSC OSPO)** under the
**Open Source Research Experience (OSRE) 2026**.

**Mentors:**
- Linsey Pang (Distinguished Scientist, PayPal)
- Bin Dong (Research Scientist, Lawrence Berkeley National Laboratory)

**References:**
- Singhal et al. (2023). "Large Language Models Encode Clinical Knowledge." *Nature*
- Singhal et al. (2023). "Towards Expert-Level Medical Question Answering with Large Language Models." *arXiv*
- Nori et al. (2023). "Capabilities of GPT-4 on Medical Challenge Problems." *arXiv*
- Lewis et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *NeurIPS*
- MedlinePlus Medical Encyclopedia — https://medlineplus.gov/
```
