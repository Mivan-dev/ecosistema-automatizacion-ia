# Sistema de Triage IA — Soporte Técnico

Automatización inteligente de tickets de soporte técnico usando n8n, OpenRouter (Cohere) y Notion.

---

## Caso de uso

Sistema de triage automático para una empresa de software B2B. Clasifica, prioriza y genera respuestas a tickets de soporte entrantes por múltiples canales (Email, Slack, WhatsApp), con validación humana antes de contactar al cliente final.

---

## Stack tecnológico

| Componente | Tecnología |
|---|---|
| Orquestador | n8n |
| Motor IA | Cohere North Mini Code (free) via OpenRouter |
| Base de datos | Notion (2 tablas) |
| Email | Gmail (OAuth2) |
| Trigger | Webhook HTTP POST |

---

## Arquitectura del sistema

El flujo tiene 6 capas funcionales:

1. **Entrada** — Webhook recibe el ticket vía POST
2. **Validación** — Filtro IF verifica campos obligatorios (asunto, descripción, canal)
3. **IA** — AI Agent (OpenRouter/Cohere) clasifica prioridad, categoría y genera respuesta sugerida
4. **Datos** — Resultado guardado en Notion (Tickets o Log de Errores según el camino)
5. **Human-in-the-loop** — Si requiere aprobación, el flujo se detiene y notifica al equipo por Gmail
6. **Salida** — Respuesta automática al cliente o notificación al equipo

Diagrama completo y detalle de cada nodo en [`documentacion_arquitectura.pdf`](./documentacion_arquitectura.pdf)

---

## Entregables

| Archivo | Descripción |
|---|---|
| `triage_workflow.json` | Blueprint importable en n8n con todos los nodos y configuraciones |
| `documentacion_arquitectura_v3.pdf` | Documentación completa (arquitectura, datos, costos, seguridad, dashboard) |
| `/evidencia/` | Screenshots de los 5 tests ejecutados en n8n |

**No se entrgará video Demo por problemas con OBS Studio, sin embargo en las capturas de los Test se ve el flujo y su funcionamiento, lo que mostraría el video.**
---

## Base de datos — Notion (modo lectura)

| Vista | Link |
|---|---|
| Tickets | https://open-hexagon-9c5.notion.site/3d727d4b39848083b065f62e4a505eb4 |
| Log de Errores | https://open-hexagon-9c5.notion.site/3d727d4b3984800a8daae069c45828f0 |

---

## Cómo importar el flujo en n8n

1. n8n → Workflows → **Import from file** → seleccionar `triage_workflow.json`
2. Configurar credencial **OpenRouter**: pegar API Key de openrouter.ai en el nodo OpenRouter Chat Model
3. Configurar credencial **Notion**: crear Internal Integration en notion.so/profile/integrations y conectarla a ambas bases de datos
4. Configurar credencial **Gmail OAuth2** en ambos nodos Gmail
5. Activar el workflow y copiar la URL del Webhook

---

## Tests ejecutados (5 escenarios)

| Test | Escenario | Resultado |
|---|---|---|
| T-01 | Happy path — respuesta automática | Ticket guardado en Notion + Gmail al cliente |
| T-02 | Happy path — requiere aprobación humana | Ticket guardado + email de aprobación al equipo |
| T-03 | Camino infeliz — campos faltantes | Error E-01: registro en Log + respuesta 400 |
| T-04 | Alta prioridad — acceso bloqueado | Ticket guardado + email de aprobación (requiere_humano=true) |
| T-05 | Canal inválido (Fax) | Filtro de validación bloquea + error registrado |

Evidencia en carpeta `/evidencia`.

---

## Seguridad

- API Keys gestionadas como credenciales encriptadas en n8n (nunca expuestas en el código)
- Email del cliente no persistido en Notion
- Tres rutas de error con log automático (validación, parseo, API down)
- Human-in-the-loop obligatorio antes de contactar al cliente en tickets de alta prioridad o bugs
- Webhook de único disparo — sin posibilidad de bucles infinitos
