# 🧠 Advanced Corrective RAG

An advanced **Retrieval-Augmented Generation (RAG)** system built with **LangChain, LangGraph, OpenAI, FAISS, and Tavily**.

This repository demonstrates how a traditional RAG pipeline can be progressively improved with **retrieval refinement, document evaluation, web-search fallback, query rewriting, and ambiguity handling**.

The goal is to build a RAG system that does not blindly trust retrieved documents. Instead, it evaluates the retrieved knowledge, decides whether it is sufficient, and takes corrective actions when the retrieved context is weak or ambiguous.

---

## 🚀 Overview

Traditional RAG follows a relatively simple pipeline:

```text
User Question
      ↓
Document Retrieval
      ↓
Context
      ↓
LLM
      ↓
Answer
```

This approach can fail when:

* Retrieved documents are irrelevant.
* Retrieved information is incomplete.
* The user's question is ambiguous.
* The required information is not available in the local documents.
* The retrieval query does not match the wording of the documents.
* The model generates an answer from insufficient context.

This project addresses these problems by introducing a **corrective RAG workflow**.

```text
                         ┌─────────────────────┐
                         │    User Question    │
                         └──────────┬──────────┘
                                    ↓
                         ┌─────────────────────┐
                         │   Retrieve Docs     │
                         └──────────┬──────────┘
                                    ↓
                         ┌─────────────────────┐
                         │ Evaluate Retrieval  │
                         └──────────┬──────────┘
                                    ↓
                    ┌───────────────┴───────────────┐
                    │                               │
              Relevant / Good                 Weak / Ambiguous
                    │                               │
                    ↓                               ↓
             Refine Context                 Rewrite Query
                    │                               ↓
                    │                         Web Search
                    │                               │
                    └───────────────┬───────────────┘
                                    ↓
                           Refine Context
                                    ↓
                              Generate Answer
                                    ↓
                                  Output
```

---

## ✨ Key Features

### 🔹 Basic RAG

The project starts with a standard RAG pipeline:

* Load PDF documents
* Split documents into chunks
* Generate embeddings
* Store embeddings in FAISS
* Retrieve relevant documents
* Pass retrieved context to an LLM
* Generate an answer based only on the retrieved context

---

### 🔹 Retrieval Refinement

Retrieved documents are not directly passed to the LLM.

Instead, the retrieved context is:

1. Combined
2. Decomposed into individual sentences
3. Evaluated for relevance
4. Irrelevant sentences are removed
5. Relevant sentences are recomposed into refined context

```text
Retrieved Documents
        ↓
Sentence Decomposition
        ↓
Relevance Filtering
        ↓
Remove Irrelevant Information
        ↓
Refined Context
        ↓
LLM
```

This reduces unnecessary information and gives the generator cleaner context.

---

### 🔹 Retrieval Evaluation

The system introduces an LLM-based retrieval evaluator.

Each retrieved document chunk receives a relevance score between:

```text
0.0 ───────────────────── 1.0
Irrelevant                  Highly Relevant
```

The evaluator also provides a reason for the score.

The system then classifies retrieval into different situations:

* **CORRECT** — at least one retrieved chunk is strongly relevant.
* **INCORRECT** — retrieved chunks are largely irrelevant.
* **AMBIGUOUS** — retrieved information has partial relevance but is not sufficiently strong.

This evaluation determines what the RAG system should do next.

---

### 🔹 Web Search Correction

When the local knowledge base cannot provide reliable information, the system can fall back to web search.

The project uses **Tavily Search** to retrieve external information.

```text
Local Retrieval
      ↓
Retrieval Evaluation
      ↓
Insufficient Context
      ↓
Web Search
      ↓
External Documents
      ↓
Context Refinement
      ↓
Answer Generation
```

This allows the system to recover from situations where the required information is not available in the local PDF knowledge base.

---

### 🔹 Query Rewriting

Before performing web search, the user's original question can be transformed into a concise search query.

For example:

```text
Original:
"What are the latest developments in transformer architectures?"

Rewritten:
"latest transformer architecture developments"
```

The rewritten query is optimized for search and can include a recency constraint when the question requires recent information.

---

### 🔹 Ambiguity Handling

The system also handles questions where retrieved documents provide partial or conflicting evidence.

For example:

```text
Question:
"Batch normalization vs layer normalization"
```

If the retrieval evaluator determines that the evidence is neither clearly sufficient nor completely irrelevant, the system treats the query as **AMBIGUOUS** and uses the corrective path.

```text
Question
   ↓
Retrieve
   ↓
Evaluate
   ↓
AMBIGUOUS
   ↓
Query Rewrite
   ↓
Web Search
   ↓
Refine Context
   ↓
Generate Answer
```

---

## 🏗️ Architecture

The complete corrective workflow is implemented using **LangGraph**.

```text
                         START
                           │
                           ▼
                     ┌───────────┐
                     │ Retrieve  │
                     └─────┬─────┘
                           │
                           ▼
                   ┌────────────────┐
                   │ Evaluate Docs  │
                   └───────┬────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
           CORRECT              INCORRECT /
                │                AMBIGUOUS
                │                     │
                │                     ▼
                │              ┌──────────────┐
                │              │ Query Rewrite│
                │              └──────┬───────┘
                │                     │
                │                     ▼
                │              ┌──────────────┐
                │              │ Web Search   │
                │              └──────┬───────┘
                │                     │
                └──────────┬──────────┘
                           ▼
                    ┌─────────────┐
                    │   Refine    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Generate  │
                    └──────┬──────┘
                           │
                           ▼
                          END
```

---

## 📚 Notebook Structure

| Notebook                        | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `1_basic_rag.ipynb`             | Implements the fundamental RAG pipeline                      |
| `2_retrieval_refinement.ipynb`  | Refines retrieved context using sentence-level filtering     |
| `3_retrieval_evaluator.ipynb`   | Evaluates retrieved chunks and determines retrieval quality  |
| `4_web_search_refinement.ipynb` | Adds web-search fallback using Tavily                        |
| `5_query_rewrite.ipynb`         | Rewrites questions into optimized web-search queries         |
| `6_ambiguous.ipynb`             | Handles ambiguous retrieval results using corrective routing |

---

## 🛠️ Tech Stack

### AI / LLM

* **OpenAI GPT-4o-mini**
* **OpenAI text-embedding-3-large**

### RAG

* **Retrieval-Augmented Generation**
* **FAISS**
* **Document chunking**
* **Semantic similarity retrieval**
* **Context refinement**
* **Retrieval evaluation**

### Frameworks

* **LangChain**
* **LangGraph**
* **Pydantic**

### Document Processing

* **PyPDFLoader**
* **RecursiveCharacterTextSplitter**

### Web Search

* **Tavily**

### Configuration

* **python-dotenv**

---

## 📁 Project Structure

```text
Advanced-Corrective-RAG/
│
├── documents/
│   ├── book1.pdf
│   ├── book2.pdf
│   └── book3.pdf
│
├── 1_basic_rag.ipynb
├── 2_retrieval_refinement.ipynb
├── 3_retrieval_evaluator.ipynb
├── 4_web_search_refinement.ipynb
├── 5_query_rewrite.ipynb
├── 6_ambiguous.ipynb
│
└── README.md
```

The notebooks currently expect three PDF knowledge sources inside the `documents/` directory:

```text
documents/book1.pdf
documents/book2.pdf
documents/book3.pdf
```

You can replace these documents with your own knowledge base.

---

## ⚙️ Installation

### 1. Clone the Repository

```bash
git clone https://github.com/rameezuetian/Advanced-Corrective-RAG.git

cd Advanced-Corrective-RAG
```

### 2. Create a Virtual Environment

Using Python:

```bash
python -m venv venv
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install langchain
pip install langchain-community
pip install langchain-openai
pip install langchain-text-splitters
pip install langgraph
pip install faiss-cpu
pip install pypdf
pip install pydantic
pip install python-dotenv
pip install tavily-python
pip install jupyter
```

> Depending on your installed LangChain version, the Tavily integration may require the newer `langchain-tavily` package.

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

### Required API Keys

| Variable         | Purpose                   |
| ---------------- | ------------------------- |
| `OPENAI_API_KEY` | OpenAI embeddings and LLM |
| `TAVILY_API_KEY` | Web search fallback       |

Keep your API keys private and **never commit `.env` to GitHub**.

Add the following to `.gitignore`:

```gitignore
.env
venv/
__pycache__/
.ipynb_checkpoints/
```

---

## ▶️ Running the Project

Start Jupyter Notebook:

```bash
jupyter notebook
```

Then execute the notebooks in order.

### Recommended Learning Path

```text
1_basic_rag.ipynb
        ↓
2_retrieval_refinement.ipynb
        ↓
3_retrieval_evaluator.ipynb
        ↓
4_web_search_refinement.ipynb
        ↓
5_query_rewrite.ipynb
        ↓
6_ambiguous.ipynb
```

Each notebook builds upon the concepts introduced in the previous one.

---

## 🔍 How Corrective RAG Works

The system follows a simple principle:

> **Do not trust retrieval blindly. Evaluate it before generating the answer.**

### Step 1 — Retrieve

The user's question is sent to the FAISS retriever.

```text
Question
   ↓
Embedding
   ↓
FAISS
   ↓
Top-K Documents
```

### Step 2 — Evaluate

Each retrieved document is evaluated by an LLM-based relevance evaluator.

```text
Document + Question
        ↓
   LLM Evaluator
        ↓
Relevance Score
        ↓
Routing Decision
```

### Step 3 — Decide

The system determines whether the retrieved information is sufficient.

```text
             Retrieval
                 │
                 ▼
            Evaluation
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    CORRECT   AMBIGUOUS  INCORRECT
       │         │         │
       └─────────┼─────────┘
                 ▼
             Correction
```

### Step 4 — Correct

If the retrieval is weak:

* Rewrite the query.
* Search the web.
* Retrieve additional information.
* Refine the resulting context.

### Step 5 — Generate

Finally, the LLM generates an answer using the refined context.

---

## 🎯 Why Corrective RAG?

Basic RAG assumes:

```text
Retrieved Context = Correct Context
```

Corrective RAG instead assumes:

```text
Retrieved Context
       ↓
     Evaluate
       ↓
Is it sufficient?
   ↙         ↘
 Yes          No
  ↓            ↓
Refine     Correct Retrieval
  ↓            ↓
  └──────┬─────┘
         ↓
      Generate
```

This provides a more robust architecture for applications where retrieval quality directly affects answer quality.

---

## 🧪 Example

Suppose the knowledge base contains information about deep learning.

User asks:

```text
"What is batch normalization?"
```

The system retrieves relevant documents.

If the evaluator determines that the documents are highly relevant:

```text
Question
   ↓
Local Retrieval
   ↓
CORRECT
   ↓
Context Refinement
   ↓
Answer
```

For a question where the local documents are insufficient:

```text
Question
   ↓
Local Retrieval
   ↓
INCORRECT / AMBIGUOUS
   ↓
Query Rewrite
   ↓
Tavily Web Search
   ↓
Context Refinement
   ↓
Answer
```

---

## 📈 Progressive Evolution

This repository intentionally demonstrates the evolution of RAG systems.

```text
Basic RAG
   │
   ▼
Retrieval Refinement
   │
   ▼
Retrieval Evaluation
   │
   ▼
Web Search Correction
   │
   ▼
Query Rewriting
   │
   ▼
Ambiguity Handling
   │
   ▼
Advanced Corrective RAG
```

Instead of jumping directly into a complex architecture, each notebook introduces one additional capability.

---

## 🧠 Learning Objectives

By working through this repository, you can learn:

* How a basic RAG pipeline works
* How to load and process PDF documents
* How document chunking affects retrieval
* How embeddings work with FAISS
* How to build workflows with LangGraph
* How to evaluate retrieved documents
* How to use structured LLM outputs with Pydantic
* How to refine retrieved context
* How to integrate web search into RAG
* How to rewrite queries for better retrieval
* How to handle ambiguous retrieval results
* How to build conditional RAG workflows
* How to create self-correcting RAG systems

---

## 🔮 Future Improvements

Potential improvements for this project include:

* [ ] Replace deprecated Tavily integration with the current `langchain-tavily` package
* [ ] Add conversation memory
* [ ] Add streaming responses
* [ ] Add source citations to generated answers
* [ ] Add retrieval and generation evaluation metrics
* [ ] Add a frontend using Streamlit or FastAPI
* [ ] Add support for multiple document formats
* [ ] Add persistent vector database support
* [ ] Add reranking models
* [ ] Add hybrid keyword + semantic retrieval
* [ ] Add automated evaluation using RAGAS
* [ ] Add observability with LangSmith
* [ ] Add production-ready API endpoints
* [ ] Add Docker support

---

## ⚠️ Important Notes

This repository is primarily designed as an **educational and experimental implementation of advanced RAG techniques**.

The notebooks use specific local PDF files and API-based models, so you may need to modify:

* Document paths
* OpenAI model names
* Embedding models
* Retrieval parameters
* Relevance thresholds
* Prompt templates
* Tavily configuration

for your own environment.

---

## 🤝 Contributing

Contributions are welcome.

If you would like to improve the project:

1. Fork the repository.
2. Create a new branch.

```bash
git checkout -b feature/your-feature
```

3. Make your changes.
4. Commit your changes.

```bash
git commit -m "Add your feature"
```

5. Push the branch.

```bash
git push origin feature/your-feature
```

6. Open a Pull Request.

---

## 📄 License

No license is currently specified for this repository.

If you intend to distribute or reuse the project publicly, consider adding an appropriate open-source license.

---

## 👨‍💻 Author

**Muhammad Rameez**

GitHub:
https://github.com/rameezuetian

---

## ⭐ Support

If this project helped you understand **Corrective RAG, LangGraph, or advanced retrieval workflows**, consider giving the repository a ⭐ on GitHub.

---

### 🔗 Repository

https://github.com/rameezuetian/Advanced-Corrective-RAG
