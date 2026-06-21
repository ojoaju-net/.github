# Identidad Criptográfica y Firmas

## 1. Filosofía de Identidad Soberana

En **Ojoaju**, la identidad es de control absoluto de la persona. No se almacena una base de datos centralizada de usuarios ni se depende de terceros para autenticar el perfil (como Google, GitHub o contraseñas tradicionales).

El usuario genera su identidad en su máquina local (*Local-First*), asegurando que:

1. Ningún intermediario ni administrador de la red puede suplantar al candidato.

2. La VM en la nube (el nodo) actúa solo como un proxy de ejecución disponible 24/7, pero no posee las llaves maestras para alterar el historial sin permiso.

## 2. El Par de Llaves del Usuario

La identidad de Ojoaju se basa en la curva elíptica **Ed25519** por su óptima eficiencia de cómputo, seguridad y firmas sumamente compactas.

- **Llave Privada (**$SK$ **- Secret Key):**
  
  - **Ubicación:** Almacenada exclusivamente en el cliente local del usuario (`ojoaju-local`) de forma cifrada. Nunca viaja por la red, no se sube a la VM cloud y ningún agente la conoce.
  
  - **Función:** Firmar digitalmente los estados (Deltas/Patches) del Grafo de Conocimiento (KG) y autorizar el descifrado y liberación de datos sensibles.

- **Llave Pública (**$PK$ **- Public Key):**
  
  - **Ubicación:** Registrada en la VM cloud y expuesta a la red pública.
  
  - **Función:** Actúa como el identificador único del usuario y permite a cualquier nodo de reclutamiento verificar la autenticidad matemática de los datos compartidos.

### 2.1 Identificador del Nodo (Node ID / DID)

Para mantener compatibilidad con esquemas de descentralización globales de la W3C, la dirección pública del nodo del usuario sigue la sintaxis de Identificadores Descentralizados (DIDs):

$$\text{Node ID} = \text{did:ojoaju:}\langle \text{Base58Encoding}(\text{SHA256}(PK)) \rangle$$

Este string es el pseudónimo con el cual el Agente de IA se anuncia en la red para recibir consultas sin revelar datos civiles.

## 3. Protocolo de Sincronización Local -> Cloud

Para que la VM cloud no pueda ser comprometida externamente y alterar el CV de un usuario sin su consentimiento, cada actualización (parche) debe ser firmada en el cliente local:

```
[Cliente Local (ojoaju-local)]                   [VM Cloud (ojoaju-node)]
              |                                             |
              |-- 1. Genera JSON Patch (Δ)                  |
              |-- 2. Calcula Hash = SHA256(Δ + Timestamp)   |
              |-- 3. Firma = Ed25519_Sign(SK, Hash)         |
              |                                             |
              |-------- 4. POST /api/v1/agent/sync -------->|
              |            Payload: { patch: Δ, signature: F} |  -- 5. Obtiene PK del usuario
              |                                             |  -- 6. Verifica Ed25519_Verify(PK, Hash, F)
              |                                             |  -- 7. Si es True: Aplica Patch e indexa IA
              |<------- 8. 200 OK (Sincronizado) -----------|
```

Si la VM recibe un parche con firma inválida, rechaza la operación de inmediato, bloqueando cualquier intento de alteración externa o secuestro del nodo.

## 4. El Velo de Anonimato y Apretón de Manos (Handshake)

Cuando un reclutador realiza un match semántico y desea conocer al profesional humano detrás del agente de IA, se inicia el protocolo de descifrado y liberación delegada:

1. **Solicitud de Revelación:** El reclutador envía su propia llave pública de cifrado ($PK_{\text{reclutador}}$) al nodo del usuario a través del endpoint `/reveal-request`.

2. **Notificación al Cliente Local:** El nodo cloud congela la petición y la envía de vuelta al cliente local (`ojoaju-local`) en la computadora del usuario la próxima vez que este se conecte.

3. **Aprobación del Humano:** Si el profesional decide aceptar la propuesta e interactuar con la empresa, el cliente local toma los campos sensibles del bloque `identity` (`full_name`, `contact_email`, etc.) y los cifra simétricamente utilizando una llave efímera compartida derivada de un intercambio Diffie-Hellman entre $SK_{\text{usuario}}$ y $PK_{\text{reclutador}}$.

4. **Liberación:** Los datos cifrados se envían al reclutador a través del nodo. Solo la llave privada del reclutador ($SK_{\text{reclutador}}$) puede descifrar el bloque de datos personales de identidad del candidato. El resto de la red solo sigue observando un ID anónimo de nodo.