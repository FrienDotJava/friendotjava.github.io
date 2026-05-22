---
title: "AskPeraturan OJK"
date: 2025-01-01
draft: false
tags: ["RAG", "LangGraph", "FastAPI", "Next.js", "ChromaDB", "Mistral"]
description: "A RAG system that answers questions about OJK financial regulations in Bahasa Indonesia, with hybrid retrieval and agentic web fallback."
image: /images/projects/ask-peraturan-ojk.png
---

## Overview
This project is a Retrieval-Augmented Generation (RAG) application that lets users ask questions about OJK (Otoritas Jasa Keuangan) regulations in plain Bahasa Indonesia and get direct, cited answers. The system retrieves relevant content from official POJK regulatory PDFs and generates grounded responses using a large language model, without requiring users to manually search through dense legal documents.

{{< youtube 3275c-0z_Bs >}}
---

## The Problem
OJK regulations are spread across dozens of POJK documents, each containing complex legal language that is hard to navigate for founders, fintech operators, and compliance teams. Finding a specific rule often means downloading multiple PDFs, skimming through pages of text, and still being unsure if the answer is complete. There was no simple way to just ask a question and get a direct, sourced answer. This project solves that by building a conversational interface on top of the official regulatory documents.

---

## Goals & Scope
- Build a question-answering system grounded strictly in official OJK regulatory documents
- Support natural language queries in Bahasa Indonesia with accurate multilingual retrieval
- Ensure answers are always cited and never hallucinated beyond the retrieved context
- Add a web search fallback for cases where the local knowledge base is insufficient
- Evaluate the system rigorously using the RAGAS framework

Out of scope: real-time regulation updates, user authentication, and multi-turn conversation history.

---

## System Design & Architecture

<img src="/images/projects/askperaturan-ojk/arch.png" alt="Architecture Diagram" style="width: 100%; max-width: 700px; border-radius: 8px; display: block; margin: 1rem auto;">

The pipeline works in a sequential, agentic flow orchestrated by LangGraph. When a user submits a question, it first goes through a hybrid retriever that runs BM25 keyword search and ChromaDB semantic search in parallel. The merged results are then reranked by a Cohere cross-encoder to select the top 3 most relevant chunks. A grading node uses the LLM to decide whether those chunks are sufficient to answer the question. If they are not, Tavily web search is triggered as a fallback. Finally, the answer generator produces a response using Mistral Medium 3 with a strict grounding prompt, and tokens are streamed back to the Next.js frontend via Server-Sent Events.

Each step in the pipeline is a separate LangGraph node with a defined state, making the flow easy to extend and debug independently.

---

## Tech Stack
<table class="table table-bordered mt-3 text-light">
  <thead>
    <tr>
      <th>Layer</th>
      <th>Technology</th>
      <th>Why I chose it</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Frontend</td>
      <td><strong>Next.js (TypeScript)</strong></td>
      <td>Modern, responsive UI with built-in SSE support for streaming responses</td>
    </tr>
    <tr>
      <td>API Layer</td>
      <td><strong>FastAPI</strong></td>
      <td>Async, lightweight backend that handles streaming endpoints cleanly</td>
    </tr>
    <tr>
      <td>Orchestration</td>
      <td><strong>LangChain + LangGraph</strong></td>
      <td>Agentic workflow with clear node boundaries and easy state management</td>
    </tr>
    <tr>
      <td>LLM</td>
      <td><strong>Mistral Medium 3</strong></td>
      <td>Strong instruction-following for strict grounding prompts</td>
    </tr>
    <tr>
      <td>Embeddings</td>
      <td><strong>Cohere embed-multilingual-v3.0</strong></td>
      <td>Accurate multilingual embeddings that handle Bahasa Indonesia well</td>
    </tr>
    <tr>
      <td>Vector Store</td>
      <td><strong>ChromaDB</strong></td>
      <td>Lightweight, self-hosted vector database suitable for this document scale</td>
    </tr>
    <tr>
      <td>Retrieval</td>
      <td><strong>BM25 + Semantic + Cross-Encoder</strong></td>
      <td>Three-layer approach captures both keyword and semantic matches, then reranks</td>
    </tr>
    <tr>
      <td>Web Fallback</td>
      <td><strong>Tavily Search</strong></td>
      <td>Reliable web search API for queries not covered by the local knowledge base</td>
    </tr>
    <tr>
      <td>Evaluation</td>
      <td><strong>RAGAS</strong></td>
      <td>Standard RAG evaluation framework covering faithfulness, recall, and precision</td>
    </tr>
    <tr>
      <td>Containerization</td>
      <td><strong>Docker + Docker Compose</strong></td>
      <td>Consistent local and production environments</td>
    </tr>
    <tr>
      <td>CI/CD</td>
      <td><strong>GitHub Actions</strong></td>
      <td>Automated lint, type check, test, and deployment on every push and merge</td>
    </tr>
    <tr>
      <td>Deployment</td>
      <td><strong>Vercel + Render</strong></td>
      <td>Free-tier deployment with Nginx configured for SSE streaming</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Research & Prototyping
The first step was understanding the structure of OJK regulatory documents. POJK PDFs follow Indonesian legal formatting with specific section markers like `BAB`, `Bagian`, `Pasal`, and `Ayat`. I tested plain vector search first and quickly found it struggled with regulation-specific queries that contain exact numbers like "POJK 77/2016" or "Pasal 8 ayat 3". This made clear that keyword search needed to be part of the retrieval strategy alongside semantic search.

### Phase 2: Core Implementation
I built the retrieval pipeline in three layers. BM25 handles exact matches for regulation codes and article numbers, while ChromaDB handles meaning-based queries. Both run in parallel and their results are merged before being passed to the Cohere reranker, which scores each chunk directly against the query and selects the top 3. The agentic grading node was added to handle edge cases where no retrieved chunk sufficiently answers the question, triggering Tavily web search as a fallback before generation.

For document ingestion, each PDF's first page is processed by an LLM to extract a clean, structured title before chunking. Splitting uses legal-aware separators that respect Indonesian regulatory boundaries rather than generic character limits.

### Phase 3: Testing & Iteration
I evaluated the system using RAGAS on 25 test questions covering OJK fintech, lending, and capital market regulations. The strict grounding prompt was iterated several times to reach a faithfulness score of 1.0, which means the model never generates content beyond what the retrieved documents contain. Context precision at 0.75 indicated some retrieved chunks were not fully relevant, which pointed to opportunities for reranker tuning with a larger test set.

---

## Challenges & How I Solved Them

**Challenge 1: Handling regulation-specific keyword queries**  
Plain semantic search struggled with queries that contain exact regulation numbers and article references. A user asking about "POJK 77/2016 Pasal 8" expects the exact document, not a semantically similar one. I solved this by adding BM25 alongside semantic search in an ensemble retriever with equal weighting. Both run in parallel and their results are merged before reranking, which means the system benefits from both exact keyword matching and meaning-based retrieval.

**Challenge 2: Preventing hallucination on unanswerable questions**  
LLMs have a tendency to generate plausible-sounding answers even when the retrieved context does not support them. To address this, I wrote a strict grounding prompt that explicitly instructs the model to refuse answering when information is not present in the documents. The LangGraph grading node also filters out insufficient retrieval results before generation, so the model only sees relevant context. This produced a RAGAS faithfulness score of 1.0 on the test set.

**Challenge 3: Multilingual retrieval accuracy**  
OJK documents are written in formal Bahasa Indonesia with legal terminology that differs significantly from everyday language. Standard English-focused embedding models perform poorly on this. I chose Cohere's `embed-multilingual-v3.0`, which is trained on multilingual data and handles Bahasa Indonesia accurately. Combined with BM25 for keyword matches, retrieval context recall reached 0.96 on the test set.

**Challenge 4: Streaming responses through the pipeline**  
LangGraph's agentic pipeline involves multiple async steps before the final generation node, which made streaming the response tokens back to the frontend non-trivial. I configured FastAPI with Server-Sent Events and set Nginx's `proxy_buffering off` so tokens could flow through without being batched by the reverse proxy.

---

## Results & Impact
- RAGAS Faithfulness of 1.00, the system never hallucinates beyond retrieved documents
- RAGAS Context Recall of 0.96, retrieval finds the right chunks 96% of the time
- RAGAS Answer Correctness of 0.87 against ground truth answers
- Context Precision of 0.75, indicating room for further reranker tuning
- Fully deployed with a public demo on Vercel and Render

---

## What I Learned
This project taught me that retrieval quality matters far more than generation quality in a RAG system. Getting a faithfulness of 1.0 was mostly a prompting and retrieval problem, not a model selection problem. The ensemble retrieval approach was a key insight: neither BM25 nor semantic search alone was sufficient for a legal document domain with specific codes and article references.

I also learned how to build agentic workflows with LangGraph and how to think about each pipeline step as an independent node with its own state. This made debugging much easier because I could inspect the output of each node separately. Setting up SSE streaming through FastAPI and Nginx was another practical skill I picked up from building this project.

---

## Future Work
- Add a scheduled ingestion pipeline so new OJK regulations are automatically indexed without manual runs
- Improve context precision beyond 0.75 by fine-tuning the reranker on a domain-specific dataset of OJK queries
- Build multi-turn conversation support so users can ask follow-up questions within a session
- Add a regulation change tracking feature that highlights what changed between POJK versions

---

## Links
- [GitHub Repository](https://github.com/FrienDotJava/ask-peraturan-ojk)
- [Live Demo](https://ask-peraturan-ojk.vercel.app/)
- [API](https://ask-peraturan-ojk-backend.onrender.com)