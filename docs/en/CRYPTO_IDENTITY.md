# Cryptographic Identity and Signatures

## 1. Sovereign Identity Philosophy

In **Ojoaju**, identity is fully sovereign. No centralized server holds credentials, and we bypass third-party identity providers (e.g., Google, GitHub, or traditional passwords).

The user generates their identity locally (*Local-First*), which guarantees that:

1. No network administrator or intermediary can impersonate the candidate.

2. The cloud VM (the node) acts purely as an execution proxy available 24/7, but does not possess the master keys to manipulate history without authorization.

## 2. User Key Pair

The Ojoaju identity schema leverages the **Ed25519** elliptic curve algorithm for its high computational efficiency, strong security guarantees, and compact signature sizes.

- **Secret Key (**$SK$**):**
  
  - **Location:** Stored exclusively on the user's local machine inside the local client (`ojoaju-local`), encrypted at rest. It never travels across the network, is never uploaded to the cloud VM, and remains unknown to the AI agent.
  
  - **Role:** Digitally signs knowledge graph states (Deltas/Patches) and authorizes decrypting and releasing sensitive identity blocks.

- **Public Key (**$PK$**):**
  
  - **Location:** Stored on the cloud VM and exposed to the public network.
  
  - **Role:** Acts as the user's unique identifier and allows recruiting nodes to verify the mathematical validity of shared knowledge structures.

### 2.1 Node Identifier (Node ID / DID)

To guarantee compatibility with international decentralized standards (W3C), a user's node public address conforms to the Decentralized Identifier (DID) syntax:

$$\text{Node ID} = \text{did:ojoaju:}\langle \text{Base58Encoding}(\text{SHA256}(PK)) \rangle$$

This string serves as the pseudonym with which the AI Agent advertises itself on the network to receive matching requests without revealing civil data.

## 3. Local-to-Cloud Sync Protocol

To secure the cloud VM against remote hijacking and unauthorized CV alterations, every graph update (patch) must be signed locally:

```
[Local Client (ojoaju-local)]                      [Cloud VM (ojoaju-node)]
              |                                               |
              |-- 1. Generate JSON Patch (Δ)                  |
              |-- 2. Compute Hash = SHA256(Δ + Timestamp)     |
              |-- 3. Sign = Ed25519_Sign(SK, Hash)            |
              |                                               |
              |-------- 4. POST /api/v1/agent/sync ---------->|
              |            Payload: { patch: Δ, signature: F} |  -- 5. Fetch user PK
              |                                               |  -- 6. Verify Ed25519_Verify(PK, Hash, F)
              |                                               |  -- 7. If True: Apply Patch & re-index vector DB
              |<------- 8. 200 OK (Synchronized) -------------|
```

If the cloud VM receives a patch with an invalid signature, the operation is immediately rejected, locking out any malicious attempts to hijack the node's profile.

## 4. The Veil of Anonymity and Handshake Protocol

When a recruiter establishes a semantic match and requests access to the human behind the AI Agent, a delegated decryption handshake protocol is initiated:

1. **Reveal Request:** The recruiter sends their own encryption public key ($PK_{\text{recruiter}}$) to the user's node via the `/reveal-request` endpoint.

2. **Local Client Notification:** The cloud node freezes the request and relays it to the local client (`ojoaju-local`) the next time the user connects.

3. **Human Approval:** If the professional accepts the proposal, the local client extracts the sensitive `identity` block (`full_name`, `contact_email`) and encrypts it symmetrically using an ephemeral shared key derived via Diffie-Hellman key exchange between $SK_{\text{user}}$ and $PK_{\text{recruiter}}$.

4. **Release:** The encrypted payload is delivered back to the recruiter. Only the recruiter's secret key ($SK_{\text{recruiter}}$) can decrypt the candidate's personal contact details. The rest of the network continues to see only an anonymous Node ID.