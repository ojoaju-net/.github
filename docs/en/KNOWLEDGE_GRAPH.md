# Knowledge Graph (KG) Specification

## 1. Overview

The **Knowledge Graph (KG)** is a unified, sovereign, and user-controlled data structure that consolidates an individual's professional, academic, independent project, and scientific research trajectory.

Unlike traditional rigid profile templates, this graph is designed to:

1. **Support Informality and Partial Completeness:** Allow users to declare unfinished university studies, papers in progress, or independent field research, granting them structured semantic value.

2. **Be Efficiently Consumed by AI Agents:** The data is structured so that a Large Language Model (LLM) can reason about the user's factual capabilities, filtering out commercial hype, buzzwords, and marketing noise.

## 2. Graph Schema Spec (JSON)

```
{
  "meta": {
    "version": "1.0.0",
    "last_updated": "2026-06-20T18:23:00Z",
    "node_id": "did:ojoaju:6f8a9b123456789abcdef"
  },
  "privacy_rules": {
    "allow_anonymous_matching": true,
    "target_salary_currency": "USD",
    "target_salary_min": 6000,
    "reveal_identity_policy": "explicit_approval_required"
  },
  "identity": {
    "hidden_by_default": true,
    "full_name": "User Full Name",
    "contact_email": "user@example.com",
    "location": "City, Country",
    "links": {
      "github": "https://github.com/username",
      "blog": "https://username.github.io"
    }
  },
  "knowledge_graph": {
    "core_expertise": [
      "Distributed Systems", 
      "Software Architecture", 
      "Python"
    ],
    "verified_capabilities": [
      {
        "capability": "Designing asynchronous, high-concurrency architectures.",
        "evidence_source": "local_repo:etherware-cloud",
        "ai_analysis": "The user demonstrates advanced understanding of concurrency handling and service decoupling via message passing."
      }
    ],
    "projects": [
      {
        "id": "project_01",
        "name": "coDueño",
        "role": "Lead Architect & Developer",
        "description": "AI Assistant for the real estate market deployed on GCP.",
        "tech_stack": ["Python", "Cloud Run", "Telegram API", "BigQuery"],
        "solved_challenges": [
          "Ingestion and normalization of analytical data for non-technical users using autonomous agents.",
          "Infrastructure cost optimization through a local-first strategy during early development phases."
        ]
      }
    ],
    "academic_and_research": {
      "studies": [
        {
          "institution": "Universidad de Buenos Aires (UBA)",
          "discipline": "Scientific Computing",
          "degree_type": "Bachelor of Science",
          "status": "partial",
          "details": {
            "completed_credits_percentage": 85,
            "period": "1998 - 2006",
            "reason_for_status": "Early transition to the industrial sector and independent software development.",
            "key_learnings": [
              "Mathematical modeling of complex systems",
              "Advanced numerical calculation",
              "Analysis and design of high-complexity algorithms"
            ]
          },
          "verified": false
        }
      ],
      "scientific_publications": [
        {
          "title": "Heuristic optimization in message-centric systems for high concurrency",
          "authors": ["Author, A.", "Rocha, C. S."],
          "status": "draft",
          "venue": "Private research repository",
          "abstract": "Analysis of race condition mitigation using local knowledge graphs.",
          "links": []
        }
      ],
      "field_research_and_investigations": [
        {
          "id": "investigation_01",
          "topic": "Sovereignty & History: Chronology of historical incidents in the Malvinas Islands",
          "type": "Independent Research",
          "status": "active",
          "description": "Collection and documentary analysis of primary sources on sovereignty incidents.",
          "artifacts_generated": [
            "Chronological database of historical documents",
            "Analytical essays written for personal blog publication"
          ]
        },
        {
          "id": "investigation_02",
          "topic": "Traceability of missing children investigations (1975-1978 Period)",
          "type": "Institutional Project (Sadosky Foundation)",
          "status": "completed",
          "description": "Design and development of a critical data tracking system for the reconstruction of historical memory.",
          "impact_analysis": "Enabled the unification of search criteria and cross-referencing of structured analytical data."
        }
      ]
    }
  }
}
```

## 3. Specific Critical States Specifications

### 3.1 Partial Studies (`status = "partial"`)

The Ojoaju protocol prohibits penalizing incomplete academic backgrounds. When a study is marked as partial, the matching AI processes it according to:

- `completed_credits_percentage`: Determines the depth of the hard theoretical foundation acquired (e.g., completing 85% of a computer science degree indicates a massive algorithmic background).

- `key_learnings`: An explicit list of core domains mastered.

- `reason_for_status`: Provides the professional or personal narrative justifying the early transition to the job market in a constructive way.

### 3.2 Draft Publications (`status = "draft"`)

Allows the user to document cutting-edge research and ongoing projects before passing through traditional peer-review committees. The agent's vector search engine indexes the abstract to match technical recruiters on active complex research.

### 3.3 Field Research and Investigations (`field_research_and_investigations`)

Designed to log contributions of high methodological, social, or historical value that do not fit into traditional corporate roles (e.g., ad-honorem analytical tool development for NGOs, historical data auditing, etc.).