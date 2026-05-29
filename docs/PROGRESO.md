# PROGRESO — EKIO Diagnostico EHS

Ultima actualizacion: 29/05/2026

---

## Estado actual: FUNCIONAL

El pipeline completo esta operativo: formulario → n8n → Klaviyo.
Pendiente: clonar flow "by EDE" en "TEST n8n" + integracion ManyChat.

---

## Lo construido

### 1. Formulario HTML (`form/ekio-diagnostico-v4.html`)

- 25 preguntas de rating 0-10 sobre sintomas EHS
- Auto-avance al pulsar una respuesta (280ms de delay)
- Progreso visual con barra superior
- Pantalla de consentimiento legal (RGPD)
- Pantalla de datos de contacto: nombre, ciudad, email, WhatsApp (opcional)
- Opt-in WhatsApp checkbox
- Pantalla de gracias con instrucciones post-envio
- Mobile-first, max-width 480px
- Animacion slide entre pantallas
- Version stamp visible: v4 · 28/05/2026 21:06 Madrid

**Categorias de sintomas (subscores):**
| Categoria | Preguntas incluidas |
|-----------|-------------------|
| energia | fatiga, sueno_exceso |
| sueno | insomnio |
| emocional | irritabilidad, nerviosismo |
| piel | piel_erupciones, pinchazos_piel |
| cardiovascular | taquicardia, presion_arterial, presion_pecho, falta_oxigeno |
| digestivo | colon_irritable |
| auditivo | perdida_auditiva, dolor_oidos, tinnitus |
| neurologico | dolor_cejas, mareo, concentracion, memoria, dolor_cabeza, presion_cabeza, banda_cabeza |
| visual | vision_borrosa |
| facial | enrojecimiento, calor_rostro |

### 2. Workflow n8n (`n8n/ekio-n8n-workflow.json`)

8 nodos en total:

| Nodo | Tipo | Funcion |
|------|------|---------|
| Webhook EKIO | Webhook | Recibe POST del formulario (path: ekio-diagnostico) |
| Preparar payload | Code (JS) | Calcula score, subscores, perfil; construye payload Klaviyo |
| Klaviyo — Crear Evento | HTTP POST | Envia evento Diagnostico_Completado a Klaviyo API v2 |
| Klaviyo — Formulario Cumplimentado | HTTP POST | Envia evento legacy Formulario Cumplimentado (compatibilidad) |
| ¿WhatsApp opt-in? | IF | Bifurca segun wa_opt === true |
| ManyChat — Activar flow | HTTP POST | (DESACTIVADO) Envia a ManyChat cuando wa_opt = true |
| Sin WhatsApp | NoOp | Rama cuando no hay opt-in WhatsApp |
| Respond to Webhook | Respond | Devuelve 200 con el payload procesado |

**Propiedades criticas enviadas a Klaviyo:**
- `score_total` — score numerico total
- `Score_TF_EncuestaSensibilidad` — alias de compatibilidad con el flow "by EDE"
- `perfil_ehs` — label del perfil
- `perfil_slug` — slug del perfil (sin_indicios, posible_inicio, intermitente, electrosensible, ehs)
- `nivel_ehs` — numero de nivel (1-5)
- `subscore_*` — 10 subscores por categoria

### 3. Integracion Klaviyo

- Evento: `Diagnostico_Completado` (trigger del flow TEST n8n)
- Evento legacy: `Formulario Cumplimentado` (compatibilidad con flow original)
- Perfiles creados/actualizados automaticamente via API v2
- Todos los perfiles quedan con Email: Suscrito en Klaviyo

### 4. Perfiles EHS y umbrales

| Score | Nivel | Slug | Label |
|-------|-------|------|-------|
| 0-39 | 1 | sin_indicios | Sin indicios claros de posible EHS |
| 40-58 | 2 | posible_inicio | Indicaciones de posible desarrollo de ElectroSensibilidad |
| 59-71 | 3 | intermitente | Potencial caso de Electro-Sensibilidad Intermitente |
| 72-110 | 4 | electrosensible | Potencial caso de Electro-Sensibilidad |
| 111+ | 5 | ehs | Potencial caso de EHS (Hipersensibilidad Electromagnetica) |

---

## Pendiente

- [ ] Clonar estructura de flow "by EDE" (ID: RQ83ti) en flow "TEST n8n" (ID: WAhmU7)
  - Leer acciones y emails del flow origen via Klaviyo API
  - Recrear condicionales usando Score_TF_EncuestaSensibilidad
  - Recrear emails con contenido HTML original
- [ ] Integrar ManyChat (nodo desactivado en n8n — falta token)
- [ ] Deploy en Shopify (formulario actualmente en Vercel)
- [ ] Verificar pipeline end-to-end con un submit real

---

## URLs del sistema

| Servicio | URL |
|---------|-----|
| Formulario (Vercel) | https://ekio-klaviyo-n8n.vercel.app |
| n8n instancia | https://crm-ekio-n8n.idv05l.easypanel.host |
| Webhook n8n | https://crm-ekio-n8n.idv05l.easypanel.host/webhook/ekio-diagnostico |
| Klaviyo flow origen | https://www.klaviyo.com/flow/RQ83ti |
| Klaviyo flow destino | https://www.klaviyo.com/flow/WAhmU7 |

---

## Historial de cambios

| Fecha | Version | Cambio |
|-------|---------|--------|
| 28/05/2026 | v4 | Version inicial: 25 preguntas, workflow n8n 8 nodos, integracion Klaviyo |
| 29/05/2026 | — | Repositorio GitHub creado, deploy Vercel configurado |
