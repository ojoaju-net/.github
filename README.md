# Ojoaju Network: The Sovereign, Agent-Driven Knowledge Protocol

**Ojoaju** (from the Guaraní word for *link*, *connection*, or *bond*) is a decentralized, local-first protocol designed for the preservation, verification, and semantic matching of real-world technical and academic capabilities, entirely removing commercial noise, self-promotion, and information asymmetry from the modern talent market.

## The Problem with the Status Quo

Centralized professional networks (such as LinkedIn) have turned talent validation into an engagement-farming social media platform. Algorithms reward constant self-marketing ("noise") over authentic technical and academic experience ("seniority").

This forces professionals to inflate their résumés and artificially optimize them to bypass static applicant tracking systems (ATS), destroying efficiency and transparency in global technical recruitment.

## The Ojoaju Paradigm

Ojoaju breaks this paradigm by returning absolute data sovereignty to the professional through a three-layer hybrid architecture:

1. **Local-First Sovereignty (Local Client):** Users consolidate their authentic knowledge (local source code repos, GitHub history, scientific papers, incomplete research, independent reports) locally on their own machines.

2. **Autonomous Cloud Nodes (Personal VM):** A lightweight Docker image runs 24/7 on a minimal, low-cost virtual machine. It syncs sanitized, structured knowledge from the local client using cryptographically signed patches.

3. **AI Agent Representation:** Each cloud node hosts an AI Agent trained on the user's local knowledge base. This agent evaluates recruiting queries semantically and anonymously. The human's identity is only revealed upon explicit user approval once genuine alignment is verified.

## Repository Organization

The Ojoaju ecosystem is divided into the following modular repositories:

- **`.github` / `ojoaju-docs`:** Central repository containing the global protocol specifications and standards.

- **`ojoaju-local`:** Desktop/CLI client. Responsible for data ingestion, sanitization, cryptographic key pair generation, and signing of patches on the local machine.

- **`ojoaju-node`:** Docker image for the cloud VM. Contains the indexed database, the vector search engine, and the AI Agent designed to answer network queries.

- **`ojoaju-landing`:** Public-facing landing page and developer documentation portal.

## Technical Specifications

You can dive deeper into the low-level architecture designs in our central repository docs:

### English (Default)

- [Knowledge Graph (KG) Specification](docs/en/KNOWLEDGE_GRAPH.md)

- [Cryptographic Identity and Signing](docs/en/CRYPTO_IDENTITY.md)

### Español

- [Especificación del Grafo de Conocimiento (KG)](docs/es/KNOWLEDGE_GRAPH.md)

- [Identidad Criptográfica y Firmas en Origen](docs/es/CRYPTO_IDENTITY.md)