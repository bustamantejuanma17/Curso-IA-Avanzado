# Pre-entrega Ejercicio 4: Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

**Autor:** Juan Manuel Bustamante  
**Módulo:** Integraciones Avanzadas e Interconexión de Sistemas  
**Ecosistema:** Automación de Soporte e-Commerce (Gmail + HubSpot CRM + Slack + Airtable)

---

## 📌 Descripción General del Sistema

Este proyecto evoluciona la arquitectura agéntica con memoria persistente del Módulo 3, conectándola bidireccionalmente con herramientas corporativas externas reales mediante protocolos OAuth2 y llaves de servicio privadas. 

El sistema captura consultas entrantes por correo electrónico, realiza clasificación y redacción mediante agentes multi-nivel (Workers), gestiona el contexto histórico en Airtable y sincroniza automáticamente los contactos en HubSpot CRM y las alertas operativas en Slack.

---

## 🔑 Credenciales Activas en n8n

A continuación se adjunta la captura de pantalla de la sección **Credentials** en n8n con las tres conexiones externas (Gmail, HubSpot y Slack) autenticadas y en estado activo (semáforo en verde):

<img src="https://github.com/user-attachments/assets/1fb0ca44-b71a-4ea9-a950-01a7aecd841e" alt="Credenciales Activas n8n" width="100%" />

---

## 🛡️ Implementación de los 4 Guardrails de Seguridad (Rúbrica)

Para garantizar estabilidad en entornos productivos, se implementaron 4 barreras de contención no-code en el lienzo principal:

1. **Anti Auto-Reply (Resiliencia contra Bucles Infinitos):**  
   * **Nodo:** `If: ¿Es auto-reply?` $\rightarrow$ `Stop: Auto-reply ignorado`
   * **Función:** Escanea el payload del `Gmail Trigger` y desvía al nodo de parada instantánea todo correo entrante con asuntos o remitentes que contengan etiquetas como `Auto-reply`, `Out of office`, `Undeliverable`, `no-reply@` o `noreply@`.

2. **CRM Look up / Búsqueda Previa (Anti-Duplicados Error 409):**  
   * **Nodo:** `Search contacts` $\rightarrow$ `If` (HubSpot)
   * **Función:** Ejecuta una compuerta de verificación por email antes de disparar la creación de contactos en HubSpot. Si el cliente ya existe, encausa el flujo directamente sin duplicar la entidad en el CRM, evitando errores 409.

3. **Human-in-the-Loop / HITL (Gobernanza de Emisión):**  
   * **Nodo:** `Create a draft` (Gmail)
   * **Función:** Las respuestas generadas por los agentes de IA no se envían de forma directa al cliente final, sino que se depositan como **Borradores** en la casilla de Gmail para aprobación y supervisión humana previa.

4. **Limpieza y Sanitización de Payload (Prevención de Error 400):**  
   * **Nodo:** `SET: Limpiar Payload` y `SET: Limpieza Payload Slack`
   * **Función:** Sanitiza textos mediante expresiones regulares, remueve etiquetas HTML, acota entradas largas (`chatInput` $\le$ 2000 caracteres) y estructura el mensaje enviado a Slack (`#operaciones`) para evitar saturation de scopes o payloads pesados.

---

## 📐 Estructura de Workflows Incluidos

* **`checkpoint4_Juanma_Bustamante.json`**: Workflow orquestador principal (Trigger, Guardrails, Clasificador, Memoria Airtable, CRM, Borradores y Alertas Slack).
* **`Worker 1 - Extractor.json`**: Sub-workflow especializado en extracción e integración con planillas de datos.
* **`Worker 2 - Redactor.json`**: Sub-workflow especializado en la redacción de respuestas institucionales.
