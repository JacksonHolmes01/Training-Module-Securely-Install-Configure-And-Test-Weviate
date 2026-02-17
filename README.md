# Secure Weaviate Vector Database Lab

This repository is a **step-by-step training module** that teaches how to **securely install, configure, and test a Weaviate vector database** using Docker.

It is written for beginners and assumes **no prior experience** with Weaviate, Docker, or vector databases.

This lab is part 1 of a 2 lab course. [Lab 2](https://github.com/JacksonHolmes01/Training-Module-Production-Grade-RAG-Infrastructure-Weaviate/tree/main) covers building a multi-service Retrieval-Augmented Generation (RAG) chatbot using Weaviate, Ollama, FastAPI, NGINX, and Gradio.
It is recommended that you complete lab 1 before moving to lab 2.

---

## Purpose of this repository

Vector databases are a core component of modern AI systems (RAG, semantic search, agents), but they are often deployed **without basic security controls**.

The purpose of this repository is to teach how to:

- Treat a vector database as **infrastructure**, not a black box
- Deploy it in a **secure-by-default** way
- Explicitly configure authentication and authorization
- Verify that security controls actually work
- Understand the operational lifecycle of a vector database

This lab focuses on **deployment, configuration, and security**, not embedding math or retrieval theory.

---

## What you will learn

By completing this lab, you will learn how to:

- Install Weaviate using Docker Compose
- Configure persistent storage for a vector database
- Disable anonymous access
- Enable API key authentication
- Enforce role-based access control (RBAC)
- Test authentication and authorization failures and successes
- Validate network exposure and data persistence
- Safely stop, restart, and reset a database deployment

These are the same skills required to safely run vector databases in real-world environments.

---

## What this lab does NOT cover

This repository intentionally does **not** cover:

- How embeddings are generated
- Vector similarity math (cosine, dot product, etc.)
- Index internals (HNSW tuning)
- Hybrid search or reranking

Those topics belong in separate modules that build on this foundation.

---

## How to use this repository

Follow the documentation pages **in order**. Each page builds on the previous one.

### Start here:

1. Go to the weviate-secure-lab folder and follow the instructions in the README

Do not skip validation steps — they are part of the learning.

---

## Who this repository is for

This lab is suitable for:
- Students learning about AI infrastructure
- Engineers new to vector databases
- Security-focused practitioners evaluating AI systems
- Anyone who wants a **concrete, hands-on** understanding of vector DB deployment

---

## Quick start (after prerequisites)

Once Docker is installed and running:

```bash
docker compose up -d
