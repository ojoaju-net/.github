# Backlog de Desarrollo Dirigido por IA: Ecosistema Ojoaju

Este documento contiene el plan de trabajo (backlog) estructurado en tareas atómicas y secuenciales para que un asistente de desarrollo de IA (**AI Developer Agent**) implemente el ecosistema **Ojoaju** paso a paso.

Cada tarea está diseñada bajo el principio de caja negra: tiene requisitos claros, entradas, salidas esperadas y criterios de aceptación técnicos basados en las especificaciones del Canvas.

```
graph TD
    T1[Tarea 1: Criptografía y UC-1] --> T2[Tarea 2: Emparejamiento UC-7]
    T2 --> T3[Tarea 3: Ingesta y Grafo UC-2]
    T3 --> T4[Tarea 4: Endpoint Sync UC-3]
    T4 --> T5[Tarea 5: Motor Vectorial UC-4]
    T5 --> T6[Tarea 6: Handshake Revelación UC-5]
    T6 --> T7[Tarea 7: CV Adaptativo UC-6]
```

## Fase 1: Identidad Criptográfica y Emparejamiento Seguro (UC-1 y UC-7)

### Tarea 1.1: Inicialización de Identidad en `ojoaju-local` (Python)

- **Contexto:** El cliente local de escritorio debe poder autogenerar su identidad criptográfica soberana en frío sin depender de ningún servidor de registro tradicional.

- **Instrucciones para la IA:**
  
  - Crea un script de Python llamado `crypto_identity.py`.
  
  - Utiliza la librería `cryptography` (o `nacl`) para generar un par de claves **Ed25519**.
  
  - Computa el **Node ID** (DID) de la siguiente forma:
    
    1. Toma la llave pública Ed25519 en bytes.
    
    2. Aplica un hash SHA-256.
    
    3. Codifica el resultado utilizando Base58. El DID final debe tener el formato `did:ojoaju:<Base58Hash>`.
  
  - Diseña una función para almacenar la clave privada cifrada localmente con una contraseña maestra definida por el usuario usando AES-256-GCM.

- **Entrada:** Interacción inicial del usuario (o contraseña maestra suministrada).

- **Salida:** Archivo de configuración `identity.json` conteniendo: `{"did": "did:ojoaju:...", "public_key_hex": "..."}` y el contenedor cifrado de la clave privada.

- **Criterios de Aceptación:**
  
  - La llave generada debe ser matemáticamente válida.
  
  - No se guarda la clave privada en texto plano bajo ninguna circunstancia.

### Tarea 1.2: Inicialización Huérfana y Endpoint de Emparejamiento en `ojoaju-node`

- **Contexto:** Al desplegar la VM por primera vez, esta no tiene dueño. Debe generar un token temporal que permita al cliente local vincularse de forma segura (UC-7).

- **Instrucciones para la IA:**
  
  - Desarrolla en FastAPI (`main.py` de `ojoaju-node`) un estado inicial "huérfano".
  
  - Al iniciar la aplicación, si no detecta una clave pública del propietario (`OWNER_PUBLIC_KEY`), debe generar un token criptográfico fuerte de 32 bytes (codificado en Hex) llamado `Pairing Token` y escribirlo en la salida de consola (logs).
  
  - Implementa el endpoint `POST /api/v1/auth/pair`:
    
    - Recibe: `pairing_token`, `node_id` (DID del cliente), `timestamp` y `signature`.
    
    - Valida:
      
      1. Que el `pairing_token` coincida con el generado en memoria.
      
      2. Que el `timestamp` sea reciente (ventana máxima de 5 minutos para evitar ataques de replay).
      
      3. Que la `signature` (firma Ed25519 del timestamp) sea válida usando la clave pública derivada del `node_id`.
  
  - Tras una validación exitosa: guarda permanentemente la clave pública del cliente como `OWNER_PUBLIC_KEY`, elimina el `pairing_token` de la memoria y pasa el estado del nodo a "Blindado".

- **Criterios de Aceptación:**
  
  - Cualquier llamada a `/api/v1/auth/pair` después de que el nodo ya esté blindado debe retornar un error `403 Forbidden`.
  
  - Si el token de emparejamiento no coincide, debe retornar `401 Unauthorized`.

## Fase 2: Ingesta de Conocimiento y Grafo de Datos (UC-2 y UC-3)

### Tarea 2.1: Analizador de Código y Constructor de Grafo en `ojoaju-local`

- **Contexto:** El usuario desea escanear su trabajo local para estructurar sus habilidades reales en el formato JSON formal (`KNOWLEDGE_GRAPH.md`).

- **Instrucciones para la IA:**
  
  - Crea un módulo `ingestion/scanner.py`.
  
  - Debe escanear de forma recursiva una carpeta local buscando:
    
    - Archivos de configuración (`package.json`, `requirements.txt`, `Cargo.toml`) para listar dependencias y tecnologías usadas.
    
    - Archivos Markdown (`.md`) para extraer resúmenes de investigaciones o notas académicas.
    
    - Historial de Git (lectura de los últimos 20 commits y el diff de cambios más grande).
  
  - Prepara el prompt para enviar esta información consolidada a la API de Gemini (o LLM local) para extraer: *Habilidades demostradas, Desafíos resueltos y Resultados cuantificables*.
  
  - Estructura los resultados de retorno de la IA de acuerdo al esquema oficial de JSON definido en la especificación del Grafo de Conocimiento.

- **Criterios de Aceptación:**
  
  - El script debe ignorar archivos listados en `.gitignore` (para evitar procesar carpetas pesadas como `node_modules` o `.venv`).
  
  - El JSON resultante debe pasar una validación estricta de esquema (utiliza `jsonschema` en Python).

### Tarea 2.2: Endpoint de Sincronización con Control de Versiones en `ojoaju-node`

- **Contexto:** El cliente local necesita subir sus actualizaciones al nodo cloud de forma segura mediante deltas firmados (UC-3 y `VERSION_CONTROL.md`).

- **Instrucciones para la IA:**
  
  - Crea el endpoint `POST /api/v1/sync/commit` en `ojoaju-node`.
  
  - El endpoint debe recibir:
    
    - `patch`: Un array de operaciones JSON Patch (RFC 6902).
    
    - `base_version`: El hash del commit sobre el cual se aplica el parche.
    
    - `commit_hash`: El SHA-256 autocalculado del nuevo estado del grafo.
    
    - `signature`: La firma Ed25519 de `commit_hash` realizada por la clave privada del cliente.
  
  - El middleware del nodo debe validar que la firma sea correcta utilizando la `OWNER_PUBLIC_KEY` guardada durante el emparejamiento.
  
  - Si la firma es válida, el nodo aplica el parche al archivo `knowledge_graph.json` actual, recalcula el hash para asegurar que coincide con `commit_hash` y lo guarda en el disco como el nuevo estado activo.

- **Criterios de Aceptación:**
  
  - Retornar `401 Unauthorized` si la firma es inválida.
  
  - Retornar `409 Conflict` si la `base_version` no coincide con la versión actual en la VM (conflicto de sincronización multi-dispositivo), requiriendo un pull previo.

## Fase 3: Búsqueda Semántica y Agente de IA Soberano (UC-4)

### Tarea 3.1: Indexación Vectorial en `ojoaju-node`

- **Contexto:** Cada vez que el grafo de conocimiento se actualiza, el nodo debe re-indexar semánticamente los datos en una base de datos vectorial local y ligera para habilitar búsquedas eficientes.

- **Instrucciones para la IA:**
  
  - Integra **LanceDB** o **ChromaDB** embebido como la base de datos vectorial del nodo.
  
  - Crea un script de fondo `indexing/vector_store.py` que:
    
    1. Lea las tarjetas de "Capacidades demostradas", "Proyectos" e "Investigaciones" del `knowledge_graph.json`.
    
    2. Convierta cada bloque de texto en un vector utilizando la API de embeddings de Google (`text-embedding-004`).
    
    3. Guarde los vectores en la DB local asociándolos a metadatos de referencia (ID de sección, tipo de registro, veracidad).

- **Criterios de Aceptación:**
  
  - No indexar de ninguna manera la sección `/identity` del Grafo (esta sección nunca debe tocar la base de datos de vectores).

### Tarea 3.2: Endpoint de Consulta Semántica y Orquestador de LLM

- **Contexto:** El reclutador consulta al nodo de forma anónima. El agente evalúa su compatibilidad utilizando el motor LLM bajo salvaguardas extremas de privacidad (UC-4).

- **Instrucciones para la IA:**
  
  - Implementa el endpoint `POST /api/v1/agent/match`.
  
  - Recibe: `query_text` (ej: *"Desarrollador con experiencia mitigando bloqueos mutuos en hilos"*).
  
  - Genera el embedding de `query_text` y busca los 3 vectores más cercanos en la base de datos vectorial del nodo utilizando la similitud de coseno.
  
  - Si la similitud máxima es menor a `0.75`, retorna de inmediato: `{"match": false, "message": "Compatibilidad baja con este perfil"}`.
  
  - Si la similitud es $\ge 0.75$, toma el texto de los fragmentos recuperados y arma el prompt del sistema utilizando las directrices de seguridad de `AGENTS.md`.
  
  - Llama a `gemini-2.5-flash` con temperatura `0.2` para redactar el informe de compatibilidad.
  
  - **Filtro de seguridad en la respuesta:** Antes de enviar el JSON de respuesta al reclutador, escanea el string retornado por el LLM mediante expresiones regulares (regex) y heurística para asegurar que no se haya filtrado información personal protegida (emails, números de teléfono o nombres reales).

- **Criterios de Aceptación:**
  
  - El endpoint debe responder en menos de 3 segundos bajo condiciones normales.
  
  - Cualquier mención a datos personales en el informe del LLM debe ser sanitizada automáticamente y reemplazada por `[PROTEGIDO]`.

## Fase 4: Handshake de Revelación Criptográfica (UC-5)

### Tarea 4.1: Intercambio de Claves ECDH y Revelación Segura

- **Contexto:** El usuario ha decidido aceptar la oferta de un reclutador y desea enviarle de manera cifrada e inviolable su información de contacto (nombre real, email, etc.) utilizando criptografía asimétrica.

- **Instrucciones para la IA:**
  
  - Diseña un flujo en el cliente local y en el nodo para manejar solicitudes de revelación.
  
  - El reclutador envía su llave pública efímera X25519 (ECDH) en su solicitud de revelación a través de `POST /api/v1/agent/reveal-request`.
  
  - En `ojoaju-local`, el usuario visualiza la solicitud. Al presionar "Aceptar":
    
    1. El cliente genera su propio par de llaves efímeras X25519.
    
    2. Calcula el secreto compartido utilizando Diffie-Hellman sobre curvas elípticas (ECDH).
    
    3. Deriva una clave simétrica AES-256-GCM a partir del secreto compartido mediante HKDF.
    
    4. Cifra los campos sensibles del bloque `/identity` con esta clave simétrica.
    
    5. Envía la carga cifrada y su clave pública efímera X25519 al nodo para que el reclutador la descargue.

- **Criterios de Aceptación:**
  
  - El nodo cloud actúa puramente como un repetidor (relayer) de datos cifrados; bajo ningún concepto el nodo cloud puede descifrar los datos de identidad del usuario en tránsito (cifrado de extremo a extremo, E2EE).
