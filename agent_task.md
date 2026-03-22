# Agentic RAG Architecture & Root Agent Task

## System Overview

A Corporate RAG system utilizing a **Root Agent** connected to 4 distinct MCP (Model Context Protocol) vector-search servers:

1. `user_guides_mcp` — General user tutorials.
2. `help_articles_mcp` — Troubleshooting and FAQs.
3. `contract_templates_mcp` — Standardized legal templates.
4. `user_documents_mcp` — Specific user history and custom documents.

> **Client Requirement:** The architecture and prompt must be fully compatible and optimized for **Azure OpenAI LLMs**.

## Execution Flow

1. The Root Agent receives the user's query.
2. **Query-Intent Classification:** Before querying MCP servers, the agent classifies the query intent (e.g., legal, how-to, account-specific, troubleshooting) to inform how it weights results from each source.
3. The Root Agent interrogates all 4 MCP servers in parallel to fetch top-K relevant chunks from each topic.
4. **Chunk Payload Specification:** Each chunk returned by an MCP server must include: `chunk_id`, `text`, `vector_score`, `source_mcp`, and `metadata` (e.g., document title, last updated date).

## Root Agent Prompt Requirements

Create a comprehensive system prompt for the Root Agent that strictly enforces these rules:

1. **Context Synthesis & Filtering:** Review chunks from all 4 MCPs. Actively discard chunks that have low relevance to the user's core query to avoid context bloat.
2. **Intelligent Prioritization:**
   - Evaluate chunks based primarily on their vector search similarity score.
   - **Hierarchy of Truth:** If information conflicts across sources, explicitly define which source takes precedence (e.g., prioritize specific `user_documents` or `contract_templates` over general `help_articles`).
3. **Response Generation:** Formulate a cohesive answer relying *only* on the most reliable, high-scoring chunks. Do not hallucinate information outside the provided context.
4. **Source Attribution:** Clearly cite which of the 4 sources and which specific chunks were used to synthesize the final answer.
5. **Fallback Mechanism:** If all returned chunks have low scores or lack relevance, explicitly inform the user that the requested information could not be found within the corporate knowledge base.
6. **Azure OpenAI Integration:** Structure the instructions and token boundaries to account for Azure OpenAI deployment specifications (e.g., `max_tokens`, content filtering behavior, API version compatibility).
7. **Clarification Protocol:** If the query is ambiguous and could match multiple domains equally, the agent should ask a clarifying question before generating a speculative answer.
8. **Confidence Indicator:** The agent should output a confidence level (High / Medium / Low) alongside each answer to signal how well-supported it is by the retrieved context.