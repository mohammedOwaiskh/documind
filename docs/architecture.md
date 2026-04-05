# DocuMind - Architecture

In the modern world, where every transaction is recorded and every data is stored as a document in big financial institutions,retriving a very specific information from this corpus and presenting it in a clean format to the stakeholders becomes very tedious and time-consuming. However, AI can be helpful in this situation, a week-long job can be done in a matter of few-hours with the help of DocuMind. 

DocuMind is a multi-agent document intelligence system that performs multi-hop reasoning to answer complex financial user queries. This document describes the overall system architecture of DocuMind.


## Table of Contents

- [Data Ingestion Pipeline](#1-data-ingestion-pipeline)
   - [Document Loader](#11-document-loader)
   - [Chunking Module](#12-chunking)
   - [Embedding](#13-embedding)
   - [Vector Store](#14-vector-store)
- [Retrieval Layer](#2-retrieval-layer)
   - [Query Expansion & Understanding](#21-query-exapansion--understanding)
   - [Hybrid Search](#22-hybrid-search)
   - [Re-Ranking](#23-re-ranking)
- [Agent Orchestration Layer](#3-agent-orchestration-layer)
- [Infrastructure Layer](#4-infrastructure-layer)
   - [Evaluation Engine](#41-evaluation-engine)
   - [Observability](#42-observability)
   - [Deployment](#43-deployment)
- [Frontend](#frontend)

![System Architecture](documind_system_architecture.svg)

## 1. Data Ingestion Pipeline

The Data Ingestion Pipeline will be designed to Load, Process and Ingest the financial documents PDFs into a vector database. This process will include Chunking and Embedding of the documents to produce indexes which can be retrieved later in the Retrieval Layer. It will be an offline ingestions of the documents, which means it will be triggered by a simple CLI action.

### 1.1. Document Loader

This is the first step in the pipeline. Here the PDF will be handled. It includes cleaning the PDFs, extracting tables, page headers, footers, etc. Since, the financial PDFs are large files with complex file structure, this step becomes very important in the pipeline.

### 1.2. Chunking Module

After the document is loaded in the pipeline, the Chunking module with create chunks of the documents. I will experiment with different chunking strategy (fixed-size, sentence-aware, semantic chunking). Trade-offs between different strategies will be observed and documented. The final strategy will be decided and implemented after the POC.

### 1.3. Embedding Model

After chunking the document, an embedding model will be used to create embeddings of the chunks which will be stored into the database.

### 1.4. Vector Store

[Qdrant](https://qdrant.tech/) is expected to be used to store the vector embeddings. It is an open-source, production-ready vector database that provides some internal features like hybrid search.

## 2. Retrieval Layer

When the user-query is recieved, a REST API is triggered. Then, a query understanding LLM will understand the query, re-write it and then a hybrid search will be used to retrieve the information required to answer to this query.

### 2.1. Query Exapansion & Understanding

A user query can be a simple 1 line text, for e.g., ``"What is the YoY earning of company X in last Y years?"``. In a multi-agent setup, it is necessary to re-write this query into smaller sub-queries which can then be used to retrieve individual information. An LLM is used in this step.

### 2.2. Hybrid Search

Provided by Qdrant out of the box, this strategy combines BM25, keyword based search and vector search to retrieves information. This search specially becomes useful when exact keywords in the search query matters. In this projects, keywords likes `quarter`, `earnings`, or even specific dates, names and figures will be handled by hybrid search.

### 2.3. Re-Ranking

Using a cross-encoder re-ranker, the top-20 results retrieved from hybrid search will then be re-ranked to return top-5 results. This process significantly improves the output quality and leads to less errors and LLM Hallucinations.


## 3. Agent Orchestration Layer

DocuMind is a multi-agent system, which every task is done by a task-specific AI agent. This project is intended to use 4 AI agents, each with a dedicated task from task planning to the final response:

- <b>Orchestrator Agent:</b> This agent will plan the task and route the request to other agents.
- <b>Retrieval Agent:</b> This agent will search for the specific information from the vector database and retrieve them.
- <b>Reasoning Agent:</b> This agent will analyse the retrieved intermediate information and extracts specific facts
- <b>Answer Agent:</b> This agent will answer the query based upon all the retrieved intermediate answer with a proper citation for each of the facts.

## 4. Infrastructure Layer

This layer deals with Evaluation, Observability and Deployments. This layer will distinguish a personal project with a fully implemented production-grade AI system.

### 4.1. Evaluation Engine
In RAG based AI Systems, measuring that output quality is very important. Hence this engine will use RAGAS and an LLM-as-a-judge to score the outputs. Moreover, a golden-test set of 30-50 manually created question-answers pairs will be created to test the system failure's.

### 4.2. Observability
Since, all of the multi-agent system is wrapped under REST endpoints, tracing the request flow in a well-structured JSON logs will be implemented with either LangSmith or Phoenix. Each log will have consistent fields like ``trace_id``, ``model``, ``status``, etc. to ensure better tracability and observability. Moreover, endpoints like,  `/metrics` and `/health` will be exposed. 

### 4.3. Deployment

The system will be deployed through a Docker container to a live URL using Railway or Fly.io. The CI/CD pipeline will be created on GitHub actions, that will run a test suite, an evaluation suite on golden-test set on every push to the main branch


## Frontend

To make this application accessible and presentable to users, a frontend using Streamlit will be developed. 


## Notes

The project is still in the early designing and development phase and majority of the above discuss points are not yet implemented. This repository will be regularly updated and we ask you for your patience until the system gets in action🤗