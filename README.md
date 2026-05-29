# EKIO — Diagnóstico EHS

Sistema de diagnóstico de Electrohipersensibilidad (EHS) que reemplaza Typeform con solución propia.

## Stack

| Componente | Tecnología |
|-----------|-----------|
| Formulario | HTML + JS vanilla, mobile-first |
| Procesamiento | n8n self-hosted en VPS (Easypanel) |
| Email marketing | Klaviyo |
| WhatsApp | ManyChat (pendiente integración) |
| Ecommerce | Shopify |
| Deploy formulario | Vercel |

## Estructura del repositorio

```
ekio-formulario/
├── form/
│   └── ekio-diagnostico-v4.html   # Formulario HTML (25 preguntas EHS)
├── n8n/
│   └── ekio-n8n-workflow.json     # Workflow n8n exportado
├── docs/
│   └── PROGRESO.md                # Documentación del estado del proyecto
├── .env.example                   # Variables de configuración
└── README.md
```

## Despliegue

### Vercel (formulario)

El fichero `form/ekio-diagnostico-v4.html` está desplegado en Vercel con deploy automático en cada push a `main`.

**URL de producción:** https://ekio-klaviyo-n8n.vercel.app

### n8n (workflow)

1. Importar `n8n/ekio-n8n-workflow.json` en tu instancia n8n
2. Actualizar las credenciales de Klaviyo en los nodos HTTP
3. Activar el workflow

**Instancia n8n:** `https://crm-ekio-n8n.idv05l.easypanel.host`

## Actualizar la URL del webhook en el formulario

En `form/ekio-diagnostico-v4.html`, línea ~373:

```js
const N8N_WEBHOOK = 'https://crm-ekio-n8n.idv05l.easypanel.host/webhook/ekio-diagnostico';
```

Cambia esa URL si cambias de instancia n8n.

## Actualizar version stamp

En `renderWelcome()` (línea ~513) hay un stamp visible en la pantalla de inicio:

```html
<p ...>v4 · 28/05/2026 21:06 Madrid</p>
```

Actualiza fecha/hora cada vez que modifiques el formulario.

## Variables de entorno necesarias

Ver `.env.example` para todas las variables requeridas.

## Flujo completo

```
Usuario completa formulario
    |
    v
POST -> n8n webhook
    |
    v
n8n: calcula score + subscores + perfil EHS
    |
    v
Klaviyo: evento Diagnostico_Completado (perfil actualizado)
    |
    v
Klaviyo: evento Formulario_Cumplimentado
    |
    v
Klaviyo Flow "TEST n8n": envia email personalizado segun perfil
    |
    v (opcional)
ManyChat: activa flow WhatsApp si wa_opt = true
```

## Perfiles EHS

| Score | Perfil |
|-------|--------|
| 0-39 | Sin indicios claros de posible EHS |
| 40-58 | Indicaciones de posible desarrollo de ElectroSensibilidad |
| 59-71 | Potencial caso de Electro-Sensibilidad Intermitente |
| 72-110 | Potencial caso de Electro-Sensibilidad |
| 111+ | Potencial caso de EHS (Hipersensibilidad Electromagnetica) |
