# ElevenLabs Supabase Voice RAG Agent

An n8n workflow that answers questions over a webhook. It retrieves documents from a Supabase vector database and replies with a one-sentence, cited answer.

![Illustration with the n8n and ElevenLabs logos, a robot with a headset saying Hello, a document icon with a magnifying glass, and a microphone](images/thumbnail.png)

## Overview

The project contains one workflow. An application sends a question to a webhook. An AI agent searches a Supabase vector store, reranks the matches, and writes an answer grounded in the retrieved documents. A Postgres table stores the conversation so follow-up questions keep their context. The workflow returns the cleaned answer to the caller.

![Workflow canvas showing a Webhook node feeding an AI Agent node, which connects to an Edit Fields node and then a Respond to Webhook node. The agent has a Gemini chat model, Postgres chat memory, and a Supabase vector database tool with Gemini embeddings attached.](images/ElevenLabs-Voice-RAG-AI-Agent-with-Automated-Document-Ingestion-1.PNG)

## Workflows

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| ElevenLabs Supabase Voice RAG Agent | Webhook (POST) | Answers a question in one cited sentence using documents retrieved from Supabase. |

## Workflow Details

### ElevenLabs Supabase Voice RAG Agent

This workflow turns an incoming question into a single-sentence, source-cited answer drawn from a vector knowledge base.

**Trigger:** A **Webhook** node accepts POST requests and holds the response open until a Respond to Webhook node replies.

Execution steps:

1. The Webhook node receives the POST request and passes it to the AI Agent node.
2. The AI Agent node loads the last 10 messages from Postgres Chat Memory for the current session.
3. The agent calls the Supabase Vector Database tool, which embeds the question with Gemini Embeddings.
4. The tool retrieves the 10 closest document chunks from Supabase.
5. The Cohere reranker reorders those chunks by relevance.
6. The Gemini 2.5 Flash model writes the answer from the reranked chunks.
7. The Set node removes double quotation marks from the answer.
8. The Respond to Webhook node returns the cleaned answer to the caller.

Agent behavior defined in the system prompt:

- The answer is exactly one sentence, followed by a source citation on a separate line.
- The agent uses only information retrieved from the vector database.
- When retrieval returns nothing, the agent replies that no relevant information was found in the knowledge base.
- The agent uses chat memory to resolve references to earlier questions.

Model fallback:

- Gemini 2.5 Flash is the primary chat model.
- Gemini 2.5 Flash Lite is the fallback chat model.

Error handling: the workflow settings specify an error workflow that runs when an execution fails.

Branches, loops, and sub-workflow calls: none.

Integrations: Google Gemini, Supabase, Postgres, Cohere.

## Integrations

- Google Gemini (chat models and embeddings)
- Supabase (vector store)
- Postgres (chat memory)
- Cohere (reranker)

## Prerequisites

- An n8n instance with the LangChain (AI) nodes available.
- A Google Gemini (PaLM) API credential.
- A Supabase API credential and a Supabase project with a vector table and a matching search function.
- A Postgres credential for a database that n8n can write to.
- A Cohere API credential.
- An existing n8n workflow to serve as the error workflow, if you keep the error workflow setting.

## Setup

1. Import `workflows/ElevenLabs_Supabase_Voice_RAG_Agent.json` into n8n.
2. Create the Google Gemini, Supabase, Postgres, and Cohere credentials in n8n.
3. Assign each credential to its nodes: both Gemini chat models and Embeddings Gemini, the Supabase Vector Database, Postgres Chat Memory, and Reranker Cohere.
4. Review the Supabase table and search function settings on the Supabase Vector Database node.
5. Select an error workflow in the workflow settings, or clear the setting.
6. Activate the workflow.
7. Send a POST request to the production webhook URL shown on the Webhook node.
