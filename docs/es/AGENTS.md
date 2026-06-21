# Especificación de Agentes de IA y Parámetros de Implementación

## 1. Visión General

En **Ojoaju**, el Agente de IA es el representante activo y autónomo del Nodo del usuario. En lugar de ejecutar búsquedas mediante palabras clave tradicionales, los reclutadores consultan la red semánticamente. El agente de la VM del nodo cloud (`ojoaju-node`) procesa estas consultas contra el **Grafo de Conocimiento (KG)** local, actuando como un guardián soberano que evalúa la compatibilidad sin revelar los datos de identidad protegidos del usuario.

## 2. Parámetros Técnicos de Implementación

Para asegurar una ejecución determinista, altamente eficiente y de bajo costo en una micro-VM, el agente del nodo opera bajo las siguientes restricciones:

### 2.1 Parámetros de Ejecución del Modelo de Lenguaje (LLM)

- **Modelo Recomendado:** `gemini-2.5-flash` (vía API) o un modelo local `Llama-3-8B-Instruct` (Q4_K_M GGUF mediante llama.cpp para nodos completamente autocontenidos).

- **Temperatura (Temperature):** $\le 0.2$ (es obligatorio usar temperaturas bajas para prevenir alucinaciones y forzar la adherencia estricta a los hechos del KG).

- **Tokens Máximos de Salida (Max Output Tokens):** `1024` tokens (suficiente para resúmenes de compatibilidad).

- **Límite de Ventana de Contexto (Context Window):** `8192` tokens.

### 2.2 Parámetros de Base de Datos Vectorial y Embeddings

- **Modelo de Embedding:** `text-embedding-004` (Google Gemini) o `bge-large-es-v1.5` (alternativa local).

- **Dimensión del Vector:** `768` (Gemini) o `1024` (BGE).

- **Métrica de Distancia:** Similitud de Coseno.

- **Umbral de Match (**$T_{\text{match}}$**):** $0.75$ (las consultas que tengan un score inferior se rechazan de forma automática con una respuesta de compatibilidad baja).

$$\text{Similitud de Coseno}(A, B) = \frac{A \cdot B}{\|A\| \|B\|} \ge 0.75$$

## 3. Salvaguardas de Seguridad y Prompts del Sistema

El Agente opera como un **lector proxy no confiable (untrusted-proxy reader)**. Bajo ninguna circunstancia debe acceder o exponer campos del bloque `/identity` del archivo `KNOWLEDGE_GRAPH.md` a menos que se haya completado un handshake criptográfico ECDH autorizado (ver `CRYPTO_IDENTITY.md`).

### 3.1 Prompt del Sistema Central (La Persona del Agente)

```
INSTRUCCIÓN DEL SISTEMA:
Eres el representante de IA soberano del usuario 'did:ojoaju:<NodeID>' de la plataforma Ojoaju.
Tu objetivo es evaluar la compatibilidad de este perfil con la descripción de puesto/proyecto de un reclutador utilizando ÚNICAMENTE los hechos verificados de su Grafo de Conocimiento (KG).

REGLAS CRÍTICAS DE SEGURIDAD:
1. NUNCA reveles el nombre real del usuario, correo, teléfono, dirección física, o nombres corporativos exactos de sus empleadores actuales o pasados a menos que se te indique explícitamente mediante un token de handshake criptográfico validado.
2. Si te preguntan por información de contacto o identidad, responde estrictamente: "Información protegida por DID soberano. Por favor, inicie una solicitud de handshake."
3. NO alucines. Si una habilidad, herramienta o proyecto no está presente en el KG, declara que el candidato no cuenta con evidencia verificada para ello.
4. Concéntrate estrictamente en la evidencia fáctica del KG (ej. análisis de código, créditos académicos parciales, papers científicos, desafíos resueltos).
```

## 4. Flujo de Trabajo e Interacción del Agente

El ciclo de vida del agente durante una consulta semántica está gobernado por el siguiente flujo operativo:

```
graph TD
    Query[Consulta Semántica del Reclutador] --> Search[Endpoint: /api/v1/agent/match]
    Search --> Embed[Generar Vector de la Consulta]
    Embed --> VecSearch[Búsqueda Vectorial sobre Embeddings del KG]
    VecSearch --> Metric{¿Similitud de Coseno >= 0.75?}
    Metric -->|No| Reject[Retornar: Match de Compatibilidad Baja]
    Metric -->|Si| LLM[Armar Prompt con Contexto del KG]
    LLM --> Guard{Verificar que la salida NO tenga datos de identidad protegidos}
    Guard -->|Filtrado| Block[Sanitizar/Filtrar Salida]
    Guard -->|Seguro| Report[Retornar Informe de Compatibilidad e ID de Nodo]
```

## 5. Directorio de Documentación del Protocolo

Utiliza este índice para navegar por las especificaciones técnicas completas del ecosistema **Ojoaju**:

| Documento de Especificación     | Ruta del Archivo                                    | Alcance y Propósito                                                                                |
| ------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Visión General**              | `README.md`                                         | Filosofía central, arquitectura de alto nivel y visión de la organización.                         |
| **Grafo de Conocimiento**       | [`KNOWLEDGE_GRAPH.md`](./KNOWLEDGE_GRAPH.md "null") | Esquema JSON formal de perfiles, estudios académicos parciales e investigaciones.                  |
| **Identidad Soberana**          | [`CRYPTO_IDENTITY.md`](./CRYPTO_IDENTITY.md "null") | Criptografía asimétrica Ed25519, generación de DIDs y protocolo de handshake criptográfico.        |
| **Control de Versiones**        | [`VERSION_CONTROL.md`](./VERSION_CONTROL.md "null") | Log de commits inmutable y lineal, JSON Patches y matemática de recuperación Roll-Forward.         |
| **Protocolo de Sincronización** | [`SYNC_PROTOCOL.md`](./SYNC_PROTOCOL.md "null")     | Endpoints de la API (`POST /api/v1/sync/commit`), firmas y re-indexación vectorial por deltas.     |
| **Casos de Uso UDD**            | [`UDD_USE_CASES.md`](./UDD_USE_CASES.md "null")     | Flujos detallados de UX mapeados a lógica técnica de backend (UC-1 a UC-7).                        |
| **Especificación de Agentes**   | `AGENTS.md` *(Este Archivo)*                        | Ejecución del LLM, umbrales matemáticos de similitud, prompts de seguridad y métricas vectoriales. |
