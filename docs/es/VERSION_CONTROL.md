# Control de Versiones del Grafo de Conocimiento

## 1. Filosofía de Versionado

Para preservar la soberanía y la integridad de la información profesional y académica de un usuario, **Ojoaju** implementa un **historial de versiones lineal, inmutable y de solo anexión (append-only)**.

En lugar de tratar al Grafo de Conocimiento (KG) como una base de datos tradicional que se sobrescribe con cada cambio, el KG se maneja como una secuencia cronológica de transiciones de estado criptográficas (commits).

### Reglas Clave del Versionado en Ojoaju:

1. **Historial Lineal:** No se permiten ramas (branches), fusiones (merges) o bifurcaciones (forks) en el nodo de perfil activo. La línea de tiempo es una lista enlazada simple donde cada versión $V_n$ apunta a su versión padre $V_{n-1}$.

2. **Inmutabilidad:** Una vez que una versión se confirma en el nodo cloud con una firma criptográfica válida, no puede ser editada, eliminada o reordenada.

3. **Recuperación por Roll-Forward:** Revertir a un estado pasado no se logra borrando el presente. En su lugar, se anexa un nuevo commit hacia adelante cuyo efecto neto resulta en el estado histórico deseado.

## 2. Esquema del Manifiesto de Commit

Cada transición de estado se empaqueta en un **Manifiesto de Commit**. Este manifiesto contiene el delta de cambio (patch), metadatos y la prueba criptográfica de autorización.

```
{
  "version_id": "sha256_hash_del_cuerpo_del_commit",
  "parent_id": "sha256_hash_del_commit_padre | null",
  "timestamp": "2026-06-20T19:30:00Z",
  "author_node_id": "did:ojoaju:6f8a9b123456789abcdef",
  "patch": [
    { "op": "replace", "path": "/knowledge_graph/projects/0/role", "value": "Principal Architect" }
  ],
  "signature": "ed25519_signature_hex"
}
```

### Verificación del Hash

El `version_id` se calcula mediante el hash de la representación JSON canónica del cuerpo del commit (excluyendo el campo `signature`):

$$\text{version\_id} = \text{SHA256}(\text{CanonicalJSON}(\text{parent\_id} + \text{timestamp} + \text{author\_node\_id} + \text{patch}))$$

La firma (`signature`) es generada por el cliente local utilizando la Llave Privada ($SK$) del usuario:

$$\text{signature} = \text{Ed25519\_Sign}(SK, \text{version\_id})$$

## 3. Algoritmo de Reconstrucción de Estados

Para mostrar el estado actual o cualquier estado pasado del perfil, el nodo procesa la cadena de commits comenzando desde el Commit de Génesis ($V_0$, donde el grafo está vacío `{}`) hasta llegar a la versión objetivo $V_{\text{target}}$.

$$\text{Estado}(V_{\text{target}}) = \text{AplicarPatches}(\{\}, V_0 \to V_1 \to \dots \to V_{\text{target}})$$

```
graph TD
    G[Génesis:  { } ] -->|Aplicar V1 Patch| V1[Estado V1]
    V1 -->|Aplicar V2 Patch| V2[Estado V2 (Actual)]
```

Al almacenar solo los parches (deltas) en lugar de capturas completas del grafo, el almacenamiento en la VM cloud se optimiza enormemente y los cambios son fácilmente auditables.

## 4. Recuperación por Roll-Forward (Reversión de Cambios)

Si un usuario desea restaurar un estado de perfil antiguo (por ejemplo, recuperar datos eliminados por error), el protocolo prohíbe reescribir la historia.

Para revertir el estado actual $V_{\text{actual}}$ de regreso a un estado histórico objetivo $V_{\text{target}}$:

1. El cliente local reconstruye el estado JSON tanto de $V_{\text{actual}}$ como de $V_{\text{target}}$.

2. Calcula un **diff inverso** entre ambos estados utilizando el estándar JSON Patch.

3. Empaqueta este diff inverso en un nuevo commit $V_{\text{actual}+1}$.

4. Al aplicarse, $V_{\text{actual}+1}$ devuelve el grafo activo a la estructura exacta de datos de $V_{\text{target}}$, manteniendo intacto el registro cronológico.

$$\text{Patch}(V_{\text{actual}+1}) = \text{CalcularJSONDiff}(\text{Estado}(V_{\text{actual}}), \text{Estado}(V_{\text{target}}))$$

### Diagrama de Roll-Forward hacia V1 desde V2:

```
graph LR
    V1[Estado V1] -->|Delta V2 Aplicado| V2[Estado V2]
    V2 -->|Patch Inverso V2 a V1 Aplicado| V3[Estado V3 (Clon de V1)]
```

Este mecanismo garantiza que la cadena de custodia criptográfica nunca se rompa y que los índices de la base de datos vectorial del agente de IA puedan actualizarse de manera incremental sin necesidad de reconstrucciones completas.
