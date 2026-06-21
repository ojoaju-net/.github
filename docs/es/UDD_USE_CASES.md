# Casos de Uso Dirigidos por el Usuario (UDD)

Este documento detalla las historias de usuario y los flujos de interacción clave que guían el desarrollo del ecosistema **Ojoaju**. Cada caso de uso define la experiencia del usuario (UX) en el nivel superior y su correspondiente correlato técnico en el nivel inferior.

```
graph TD
    UC1[UC-1: Inicialización de Identidad] --> UC7[UC-7: Despliegue y Vinculación Segura]
    UC7 --> UC2[UC-2: Ingesta de Datos]
    UC2 --> UC3[UC-3: Sincronización Cloud]
    UC3 --> UC4[UC-4: Búsqueda Anónima]
    UC4 --> UC5[UC-5: Revelación de Identidad]
    UC2 --> UC6[UC-6: Generación de CV Adaptativo]
```

## UC-1: Inicialización de Identidad Criptográfica

### Escenario de Usuario (UX)

> "Instalo la aplicación de escritorio de Ojoaju en mi computadora por primera vez. Quiero configurar mi entorno de forma segura, sin tener que registrarme con un email o contraseña en un servidor central, y recibir mi dirección única en la red."

### Flujo de Interacción

1. El usuario ejecuta la aplicación local (`ojoaju-local`).

2. Presiona el botón "Generar Nueva Identidad Soberana".

3. La aplicación le solicita mover el mouse aleatoriamente para agregar entropía física al generador criptográfico.

4. Se le presenta su **Node ID** público (ej: `did:ojoaju:6f8a9b...`) y un archivo de respaldo cifrado con una contraseña maestra elegida por el usuario.

### Lógica Técnica de Bajo Nivel

- Generación de una semilla de entropía local para el par de claves **Ed25519**.

- Escritura del par de llaves en el almacenamiento seguro del sistema operativo (Keychain en macOS, Credential Manager en Windows o Secret Service API en Linux).

- Cálculo del hash criptográfico SHA256 de la llave pública y codificación en Base58 para generar el `Node ID` (DID).

- Exportación de la clave pública ($PK$) y registro en el archivo de configuración del cliente local.

## UC-7: Despliegue de la VM y Vinculación Segura (Secret Handshake)

### Escenario de Usuario (UX)

> "He alquilado una máquina virtual (VM) económica en la nube y quiero desplegar mi nodo de Ojoaju (`ojoaju-node`). Necesito asegurarme de que mi cliente local se conecte exclusivamente a mi VM y que ningún atacante que descubra mi dirección IP pueda controlar mi nodo o interceptar mis búsquedas. Para ello, realizo un emparejamiento seguro con un secreto unívoco."

### Flujo de Interacción

1. El usuario levanta la imagen Docker de `ojoaju-node` en su VM cloud.

2. Al iniciar por primera vez, la consola del contenedor Docker de la VM autogenera un token de acceso seguro de un solo uso (**Pairing Token**) y lo expone en la salida estándar de logs.

3. El usuario copia la dirección IP (o dominio) de su VM y el Pairing Token desde la consola de su proveedor cloud.

4. En su cliente de escritorio (`ojoaju-local`), va a la sección "Conectar Nodo Cloud" e introduce la IP de la VM y el Pairing Token.

5. El cliente realiza una prueba de conexión automática. Tras un instante, la pantalla se actualiza con un mensaje verde: *"VM Conectada con éxito y blindada criptográficamente"*. A partir de este momento, el Pairing Token de la VM se autodestruye y la VM ya no acepta más intentos de emparejamiento.

### Lógica Técnica de Bajo Nivel

- **Inicialización Huérfana:** Al arrancar en frío, la VM (`ojoaju-node`) detecta que no tiene ninguna llave pública autorizada asociada. El sistema genera un secreto criptográfico aleatorio fuerte de 32 bytes (codificado en Base64/Hex) y lo almacena temporalmente en su base de datos local en memoria, imprimiéndolo en los logs del sistema.

- **Protocolo de Reclamación de Nodo:** El cliente local envía una petición `POST /api/v1/auth/pair` a la VM con el siguiente payload:
  
  - El `Pairing Token` original.
  
  - El `Node ID` ($PK$) del usuario.
  
  - Una firma Ed25519 de un timestamp actual firmada con la clave privada local ($SK$).

- **Blindaje del Nodo:** La VM valida que el Pairing Token coincida y que la firma sea matemáticamente correcta. Tras el éxito, la VM:
  
  1. Almacena permanentemente el $PK$ del usuario en su archivo de configuración interno como la **Única Firma Autorizada (Owner PK)**.
  
  2. Borra de forma definitiva el Pairing Token de su memoria.
  
  3. Rechaza cualquier conexión futura a los endpoints `/sync` u `/identity` que no vengan acompañadas de un encabezado de autorización firmado por la clave privada del Owner.

## UC-2: Ingesta de Conocimiento y Generación del Grafo

### Escenario de Usuario (UX)

> "Quiero que la plataforma consolide mis habilidades reales. Le indico a Ojoaju dónde tengo mis repositorios de código locales, mi perfil de GitHub y algunos borradores de investigaciones que escribí en Markdown. Quiero que el sistema procese esto de forma local y me muestre qué capacidades fácticas detectó, sin revelar código sensible."

### Flujo de Interacción

1. El usuario entra a la sección "Fuentes de Conocimiento" en su cliente local.

2. Agrega una carpeta local con sus proyectos de desarrollo y vincula su cuenta de GitHub.

3. Presiona "Analizar Fuentes".

4. Una barra de progreso muestra la extracción. Al finalizar, el usuario ve una interfaz interactiva con tarjetas que representan:
   
   - **Capacidades técnicas demostradas** (ej: *"Resolución de race-conditions en hilos de Python"*).
   
   - **Proyectos estructurados** con sus desafíos resueltos.
   
   - **Historial académico y de investigación** (incluso estudios incompletos con su justificación).

5. El usuario puede editar los textos generados por la IA local, eliminar capacidades que no desea exponer o agregar notas manuales antes de guardar.

### Lógica Técnica de Bajo Nivel

- **Scraping local:** El cliente escanea el sistema de archivos local, parsea archivos `.git`, archivos de configuración (como `package.json`, `requirements.txt`) y documentos de texto.

- **Procesamiento de LLM Local:** Llamada a un modelo LLM local (ej. a través de Llama.cpp u Ollama) para resumir y extraer desafíos resueltos y capacidades técnicas a partir de la evidencia fáctica.

- **Construcción del Grafo:** Generación del archivo JSON que cumple estrictamente con el esquema definido en `KNOWLEDGE_GRAPH.md`.

## UC-3: Sincronización y Publicación en la VM Cloud

### Escenario de Usuario (UX)

> "Una vez que verifiqué mi grafo de conocimiento localmente, quiero subirlo a mi nodo en la nube para que esté disponible las 24 horas para búsquedas de reclutadores, con la seguridad de que nadie en el camino pueda alterar mis datos."

### Flujo de Interacción

1. El usuario presiona el botón "Sincronizar con la Nube".

2. La aplicación muestra un indicador de transferencia rápida.

3. Una vez finalizado, el panel muestra el estado: **Sincronizado - Versión V3 (Activa)**.

4. El usuario puede ver el historial de cambios anteriores y, si lo desea, presionar "Restaurar versión anterior" para volver a un estado previo del perfil.

### Lógica Técnica de Bajo Nivel

- **Generación de Delta:** Comparación del JSON actual contra la última versión guardada localmente para generar un array de operaciones JSON Patch (RFC 6902).

- **Firma Criptográfica:** Cálculo del hash del commit y firma con la clave privada Ed25519 del usuario.

- **Transmisión:** Petición HTTPS POST al endpoint `/api/v1/sync/commit` de la VM en la nube (especificado en `SYNC_PROTOCOL.md`).

- **Actualización del Nodo:** La VM valida la firma, impacta el parche y actualiza atómicamente el índice vectorial del agente.

## UC-4: Consulta Semántica Anónima (Reclutador)

### Escenario de Usuario (UX)

> "Como líder técnico o reclutador, necesito encontrar un especialista en sistemas distribuidos que haya solucionado problemas reales de concurrencia. Busco en la red y el sistema me devuelve una lista de agentes que cumplen con el perfil de forma anónima, explicándome exactamente la evidencia técnica de cada uno sin revelar nombres ni datos de contacto."

### Flujo de Interacción

1. El reclutador ingresa en su propio nodo o portal de búsqueda y escribe: *"Ingeniero que haya migrado un monolito a microservicios controlando pérdidas de datos"*.

2. El sistema distribuye la consulta semántica.

3. El agente de IA de la VM de nuestro usuario recibe la consulta, analiza internamente su Grafo de Conocimiento y responde de forma automatizada: *"Tengo un match del 94%. El usuario diseñó una arquitectura basada en eventos que mitigó cuellos de botella en producción, utilizando RabbitMQ y Python"*.

4. El reclutador ve el perfil anónimo `did:ojoaju:6f8a9b...` con el detalle fáctico del proyecto, pero con los campos de contacto ocultos.

### Lógica Técnica de Bajo Nivel

- Recepción de la consulta vectorial en el endpoint `/api/v1/agent/match`.

- Cálculo de similitud de coseno entre el embedding de la consulta del reclutador y los vectores locales indexados del usuario.

- Procesamiento de contexto por la IA del nodo para redactar un informe de compatibilidad sin extraer datos de los bloques protegidos de `identity`.

## UC-5: Solicitud de Revelación de Identidad y Aprobación (Handshake)

### Escenario de Usuario (UX)

> "Un reclutador vio el informe anónimo de mi agente de IA sobre mi experiencia en concurrencia y me envió una oferta de entrevista técnica concreta. Quiero evaluar la oferta y, si me interesa, revelarle mis datos de contacto de manera segura."

### Flujo de Interacción

1. El reclutador presiona "Solicitar Datos de Contacto" en su panel, adjuntando la descripción del puesto y la banda salarial ofrecida.

2. El cliente local del usuario recibe una notificación: *"Oferta recibida para did:ojoaju:6f8a9b... - Rol: Senior backend - Rango: $7000 USD"*.

3. El usuario revisa la propuesta. Si le interesa, presiona "Aceptar e Intercambiar Datos".

4. Al instante, el reclutador recibe el nombre real del candidato, su email y su portfolio, mientras que el candidato obtiene las vías de contacto del reclutador de forma cifrada.

### Lógica Técnica de Bajo Nivel

- Petición del reclutador al endpoint `/api/v1/agent/reveal-request` enviando su llave pública $PK_{\text{reclutador}}$.

- Almacenamiento temporal de la petición en el nodo en estado pendiente.

- El cliente local descarga la petición en la siguiente sincronización y la descifra/presenta al usuario.

- Si se aprueba, se genera un canal seguro utilizando un intercambio de claves Diffie-Hellman (ECDH) para cifrar los campos del bloque `identity` en origen y enviarlos cifrados al nodo del reclutador.

## UC-6: Generación de CV Adaptativo para Canales Tradicionales

### Escenario de Usuario (UX)

> "Tengo que postularme a una empresa tradicional que no está en la red de Ojoaju y me piden un CV clásico en formato PDF. Quiero que la IA de mi cliente local redacte un CV optimizado para esa oferta utilizando únicamente mis datos reales verídicos de mi Grafo de Conocimiento."

### Flujo de Interacción

1. El usuario va a la sección "Exportar CV Tradicional" en el cliente local.

2. Copia y pega el texto de la oferta de trabajo tradicional a la que quiere aplicar.

3. Selecciona la plantilla de diseño deseada.

4. Presiona "Generar CV Optimizado".

5. La aplicación local analiza qué proyectos, publicaciones e investigaciones de su Grafo de Conocimiento son más relevantes para esa oferta en particular y genera un archivo PDF y un archivo Markdown redactados de forma impecable y veraz.

### Lógica Técnica de Bajo Nivel

- Envío del texto de la oferta laboral y de la estructura completa del Grafo de Conocimiento (`KNOWLEDGE_GRAPH.md`) a un modelo LLM local o API externa del usuario.

- Ejecución de una plantilla semántica que reorganiza y jerarquiza los elementos del JSON.

- Generación del documento final en formato estructurado de Markdown o PDF para su descarga inmediata.
