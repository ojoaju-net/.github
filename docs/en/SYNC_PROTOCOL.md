# Synchronization Protocol and Vector Re-indexing

## 1. Protocol Objective

The **Ojoaju Synchronization Protocol** defines the network interaction interface between the Local Client (`ojoaju-local`) and the Cloud Node VM (`ojoaju-node`).

Its primary objectives are:

1. **Efficiency:** Transfer only the granular changes (JSON Patches) instead of full state snapshots.

2. **Security:** Guarantee that every state mutation is mathematically authorized by the user before execution.

3. **Reactive Intelligence:** Trigger atomic updates on the AI Agent's vector database without requiring full document re-indexing.

## 2. Synchronization Architecture

The synchronization process follows a strictly unidirectional push-based flow from the client to the cloud node:

```
sequenceDiagram
    autonumber
    participant Client as ojoaju-local
    participant VM as ojoaju-node
    participant DB as Commit Log DB
    participant VectorDB as AI Vector Index

    Client->>Client: 1. Generate JSON Patch (Δ) & Local Commit Manifest
    Client->>Client: 2. Sign version_id with Ed25519 Secret Key (SK)
    Client->>VM: 3. POST /api/v1/sync/commit (Payload)
    Note over VM: Cryptographic Verification<br/>Ed25519_Verify(PK, version_id, signature)
    alt Verification Fails
        VM-->>Client: 401 Unauthorized (Invalid Signature)
    else Verification Succeeds
        VM->>DB: 4. Append Commit Manifest to Linear Log
        VM->>VM: 5. Apply JSON Patch to current State Document
        VM->>VectorDB: 6. Trigger Granular Vector Update (Delta Ingestion)
        VM-->>Client: 200 OK (Sync Successful, New Version Acknowledged)
    end
```

## 3. API Contract Specification

### 3.1 Commit Synchronization Endpoint

- **Endpoint:** `POST /api/v1/sync/commit`

- **Content-Type:** `application/json`

#### Request Payload Schema:

```
{
  "version_id": "8f3c6a9d...",
  "parent_id": "7b2e1a4c...",
  "timestamp": "2026-06-20T23:05:00Z",
  "author_node_id": "did:ojoaju:6f8a9b...",
  "patch": [
    {
      "op": "add",
      "path": "/knowledge_graph/verified_capabilities/-",
      "value": {
        "capability": "Distributed systems engineering using Python and FastAPI.",
        "evidence_source": "repo:ojoaju-node",
        "ai_analysis": "Verified code implementation of cryptographic verification middleware."
      }
    }
  ],
  "signature": "a4f8e2d1..."
}
```

#### Expected Responses:

- **`200 OK`**: Commit successfully validated, appended, and applied.
  
  ```
  {
    "status": "synchronized",
    "current_version": "8f3c6a9d...",
    "vector_index_updated": true
  }
  ```

- **`400 Bad Request`**: Out of sync error (the provided `parent_id` does not match the node's current `version_id`). Requires the client to perform a fetch to update its state before retrying.

- **`401 Unauthorized`**: Cryptographic signature verification failed.

## 4. Delta Vector Re-indexing Strategy

To minimize computational costs and execution time on the cloud node, the **Ojoaju AI Agent** never performs full graph vectorization upon receiving a commit. Instead, it reads the JSON Patch array and maps the paths directly to vector partitions.

### Mapping Patches to Vector Stores:

1. **Path Isolation:** The indexing engine parses the `path` attribute of the JSON Patch operations (e.g., `/knowledge_graph/verified_capabilities/2`).

2. **Granular Operations:**
   
   - **`add` / `replace`**: The system takes the `value` object, converts it to an embedding via the LLM pipeline, and adds or overwrites only the single vector corresponding to that path index.
   
   - **`remove`**: The system purges the vector from the index using the unique identifier derived from the path.

3. **Context Enrichment:** When indexing a granular section (like a single project or capability), the engine automatically injects the global `#core_expertise` metadata to ensure the embedding retains full contextual relevance during semantic searches by recruiters.
