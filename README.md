# ElevenLabs Supabase Voice RAG Agent

An n8n workflow that answers questions for an ElevenLabs voice agent from a Supabase vector knowledge base. It returns a one-sentence answer with a source citation for each query.

![Illustration with the n8n and ElevenLabs logos, a cartoon robot wearing a headset and saying Hello, a document with a chart, and a microphone](images/thumbnail.png)

## Overview

Voice agents need short answers that are grounded in company documents. This system receives a query over a **webhook**, retrieves matching document chunks from a Supabase vector store, and passes them to an AI agent built on Google Gemini. The workflow is named for ElevenLabs, and its one-sentence answer format and plain-text output suit speech playback.

The agent keeps conversation history in Postgres, so follow-up questions resolve against earlier exchanges. A Cohere reranker reorders the retrieved chunks before the agent answers.

![n8n canvas showing a Webhook node connected to an AI Agent node, followed by an Edit Fields node and a Respond to Webhook node, with a Gemini chat model, Postgres chat memory, and a Supabase vector store tool using Gemini embeddings attached to the agent](images/ElevenLabs-Voice-RAG-AI-Agent-with-Automated-Document-Ingestion-1.PNG)

## Workflows

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| ElevenLabs Supabase Voice RAG Agent | Webhook (POST) | Answers a query in one sentence using documents retrieved from Supabase, with the source cited. |

## Workflow Details

### ElevenLabs Supabase Voice RAG Agent

This workflow answers each incoming query with a single cited sentence drawn only from the Supabase knowledge base.

**Trigger:** A Webhook node accepts POST requests and holds the response open until a Respond to Webhook node replies.

1. The Webhook node receives the POST request and passes it to the AI Agent node.
2. The AI Agent node loads recent conversation history from the Postgres Chat Memory node, which keeps the last 10 messages for the session.
3. The agent calls the Supabase Vector Database node as a tool. The node embeds the query with Gemini embeddings and retrieves the 10 closest document chunks.
4. The Cohere reranker reorders the retrieved chunks by relevance.
5. A Gemini chat model writes the answer under a system prompt that requires exactly one sentence followed by a source citation line.
6. The "Remove quotation marks" Set node strips double quotation marks from the agent output.
7. The Respond to Webhook node returns the cleaned result to the caller.

**Agent behavior:**

- The system prompt limits answers to information retrieved from the vector store.
- The agent replies with a fixed "no relevant information" message when retrieval returns nothing.
- The agent refuses requests to drop the single-sentence format.

**Memory sessions:** The session key combines the workflow and execution identifiers, so each execution starts a new conversation.

**Error handling:** The workflow settings point to a separate error workflow.

**Branches, loops, and sub-workflows:** None.

**Integrations:** Google Gemini, Supabase, Postgres, Cohere.

## Integrations

- ElevenLabs: the voice agent platform named in the workflow title and sticky note. The workflow contains no ElevenLabs node.
- Google Gemini: chat models and embeddings.
- Supabase: vector database used for retrieval.
- Postgres: chat memory storage.
- Cohere: reranking of retrieved chunks.

## Prerequisites

- An n8n instance with the LangChain AI nodes available.
- A Google Gemini API credential.
- A Supabase API credential and a Supabase project with a vector table and a matching search function.
- A Postgres credential and a Postgres database for chat memory.
- A Cohere API credential.

## Setup

1. Import `workflows/ElevenLabs_Supabase_Voice_RAG_Agent.json` into n8n.
2. Create the Google Gemini, Supabase, Postgres, and Cohere credentials in n8n.
3. Assign each credential to its nodes: the two Gemini chat models, Gemini embeddings, Supabase Vector Database, Postgres Chat Memory, and Cohere reranker.
4. Review the Supabase Vector Database node and confirm it points to your vector table and search function.
5. Review the Postgres Chat Memory node and adjust the session key if you need sessions that span executions.
6. Assign an error workflow in the workflow settings, or remove the existing reference.
7. Activate the workflow and send POST requests to its production webhook URL.
