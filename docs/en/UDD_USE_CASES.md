# User-Driven Development (UDD) Use Cases

This document details the key user stories and interaction flows guiding the development of the **Ojoaju** ecosystem. Each use case defines the high-level user experience (UX) and its corresponding low-level technical counterpart.

```
graph TD
    UC1[UC-1: Identity Initialization] --> UC7[UC-7: Deployment and Secure Binding]
    UC7 --> UC2[UC-2: Data Ingestion]
    UC2 --> UC3[UC-3: Cloud Synchronization]
    UC3 --> UC4[UC-4: Anonymous Search]
    UC4 --> UC5[UC-5: Identity Reveal]
    UC2 --> UC6[UC-6: Adaptive Resume Generation]
```

## UC-1: Cryptographic Identity Initialization

### User Scenario (UX)

> "I install the Ojoaju desktop application on my computer for the first time. I want to set up my environment securely, without having to register with an email or password on a central server, and receive my unique address on the network."

### Interaction Flow

1. The user runs the local application (`ojoaju-local`).

2. The user clicks the "Generate New Sovereign Identity" button.

3. The application prompts the user to move the mouse randomly to add physical entropy to the cryptographic generator.

4. The user is presented with their public **Node ID** (e.g., `did:ojoaju:6f8a9b...`) and a backup file encrypted with a master password chosen by the user.

### Low-Level Technical Logic

- Generation of a local entropy seed for the **Ed25519** key pair.

- Writing the key pair to the operating system's secure storage (Keychain on macOS, Credential Manager on Windows, or Secret Service API on Linux).

- Computation of the SHA256 cryptographic hash of the public key and Base58 encoding to generate the `Node ID` (DID).

- Export of the public key ($PK$) and registration in the local client's configuration file.

## UC-7: VM Deployment and Secure Binding (Secret Handshake)

### User Scenario (UX)

> "I have rented an affordable virtual machine (VM) in the cloud and want to deploy my Ojoaju node (`ojoaju-node`). I need to ensure my local client connects exclusively to my VM and that no attacker discovering my IP address can control my node or intercept my searches. To achieve this, I perform a secure pairing with a unique secret."

### Interaction Flow

1. The user spins up the Docker image of `ojoaju-node` on their cloud VM.

2. Upon first boot, the VM's Docker container console autogenerates a single-use secure access token (**Pairing Token**) and prints it to the standard log output.

3. The user copies the IP address (or domain) of their VM and the Pairing Token from their cloud provider's console.

4. In their desktop client (`ojoaju-local`), the user navigates to the "Connect Cloud Node" section and enters the VM's IP and the Pairing Token.

5. The client performs an automatic connection test. After a moment, the screen updates with a green message: *"VM successfully connected and cryptographically shielded"*. From this moment on, the VM's Pairing Token self-destructs, and the VM no longer accepts further pairing attempts.

### Low-Level Technical Logic

- **Orphan Initialization:** On a cold start, the VM (`ojoaju-node`) detects that it has no associated authorized public key. The system generates a strong 32-byte cryptographically secure random secret (encoded in Base64/Hex), stores it temporarily in its in-memory database, and prints it to the system logs.

- **Node Reclamation Protocol:** The local client sends a `POST /api/v1/auth/pair` request to the VM with the following payload:
  
  - The original `Pairing Token`.
  
  - The user's `Node ID` ($PK$).
  
  - An Ed25519 signature of a current timestamp signed with the local private key ($SK$).

- **Node Shielding:** The VM validates that the Pairing Token matches and that the signature is mathematically correct. Upon success, the VM:
  
  1. Permanently stores the user's $PK$ in its internal configuration file as the **Only Authorized Signature (Owner PK)**.
  
  2. Definitively purges the Pairing Token from its memory.
  
  3. Rejects any future connection to the `/sync` or `/identity` endpoints that do not carry an authorization header signed by the Owner's private key.

## UC-2: Knowledge Ingestion and Graph Generation

### User Scenario (UX)

> "I want the platform to consolidate my real skills. I point Ojoaju to my local code repositories, my GitHub profile, and some draft research papers I wrote in Markdown. I want the system to process this locally and show me what factual capabilities it detected, without revealing sensitive code."

### Interaction Flow

1. The user enters the "Knowledge Sources" section in their local client.

2. The user adds a local folder containing their development projects and links their GitHub account.

3. The user clicks "Analyze Sources".

4. A progress bar shows the extraction. Once finished, the user sees an interactive interface with cards representing:
   
   - **Proven technical capabilities** (e.g., *"Resolution of race conditions in Python threads"*).
   
   - **Structured projects** with their resolved challenges.
   
   - **Academic and research history** (including incomplete studies with their context).

5. The user can edit the texts generated by the local AI, remove capabilities they do not wish to expose, or add manual notes before saving.

### Low-Level Technical Logic

- **Local Scraping:** The client scans the local file system, parsing `.git` folders, configuration files (such as `package.json`, `requirements.txt`), and text documents.

- **Local LLM Processing:** Calls a local LLM model (e.g., via Llama.cpp or Ollama) to summarize and extract resolved challenges and technical capabilities based on factual evidence.

- **Graph Construction:** Generation of the JSON file strictly adhering to the schema defined in `KNOWLEDGE_GRAPH.md`.

## UC-3: Cloud Synchronization and Publication

### User Scenario (UX)

> "Once I have verified my knowledge graph locally, I want to upload it to my cloud node so it is available 24/7 for recruiter searches, with the security that no one along the way can alter my data."

### Interaction Flow

1. The user clicks the "Sync with Cloud" button.

2. The application displays a fast transfer indicator.

3. Once completed, the panel displays the status: **Synchronized - Version V3 (Active)**.

4. The user can view the previous change history and, if desired, click "Restore previous version" to revert to a prior profile state.

### Low-Level Technical Logic

- **Delta Generation:** Comparison of the current JSON against the last locally saved version to generate an array of JSON Patch (RFC 6902) operations.

- **Cryptographic Signature:** Computation of the commit hash and signing with the user's Ed25519 private key.

- **Transmission:** HTTPS POST request to the `/api/v1/sync/commit` endpoint of the cloud VM (specified in `SYNC_PROTOCOL.md`).

- **Node Update:** The VM validates the signature, applies the patch, and atomically updates the AI Agent's vector index.

## UC-4: Anonymous Semantic Search (Recruiter)

### User Scenario (UX)

> "As a tech lead or recruiter, I need to find a distributed systems specialist who has solved real concurrency problems. I search the network, and the system returns a list of anonymous agents matching the profile, explaining exactly each one's technical evidence without revealing names or contact details."

### Interaction Flow

1. The recruiter logs into their own node or search portal and types: *"Engineer who has migrated a monolith to microservices while controlling data loss"*.

2. The system distributes the semantic query.

3. Our user's VM AI Agent receives the query, analyzes its internal Knowledge Graph, and automatically responds: *"I have a 94% match. The user designed an event-driven architecture that mitigated bottlenecks in production, using RabbitMQ and Python."*

4. The recruiter sees the anonymous profile `did:ojoaju:6f8a9b...` with the factual project details, but with the contact fields hidden.

### Low-Level Technical Logic

- Receipt of the vector query at the `/api/v1/agent/match` endpoint.

- Cosine similarity computation between the embedding of the recruiter's query and the user's indexed local vectors.

- Context processing by the node's AI to draft a compatibility report without extracting data from the protected `identity` blocks.

## UC-5: Identity Reveal Request and Approval (Handshake)

### User Scenario (UX)

> "A recruiter saw my AI agent's anonymous report on my concurrency experience and sent me a concrete technical interview offer. I want to evaluate the offer and, if interested, securely reveal my contact details to them."

### Interaction Flow

1. The recruiter clicks "Request Contact Details" in their panel, attaching the job description and the offered salary range.

2. The user's local client receives a notification: *"Offer received for did:ojoaju:6f8a9b... - Role: Senior Backend - Range: $7000 USD"*.

3. The user reviews the proposal. If interested, they click "Accept and Exchange Data".

4. Instantly, the recruiter receives the candidate's real name, email, and portfolio, while the candidate securely receives the recruiter's contact information.

### Low-Level Technical Logic

- Recruiter's request to the `/api/v1/agent/reveal-request` endpoint, sending their public key $PK_{\text{recruiter}}$.

- Temporary storage of the request on the node in a pending state.

- The local client downloads the request during the next sync cycle and decrypts/displays it to the user.

- If approved, a secure channel is generated using an Elliptic-Curve Diffie-Hellman (ECDH) key exchange to encrypt the `identity` block fields at the origin and send them encrypted to the recruiter's node.

## UC-6: Adaptive Resume Generation for Traditional Channels

### User Scenario (UX)

> "I have to apply to a traditional company that is not on the Ojoaju network, and they are asking for a classic PDF resume. I want my local client's AI to write a resume optimized for that job offer using only the true, factual data from my Knowledge Graph."

### Interaction Flow

1. The user goes to the "Export Traditional CV" section in the local client.

2. The user copies and pastes the text of the traditional job offer they want to apply for.

3. The user selects the desired design template.

4. The user clicks "Generate Optimized CV".

5. The local application analyzes which projects, publications, and research from their Knowledge Graph are most relevant to that specific offer and generates an impeccably written, factual PDF and Markdown file.

### Low-Level Technical Logic

- Submission of the job offer text and the entire structure of the Knowledge Graph (`KNOWLEDGE_GRAPH.md`) to a local LLM model or the user's external API.

- Execution of a semantic template that reorganizes and prioritizes the JSON elements.

- Generation of the final document in structured Markdown or PDF format for immediate download.
