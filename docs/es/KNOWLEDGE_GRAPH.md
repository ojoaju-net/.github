# Grafo de Conocimiento (Knowledge Graph - KG)

## 1. Visión General

El **Grafo de Conocimiento (KG)** es la estructura de datos unificada, soberana y controlada por el usuario que consolida su trayectoria profesional, académica, proyectos independientes e investigaciones científicas.

A diferencia de los esquemas rígidos tradicionales, este grafo está diseñado para:

1. **Soportar la informalidad y la completitud parcial:** Permite declarar estudios universitarios no finalizados, papers en desarrollo o investigaciones independientes, dándoles valor semántico de manera estructurada.

2. **Ser consumido eficientemente por Agentes de IA:** Los datos están estructurados para que un Modelo de Lenguaje (LLM) pueda razonar sobre las capacidades fácticas reales del usuario, abstrayendo la "propaganda" o el ruido comercial.

## 2. Esquema de Datos del Grafo

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
    "full_name": "Nombre del Usuario",
    "contact_email": "usuario@example.com",
    "location": "Ciudad, País",
    "links": {
      "github": "https://github.com/username",
      "blog": "https://username.github.io"
    }
  },
  "knowledge_graph": {
    "core_expertise": [
      "Sistemas Distribuidos", 
      "Arquitectura de Software", 
      "Python"
    ],
    "verified_capabilities": [
      {
        "capability": "Diseño de arquitecturas asíncronas de alta concurrencia.",
        "evidence_source": "local_repo:etherware-cloud",
        "ai_analysis": "El usuario demuestra un entendimiento avanzado en el manejo de concurrencia y desacoplamiento de servicios mediante paso de mensajes."
      }
    ],
    "projects": [
      {
        "id": "project_01",
        "name": "coDueño",
        "role": "Lead Architect & Developer",
        "description": "Asistente de IA para el sector inmobiliario desplegado en GCP.",
        "tech_stack": ["Python", "Cloud Run", "Telegram API", "BigQuery"],
        "solved_challenges": [
          "Ingestión y normalización de datos analíticos para usuarios no técnicos usando agentes autónomos.",
          "Optimización de costos de infraestructura mediante estrategia local-first en etapas de desarrollo."
        ]
      }
    ],
    "academic_and_research": {
      "studies": [
        {
          "institution": "Universidad de Buenos Aires (UBA)",
          "discipline": "Computación Científica",
          "degree_type": "Licenciatura",
          "status": "partial",
          "details": {
            "completed_credits_percentage": 85,
            "period": "1998 - 2006",
            "reason_for_status": "Transición temprana hacia el sector industrial y desarrollo de software independiente.",
            "key_learnings": [
              "Modelado matemático de sistemas complejos",
              "Cálculo numérico avanzado",
              "Análisis y diseño de algoritmos de alta complejidad"
            ]
          },
          "verified": false
        }
      ],
      "scientific_publications": [
        {
          "title": "Optimización de heurísticas en sistemas message-centric para alta concurrencia",
          "authors": ["Autor, A.", "Rocha, C. S."],
          "status": "draft",
          "venue": "Repositorio privado de investigación",
          "abstract": "Análisis sobre la mitigación de condiciones de carrera mediante el uso de grafos de conocimiento locales.",
          "links": []
        }
      ],
      "field_research_and_investigations": [
        {
          "id": "investigation_01",
          "topic": "Soberanía e Historia: Registro de incidentes históricos en las Islas Malvinas",
          "type": "Independent Research",
          "status": "active",
          "description": "Recopilación y análisis documental de fuentes primarias sobre incidentes de soberanía.",
          "artifacts_generated": [
            "Base de datos cronológica de documentos históricos",
            "Ensayos analíticos para publicación en blog personal"
          ]
        },
        {
          "id": "investigation_02",
          "topic": "Trazabilidad de investigaciones de niños desaparecidos (Período 1975-1978)",
          "type": "Institutional Project (Fundación Sadosky)",
          "status": "completed",
          "description": "Diseño y desarrollo del sistema de seguimiento de datos críticos para la reconstrucción de la memoria histórica.",
          "impact_analysis": "Permitió unificar criterios de búsqueda y entrecruzamiento de datos analíticos estructurados."
        }
      ]
    }
  }
}
```

## 3. Especificación de Estados Críticos

### 3.1 Estudios Parciales (`status = "partial"`)

El sistema prohíbe penalizar trayectorias académicas inconclusas. Cuando el estado de un estudio es parcial, la IA procesará la entidad basándose en:

- `completed_credits_percentage`: Determina el peso del trasfondo teórico duro adquirido (por ejemplo, tener el 85% de Ciencias de la Computación indica un trasfondo algorítmico masivo).

- `key_learnings`: Lista explícita de núcleos duros del conocimiento.

- `reason_for_status`: Proporciona la narrativa profesional/personal que justifica la transición al mercado laboral de forma constructiva.

### 3.2 Publicaciones en Desarrollo (`status = "draft"`)

Permite documentar el conocimiento de vanguardia y la investigación en curso antes de pasar por comités de referato o publicación tradicional. La IA del agente puede indexar los vectores del abstract para validar el interés técnico del candidato.

### 3.3 Investigaciones de Campo (`field_research_and_investigations`)

Diseñado para registrar contribuciones de alto valor metodológico, social o histórico que no encajan en la definición corporativa tradicional (como auditorías de datos históricos, desarrollo ad-honorem de herramientas analíticas para fundaciones, etc.).