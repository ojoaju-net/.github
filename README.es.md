# **Ojoaju Network: El protocolo de conocimiento soberano representado por agentes**

**Ojoaju** (del guaraní *enlace*, *conexión* o *vínculo*) es un protocolo descentralizado y *Local-First* diseñado para la preservación, verificación y emparejamiento semántico de capacidades técnicas y académicas reales, eliminando por completo el ruido comercial, la propaganda y la asimetría de información del mercado laboral actual.

## **El Problema con el Status Quo**

Las plataformas de empleo y redes profesionales centralizadas (como LinkedIn) han transformado la validación de talento en una red social de captación de *engagement*. El algoritmo premia la publicación constante de contenido de marketing personal ("ruido") por sobre la experiencia técnica o académica verídica ("señorío").

Esto obliga a los profesionales a inflar sus currículums y optimizarlos artificialmente para superar sistemas de filtrado de texto estático (ATS), destruyendo la eficiencia y la transparencia en la contratación global.

## **La Propuesta de Ojoaju**

Ojoaju rompe con este paradigma devolviendo el control absoluto de los datos al profesional mediante una arquitectura distribuida de tres capas:

1. **Soberanía Local-First (Cliente Local):** El usuario consolida su conocimiento real (código de repositorios locales, GitHub, papers científicos, proyectos truncados, investigaciones independientes o informes) de forma local en su máquina.  
2. **Nodos Cloud Autónomos (VM Personal):** Una máquina virtual mínima y de bajo costo ejecuta una imagen Docker ligera de Ojoaju las 24 horas del día. Sincroniza datos limpios y estructurados desde el cliente local mediante parches criptográficos.  
3. **Representación por Agentes de IA:** Cada nodo cloud expone un Agente de IA entrenado con la base de conocimiento local del usuario. Este agente evalúa las solicitudes de reclutamiento de forma semántica y anónima. La identidad del humano solo se revela bajo su aprobación explícita cuando hay una compatibilidad real.

## **Estructura de la Organización**

El ecosistema de Ojoaju se divide en los siguientes repositorios modulares:

* **.github / ojoaju-docs:** Repositorio central con la especificación técnica global del protocolo y estándares.  
* **ojoaju-local:** El cliente de escritorio / CLI. Se encarga de la ingestión, limpieza de datos, generación de claves criptográficas y firmado de parches en la máquina local del usuario.  
* **ojoaju-node:** La imagen Docker para la VM en la nube. Contiene la base de datos indexada, el motor de búsqueda vectorial y el Agente de IA para responder consultas de la red.  
* **ojoaju-landing:** La landing page pública del proyecto y portal de documentación para usuarios y reclutadores.

## **Documentación Técnica Inicial**

Podés profundizar en las especificaciones del diseño de bajo nivel en los siguientes enlaces del repositorio central:

### **Inglés (Por defecto)**

* [Knowledge Graph (KG) Specification](docs/en/KNOWLEDGE_GRAPH.md)  
* [Cryptographic Identity and Signing](docs/en/CRYPTO_IDENTITY.md)

### **Español**

* [Especificación del Grafo de Conocimiento (KG)](docs/es/KNOWLEDGE_GRAPH.md)  
* [Identidad Criptográfica y Firmas en Origen](docs/es/CRYPTO_IDENTITY.md)