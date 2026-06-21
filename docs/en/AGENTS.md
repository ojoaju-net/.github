# AI Agent Specification and Implementation Parameters

## 1. Overview

In **Ojoaju**, the AI Agent is the active, autonomous representative of the user's Node. Instead of executing queries via traditional keyword searches, recruiters query the network semantically. The user's cloud node agent (`ojoaju-node`) processes these queries against the local **Knowledge Graph (KG)**, acting as a sovereign guardian that evaluates compatibility without revealing the user’s protected identity block.

## 2. Technical Implementation Parameters

To ensure deterministic, highly efficient, and low-cost execution on a micro-VM, the cloud node agent operates under the following hard constraints:

### 2.1 Large Language Model (LLM) Runtime Parameters

- **Recommended Model:** `gemini-2.5-flash` (via API) or a local `Llama-3-8B-Instruct` (Q4_K_M GGUF via llama.cpp for fully self-hosted nodes).

- **Temperature:** $\le 0.2$ (low temperature is mandatory to prevent hallucinations and ensure strict adherence to the facts in the KG).

- **Max Output Tokens:** `1024` tokens (sufficient for compatibility summaries).

- **Context Window Limit:** `8192` tokens.

### 2.2 Vector Database & Embeddings Parameters

- **Embedding Model:** `text-embedding-004` (Google Gemini) or `bge-large-en-v1.5` (local alternative).

- **Vector Dimension:** `768` (Gemini) or `1024` (BGE).

- **Distance Metric:** Cosine Similarity.

- **Match Threshold (**$T_{\text{match}}$**):** $0.75$ (queries scoring below this threshold are rejected automatically with a `no_match` response).

$$\text{Cosine Similarity}(A, B) = \frac{A \cdot B}{\|A\| \|B\|} \ge 0.75$$

## 3. Agent Security Guardrails & System Prompts

The Agent operates as an **untrusted-proxy reader**. Under no circumstances should it access or output fields from the `/identity` block of `KNOWLEDGE_GRAPH.md` unless an ECDH handshake has been fully authorized (see `CRYPTO_IDENTITY.md`).

### 3.1 Core System Prompt (The Agent's Persona)

```
SYSTEM INSTRUCTION:
You are the sovereign AI representative of Knode/Ojoaju user 'did:ojoaju:<NodeID>'.
Your objective is to evaluate compatibility against a recruiter's job/project description using ONLY the verified facts from the provided Knowledge Graph (KG).

CRITICAL SECURITY RULES:
1. NEVER reveal the user's real name, email, phone number, physical address, or current/past employers' exact corporate names unless explicitly instructed by a validated cryptographic handshake token.
2. If asked about contact info or identity, respond strictly with: "Protected by sovereign DID. Please initiate a handshake request."
3. Do NOT hallucinate. If a skill, tool, or project is not present in the KG, state that the candidate does not have verified evidence for it.
4. Focus strictly on factual evidence from the KG (e.g., code analysis, academic credits, research papers, resolved challenges).
```

## 4. Communication & Interaction Workflow

The Agent's lifecycle during a semantic search is governed by the following operational flow:

```
graph TD
    Query[Recruiter Semantic Query] --> Search[Endpoint: /api/v1/agent/match]
    Search --> Embed[Generate Query Vector]
    Embed --> VecSearch[Vector Search over KG Embeddings]
    VecSearch --> Metric{Cosine Similarity >= 0.75?}
    Metric -->|No| Reject[Return: Low Compatibility Match]
    Metric -->|Yes| LLM[Assemble Prompt with KG Context]
    LLM --> Guard{Verify Output contains NO protected Identity Data}
    Guard -->|Leaked| Block[Filter/Sanitize Output]
    Guard -->|Safe| Report[Return Compatibility Report & Node ID]
```

## 5. Documentation Directory Map

Use this index to navigate the complete technical specifications of the **Ojoaju** protocol:

| Specification Document | File Path                                           | Scope and Purpose                                                                               |
| ---------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Global Overview**    | `README.md`                                         | Core philosophy, high-level architecture, and organizational overview.                          |
| **Knowledge Graph**    | [`KNOWLEDGE_GRAPH.md`](./KNOWLEDGE_GRAPH.md "null") | Formal JSON schema for profile data, academic studies, partial credits, and research.           |
| **Sovereign Identity** | [`CRYPTO_IDENTITY.md`](./CRYPTO_IDENTITY.md "null") | Ed25519 asymmetric cryptography, DID generation, and the cryptographic handshake protocol.      |
| **Version Control**    | [`VERSION_CONTROL.md`](./VERSION_CONTROL.md "null") | Linear immutable commit log, JSON Patches, and Roll-Forward recovery math.                      |
| **Sync Protocol**      | [`SYNC_PROTOCOL.md`](./SYNC_PROTOCOL.md "null")     | API endpoints (`POST /api/v1/sync/commit`), signature verification, and delta vector indexing.  |
| **UDD Use Cases**      | [`UDD_USE_CASES.md`](./UDD_USE_CASES.md "null")     | High-level UX flows mapped to low-level backend actions (UC-1 to UC-7).                         |
| **Agent Spec**         | `AGENTS.md` *(This File)*                           | LLM runtimes, prompt guardrails, mathematical similarity thresholds, and vector specifications. |
