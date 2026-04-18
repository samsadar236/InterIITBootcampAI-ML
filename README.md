# InterIITBootcampAI-ML
AI Dungeon Master — Memory Architecture & RAG Implementation

This document explains the memory system, RAG integration, and design learnings behind the AI Dungeon Master project. It explores how information flows through short-term and long-term storage, how modular agents manage context, and how token limits are controlled for sustained coherence.

System Overview
This system uses a Retrieval-Augmented Generation (RAG) approach combined with a multi-tier memory hierarchy to maintain narrative coherence across long interactive sessions.

Key Goals

Preserve story consistency over dozens of turns.

Balance creativity (LLM freedom) with coherence (grounded memory).

Keep performance stable and avoid token overflow.

Memory Architecture
The system maintains four hierarchical memory layers, each serving a distinct purpose:

Memory Type Purpose Capacity Persistence Short-Term Memory (STM) Stores last few turns (current context). 10 turns Volatile Recent Turns Buffer Temporary window for extracting updates. 5 turns Temporary Summary Buffer Aggregates turns for periodic summarization. 20 turns Semi-persistent Long-Term Memory (LTM) Structured world model (characters, items, locations, events, decisions). Unlimited (pruned periodically) Persistent Information Flow Between Memories

User Input → Short-Term Memory (STM) Player actions are appended to STM.

STM → DM Agent Response The Dungeon Master responds using recent context.

Recent Turns Buffer → Extractor Node (every 5 turns) Extracts structured facts (NPC updates, locations, items, etc.) Updates Long-Term Memory (LTM) with:

last_updated_turn

relevance_score

Summary Buffer → Summarizer Node (every 10 turns) Summarizes narrative events and appends a condensed entry to LTM events.

LTM → ChromaDB Vector Store LTM is embedded using SentenceTransformers (MiniLM-L6-v2) and stored for retrieval.

ChromaDB → Context Retrieval When STM is insufficient, the Retriever pulls semantically relevant context to ground future responses.

Cleaner Node (every 30 turns or token overflow) Applies exponential decay to memory relevance and prunes outdated or irrelevant records.

Memory Lifecycle Visualization User Input ↓ Short-Term Memory (STM) ↓ every 5 turns Extractor → Long-Term Memory (LTM) ↓ every 10 turns Summarizer → Condensed History ↓ ChromaDB Vector Store ↔ Retriever ↓ every 30 turns or token overflow Cleaner Node → Memory Pruning

RAG Implementation
RAG (Retrieval-Augmented Generation) is integrated via ChromaDB to ensure the model stays grounded in prior world events.

Components

Embedder: all-MiniLM-L6-v2

Vector DB: Chroma

Retrieval: top-3 semantic matches using cosine similarity

Workflow

Each long-term memory update generates text documents describing game entities.

These are embedded and stored as vectors in ChromaDB.

When the DM processes user input, it queries Chroma for related vectors.

Retrieved summaries are injected into the LLM’s system prompt.

This ensures responses remain consistent even after hundreds of turns, by retrieving relevant story context dynamically.

Technical Components LLM Engine
Groq API running Llama 3.1 8B-Instant

Optimized for low-latency inference and fast context processing.

Libraries and Frameworks Category Library Purpose LLM Interface groq Chat completions API Memory Retrieval chromadb Vector database for long-term memory Embeddings sentence-transformers Convert memory text into vector space Graph Orchestration LangGraph Node-based pipeline execution Environment dotenv, os API key and configuration handling Core Algorithms Algorithm Description Retrieval Embedding-based nearest-neighbor search in vector space Summarization Periodic text condensation for memory compression Relevance Decay Exponential decay to simulate forgetting Ranking Top-k retrieval for context relevance filtering Modularity

Each node is a standalone module:

Question Agent: Decides if STM suffices or retrieval is needed.

DM Node: Generates narrative response.

Extractor Node: Extracts structured entities into LTM.

Summarizer Node: Compresses history into summaries.

Cleaner Node: Prunes old or irrelevant data.

These modules communicate through a LangGraph tree, ensuring adaptive flow and conditional routing.

Why a Tree-Based Architecture (LangGraph) Challenge:
Linear pipelines (e.g., LangChain) were inflexible — all nodes ran sequentially, even when not needed.

Solution:

Use LangGraph — a graph-based orchestration system that enables conditional routing and modular independence.

image
Benefits:

Dynamic decision flow (run only needed nodes)

Reduces redundant API calls

Modular debugging and performance tuning

Mirrors human cognition: Recall → Generate → Reflect → Forget

Cleaner Node & Token Counter Purpose:
To prevent memory bloat and token overflow, ensuring system longevity.

Trigger Conditions:

Every 30th turn, or

When token count exceeds the model’s context window (~8K for Llama-3.1 8B).

Mechanism:

Calculate relevance using exponential decay:

relevance = base_score * exp(-decay_rate * age)

Remove entries below a threshold relevance (e.g., 0.1).

Re-index ChromaDB for faster retrieval.

If tokens > threshold, trigger summarization before cleaning.

Outcome:

Maintains optimal memory size.

Preserves recent important context.

Prevents model slowdowns and context drift.

Observations & Learnings Challenge Observation Implemented Fix Outcome Context window overflow Model lost coherence beyond turn 15 Summarization + token counter Sustained coherence up to 50+ turns Latency from redundant retrievals Retrieval triggered unnecessarily Added Question Agent Reduced average latency by ~25% Memory bloat ChromaDB grew indefinitely Cleaner node with decay Stable retrieval performance Summarization drift Some loss of fine details Hybrid summarization (entity + narrative) Preserved story integrity Narrative inconsistency Missed earlier storylines RAG grounding Logical story continuity maintained
Key Insights
Tree-based modularity provides cognitive-like control flow.

Summarization + decay ensures infinite playability within finite tokens.

RAG merges imagination with memory accuracy.

Cleaner Node keeps the system performant and self-sustaining.

Token management is as vital as narrative logic for long-term coherence.

Summary
This architecture creates a self-sustaining narrative intelligence:

It remembers, summarizes, retrieves, and forgets dynamically.

It maintains creative storytelling while ensuring long-term coherence.

It simulates cognitive processes — perception, recall, reflection, and forgetting — in a scalable, modular system.

Demo Video link - "https://drive.google.com/file/d/15imUC6lOVl1tR7CKRer5eWY1KuI1MY9C/view?usp=drive_link"

