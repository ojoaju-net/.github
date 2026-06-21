# Protocolo de Sincronización y Re-indexación Vectorial

## 1. Objetivo del Protocolo

El **Protocolo de Sincronización de Ojoaju** define la interfaz de interacción de red entre el Cliente Local (`ojoaju-local`) y la VM del Nodo Cloud (`ojoaju-node`).

Sus objetivos principales son:

1. **Eficiencia:** Transferir únicamente los cambios granulares (JSON Patches) en lugar de capturas completas del estado.

2. **Seguridad:** Garantizar que cada mutación de estado esté matemáticamente autorizada por el usuario antes de su ejecución.

3. **Inteligencia Reactiva:** Desencadenar actualizaciones atómicas en la base de datos vectorial del Agente de IA sin requerir una re-indexación completa del documento.

## 2. Arquitectura de Sincronización

El proceso de sincronización sigue un flujo unidireccional de empuje (push) estrictamente controlado desde el cliente hacia el nodo cloud:

```
sequenceDiagram
    autonumber
    participant Client as ojoaju-local
    participant VM as ojoaju-node
    participant DB as Commit Log DB
    participant VectorDB as AI Vector Index

    Client->>Client: 1. Genera JSON Patch (Δ) y Manifiesto de Commit Local
    Client->>Client: 2. Firma version_id con Llave Privada Ed25519 (SK)
    Client->>VM: 3. POST /api/v1/sync/commit (Payload)
    Note over VM: Verificación Criptográfica<br/>Ed25519_Verify(PK, version_id, signature)
    alt Verificación Falla
        VM-->>Client: 401 Unauthorized (Firma Inválida)
    else Verificación Exitosa
        VM->>DB: 4. Anexa Manifiesto de Commit al Log Lineal
        VM->>VM: 5. Aplica JSON Patch al Documento de Estado Actual
        VM->>VectorDB: 6. Dispara Actualización Vectorial Granular (Ingesta Delta)
        VM-->>Client: 200 OK (Sincronización Exitosa, Nueva Versión Aceptada)
    end
```

## 3. Especificación del Contrato de la API

### 3.1 Endpoint de Sincronización de Commits

- **Endpoint:** `POST /api/v1/sync/commit`

- **Content-Type:** `application/json`

#### Esquema del Payload de la Petición:

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
        "capability": "Ingeniería de sistemas distribuidos utilizando Python y FastAPI.",
        "evidence_source": "repo:ojoaju-node",
        "ai_analysis": "Implementación de código verificada para el middleware de verificación criptográfica."
      }
    }
  ],
  "signature": "a4f8e2d1..."
}
```

#### Respuestas Esperadas:

- **`200 OK`**: Commit validado, anexado y aplicado correctamente.
  
  ```
  {
    "status": "synchronized",
    "current_version": "8f3c6a9d...",
    "vector_index_updated": true
  }
  ```

- **`400 Bad Request`**: Error por falta de sincronización (el `parent_id` proporcionado no coincide con el `version_id` actual del nodo). Requiere que el cliente realice un *fetch* previo para actualizar su estado antes de reintentar.

- **`401 Unauthorized`**: Falló la verificación de la firma criptográfica.

## 4. Estrategia de Re-indexación Vectorial por Deltas

Para minimizar los costos de cómputo y el tiempo de ejecución en la VM cloud, el **Agente de IA de Ojoaju** nunca realiza una vectorización completa del grafo al recibir un commit. En su lugar, lee el array de operaciones de JSON Patch y mapea las rutas directamente a particiones de vectores específicas.

### Mapeo de Patches a Almacenamiento Vectorial:

1. **Aislamiento de Rutas:** El motor de indexación parsea el atributo `path` de las operaciones del JSON Patch (ej. `/knowledge_graph/verified_capabilities/2`).

2. **Operaciones Granulares:**
   
   - **`add` / `replace`**: El sistema toma el objeto `value`, lo convierte en un vector embebido (embedding) mediante la API del LLM, y agrega o sobrescribe únicamente el vector individual correspondiente a ese índice de ruta.
   
   - **`remove`**: El sistema purga el vector del índice utilizando el identificador único derivado de la ruta eliminada.

3. **Enriquecimiento del Contexto:** Al indexar una sección granular (como un solo proyecto o capacidad), el motor inyecta automáticamente los metadatos globales de `#core_expertise` para asegurar que el embedding conserve todo su trasfondo contextual durante las búsquedas semánticas realizadas por los reclutadores.
