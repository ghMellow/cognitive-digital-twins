# Cognitive Digital Twins for 5G Networks
This repository serves as the central hub for the Master's Thesis project: **"Cognitive Digital Twins for 5G Networks"**.

The project proposes a Cognitive Digital Twin (CDT) architecture for 5G radio cells, leveraging an **Agentic Knowledge Graph** approach. The system integrates real-time digital representation via Eclipse Ditto with a coordinated multi-agent framework powered by local LLMs (LangGraph) and structural constraints (Neo4j).

---

## 📂 Project Structure

The project is organized into the following repositories:

### 1. [Theory & Manuscript](https://github.com/ghMellow/cognitive-digital-twins-thesis)
Contains the theoretical foundations, architectural diagrams, bibliography, and the LaTeX source for the thesis manuscript. Also houses the **LLM Wiki** — a curated knowledge base used to augment agent reasoning, including skills for structured PDF ingestion.

* **Key focus:** Perception, Reasoning, Memory, Learning, Adaptation, and Autonomous Decision-making in CDTs.

---

### 2. [Implementation Framework](https://github.com/ghMellow/cognitive-digital-twins-framework)
The core technical engine of the project. It includes the 5G metric simulator, Eclipse Ditto configurations, Neo4j schema, and the LangGraph multi-agent orchestration.

* **Tech Stack:** Python, LangGraph, Neo4j, Eclipse Ditto, Local LLMs (Ollama / Llama 3.1).

---

### 3. [MAS Memory — First Experiment](https://github.com/ghMellow/thesis-cdt-experiment-mas-memory)
The first practical experiment after the theoretical phase. Runs controlled multi-agent tasks comparing **expert vs beginner** role profiles across two setups: same model for both agents (1A, control) vs different models (1B, comparison). Tasks cover math verification and 5G-domain reasoning (anomaly detection, root-cause analysis), scored either programmatically or via a judge LLM.

* **Tech Stack:** Python, LangGraph, Ollama (Qwen3, DeepSeek-R1).

---

### 4. [Local GLM-OCR — PDF to Markdown](https://github.com/ghMellow/local-glm-ocr-pdf-to-markdown)
A local web interface wrapping the [`glm-ocr:latest`](https://ollama.com/library/glm-ocr) multimodal model to convert complex PDFs (charts, tables, figures) into clean Markdown. 

Built specifically to support a **custom skill** in the [LLM Wiki](https://github.com/ghMellow/cognitive-digital-twins-thesis), enabling agents to reliably ingest structured information from research papers and technical documents. The frontend is a convenience layer — the core integration uses the exposed local endpoint directly.

* **Key focus:** Multimodal PDF ingestion, Markdown extraction for RAG pipelines, skill integration with the CDT knowledge base.

---

## ⚖️ Licensing

To facilitate both academic citation and industrial reuse, this project uses a dual-licensing model:

* **Software Framework:** Licensed under the [MIT License](https://opensource.org/licenses/MIT).
* **Thesis & Documentation:** Licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

---

*Developed as a Master's Thesis project by Termine Nicolò.*
