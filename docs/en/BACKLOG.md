# AI-Driven Development Backlog: Ojoaju Ecosystem

This document outlines the structured, atomic, and sequential backlog designed for an **AI Developer Agent** to implement the **Ojoaju** ecosystem step-by-step.

Each task is designed under the black-box principle: it features clear requirements, inputs, expected outputs, and technical acceptance criteria based on the Canvas specifications.

```
graph TD
    T1[Task 1: Cryptography & UC-1] --> T2[Task 2: Pairing UC-7]
    T2 --> T3[Task 3: Ingestion & Graph UC-2]
    T3 --> T4[Task 4: Sync Endpoint UC-3]
    T4 --> T5[Task 5: Vector Engine UC-4]
    T5 --> T6[Task 6: Handshake Reveal UC-5]
    T6 --> T7[Task 7: Adaptive CV UC-6]
```

## Phase 1: Cryptographic Identity & Secure Pairing (UC-1 & UC-7)

### Task 1.1: Identity Initialization in `ojoaju-local` (Python)

- **Context:** The local desktop client must be able to autogenerate its sovereign cryptographic identity in a cold environment without relying on any traditional central registration server.

- **Instructions for the AI:**
  
  - Create a Python script named `crypto_identity.py`.
  
  - Use the `cryptography` (or `nacl`) library to generate an **Ed25519** key pair.
  
  - Compute the **Node ID** (DID) as follows:
    
    1. Take the Ed25519 public key bytes.
    
    2. Apply a SHA-256 hash.
    
    3. Encode the result using Base58. The final DID must follow the format `did:ojoaju:<Base58Hash>`.
  
  - Design a function to securely store the private key locally, encrypted with a user-defined master password using AES-256-GCM.

- **Input:** Initial user interaction (or provided master password).

- **Output:** A configuration file named `identity.json` containing: `{"did": "did:ojoaju:...", "public_key_hex": "..."}` and the encrypted private key container.

- **Acceptance Criteria:**
  
  - The generated key must be mathematically valid.
  
  - The private key must never be saved in plaintext under any circumstances.

### Task 1.2: Orphan Initialization and Pairing Endpoint in `ojoaju-node`

- **Context:** When deploying the VM for the first time, it has no owner. It must generate a temporary token that allows the local client to bind securely (UC-7).

- **Instructions for the AI:**
  
  - Develop an initial "orphan" state in FastAPI (`main.py` of `ojoaju-node`).
  
  - On startup, if no owner public key (`OWNER_PUBLIC_KEY`) is detected, the application must generate a strong 32-byte cryptographic token (Hex-encoded) called `Pairing Token` and write it to the standard console output (logs).
  
  - Implement the `POST /api/v1/auth/pair` endpoint:
    
    - Receives: `pairing_token`, `node_id` (client's DID), `timestamp`, and `signature`.
    
    - Validates:
      
      1. That the `pairing_token` matches the one stored in memory.
      
      2. That the `timestamp` is recent (maximum 5-minute window to prevent replay attacks).
      
      3. That the `signature` (Ed25519 signature of the timestamp) is valid using the public key derived from the `node_id`.
  
  - Upon successful validation: permanently save the client's public key as `OWNER_PUBLIC_KEY`, purge the `pairing_token` from memory, and transition the node's status to "Shielded".

- **Acceptance Criteria:**
  
  - Any call to `/api/v1/auth/pair` after the node is shielded must return a `403 Forbidden` error.
  
  - If the pairing token does not match, it must return `401 Unauthorized`.

## Phase 2: Knowledge Ingestion & Data Graph (UC-2 & UC-3)

### Task 2.1: Code Analyzer and Graph Builder in `ojoaju-local`

- **Context:** The user wants to scan their local workspace to structure their real skills in the formal JSON schema format (`KNOWLEDGE_GRAPH.md`).

- **Instructions for the AI:**
  
  - Create a module under `ingestion/scanner.py`.
  
  - Recursively scan a specified local folder looking for:
    
    - Configuration files (`package.json`, `requirements.txt`, `Cargo.toml`) to list dependencies and technologies used.
    
    - Markdown files (`.md`) to extract research summaries or academic notes.
    
    - Git history (reading the last 20 commits and the largest change diff).
  
  - Prepare a system prompt to send this consolidated workspace context to the Gemini API (or a local LLM) to extract: *Demonstrated capabilities, Resolved challenges, and Quantifiable outcomes*.
  
  - Structure the LLM's return payload according to the official JSON schema defined in the Knowledge Graph specification.

- **Acceptance Criteria:**
  
  - The script must ignore files listed in `.gitignore` (to prevent parsing heavy folders like `node_modules` or `.venv`).
  
  - The resulting JSON must pass strict schema validation (use `jsonschema` in Python).

### Task 2.2: Synchronization Endpoint with Version Control in `ojoaju-node`

- **Context:** The local client needs to upload its updates to the cloud node securely using signed deltas (UC-3 & `VERSION_CONTROL.md`).

- **Instructions for the AI:**
  
  - Create the endpoint `POST /api/v1/sync/commit` in `ojoaju-node`.
  
  - The endpoint must receive:
    
    - `patch`: An array of JSON Patch operations (RFC 6902).
    
    - `base_version`: The commit hash upon which the patch is applied.
    
    - `commit_hash`: The self-calculated SHA-256 of the new graph state.
    
    - `signature`: The Ed25519 signature of the `commit_hash` signed by the client's private key.
  
  - The node's middleware must validate that the signature is correct using the `OWNER_PUBLIC_KEY` saved during the pairing phase.
  
  - If the signature is valid, the node applies the patch to the current `knowledge_graph.json` file, recalculates the hash to ensure it matches the `commit_hash`, and saves it to disk as the new active state.

- **Acceptance Criteria:**
  
  - Return `401 Unauthorized` if the signature is invalid.
  
  - Return `409 Conflict` if the `base_version` does not match the active version on the VM (multi-device synchronization conflict), requiring a pull first.

## Phase 3: Semantic Search & Sovereign AI Agent (UC-4)

### Task 3.1: Vector Indexing in `ojoaju-node`

- **Context:** Each time the knowledge graph is successfully updated, the node must semantically re-index the data in a lightweight local vector database to enable fast queries.

- **Instructions for the AI:**
  
  - Integrate **LanceDB** or **ChromaDB** (embedded) as the node's local vector database.
  
  - Create a background script `indexing/vector_store.py` that:
    
    1. Reads the "Demonstrated capabilities", "Projects", and "Research" blocks from the updated `knowledge_graph.json`.
    
    2. Converts each text block into a vector using Google's embedding API (`text-embedding-004`).
    
    3. Saves the vectors in the local DB associated with reference metadata (section ID, record type, veracity score).

- **Acceptance Criteria:**
  
  - Under no circumstances should the `/identity` block of the Graph be vectorized or indexed (this block must never touch the vector database).

### Task 3.2: Semantic Query Endpoint and LLM Orchestrator

- **Context:** A recruiter queries the node anonymously. The agent evaluates compatibility using the LLM under strict privacy safeguards (UC-4).

- **Instructions for the AI:**
  
  - Implement the `POST /api/v1/agent/match` endpoint.
  
  - Receives: `query_text` (e.g., *"Developer experienced in mitigating deadlocks in multi-threaded environments"*).
  
  - Generates the embedding for `query_text` and searches for the top 3 nearest vectors in the local database using cosine similarity.
  
  - If the highest similarity score is below `0.75`, immediately return: `{"match": false, "message": "Low compatibility match for this profile"}`.
  
  - If similarity is $\ge 0.75$, retrieve the matching text fragments and assemble the LLM system prompt using the security guardrails from `AGENTS.md`.
  
  - Call `gemini-2.5-flash` with a temperature of `0.2` to draft the compatibility report.
  
  - **Output Safety Filter:** Before returning the JSON response to the recruiter, scan the LLM output string using regular expressions (regex) and heuristics to ensure no protected personal information (emails, phone numbers, or real names) has leaked.

- **Acceptance Criteria:**
  
  - The endpoint must respond within 3 seconds under normal conditions.
  
  - Any mention of personal data in the LLM's draft report must be automatically sanitized and replaced with `[PROTECTED]`.

## Phase 4: Cryptographic Reveal Handshake (UC-5)

### Task 4.1: ECDH Key Exchange and Secure Reveal

- **Context:** The user has accepted a recruiter's offer and wants to share their contact info (real name, email, etc.) in an encrypted, untamperable manner using asymmetric cryptography.

- **Instructions for the AI:**
  
  - Design a workflow in the local client and cloud node to handle reveal requests.
  
  - The recruiter submits their ephemeral X25519 (ECDH) public key within their reveal request via `POST /api/v1/agent/reveal-request`.
  
  - In `ojoaju-local`, the user views the request. Upon clicking "Accept":
    
    1. The client generates its own X25519 ephemeral key pair.
    
    2. Calculates the shared secret using Elliptic-Curve Diffie-Hellman (ECDH).
    
    3. Derives an AES-256-GCM symmetric key from the shared secret using HKDF.
    
    4. Encrypts the sensitive fields of the `/identity` block using this symmetric key.
    
    5. Transmits the encrypted payload and its ephemeral X25519 public key to the node for the recruiter to pull.

- **Acceptance Criteria:**
  
  - The cloud node must act strictly as an encrypted payload relayer; under no circumstances should the cloud node be able to decrypt the user's identity data in transit (End-to-End Encryption, E2EE).
