# 🏛️ Vigilancia de Licitaciones Culturales

Workflow de [n8n](https://n8n.io) que vigila a diario las licitaciones públicas españolas (PLACSP), filtra las relacionadas con **digitalización cultural, patrimonio, realidad aumentada, contenidos audiovisuales y turismo digital**, las analiza con un modelo de IA local y envía alertas a Telegram y a una hoja de Google Sheets.

Pensado para empresas y autónomos del sector cultural/tecnológico que quieren detectar oportunidades de contratación pública sin revisar manualmente el boletín cada día.

## ¿Qué hace?

Cada mañana, de forma automática:

1. **Descarga** el feed Atom diario de la Plataforma de Contratación del Sector Público (PLACSP), siguiendo la paginación hasta cubrir los últimos días.
2. **Parsea** cada licitación extrayendo título, organismo, importe, CPV, fechas y enlace.
3. **Filtra** por palabras clave y códigos CPV del sector cultural/digital.
4. **Puntúa** la relevancia de cada una (score ponderado → BAJA / MEDIA / ALTA / MUY ALTA).
5. **Analiza** las candidatas con un modelo de IA local (vía Ollama), que valora el encaje y recomienda una acción (PRESENTARSE / ESTUDIAR / DESCARTAR).
6. **Notifica** las relevantes por Telegram y las registra en Google Sheets.

```
Schedule → Descargar+Paginar+Parsear → Filtro keywords+CPV → Score
   → IA local → Parsear respuesta → Filtrar descartadas → Formatear
   → Telegram + Google Sheets
```

## Requisitos

- **n8n** (probado en 1.120.x, self-hosted en Docker).
- **Ollama** con un modelo conversacional instalado. El workflow usa `gemma3:27b`, pero sirve cualquier modelo compatible (ajusta el nombre en el nodo de IA).
- Un **bot de Telegram** (creado con [@BotFather](https://t.me/BotFather)) y tu Chat ID.
- Una **hoja de Google Sheets** y credenciales de Google en n8n (OAuth2 o Service Account).

> El feed de PLACSP es público y no requiere autenticación.

## Instalación

1. Descarga `Vigilancia_Licitaciones_Culturales.json`.
2. En n8n: menú **⋮ → Import from File** y selecciona el JSON.
3. Configura los placeholders (ver más abajo).
4. Activa el workflow.

## Configuración (placeholders a rellenar)

Tras importar, hay que sustituir estos valores marcados en el workflow:

### 🤖 Nodo "IA local"
- **URL**: por defecto `http://localhost:11434/api/generate`. Si Ollama corre en otra máquina o n8n está en Docker, ajústala (p. ej. `http://host.docker.internal:11434/api/generate` o la IP de tu servidor).
- **model**: cambia `gemma3:27b` por el modelo que tengas instalado (`ollama list` para verlos).

### 📱 Nodo "Telegram"
- **Chat ID**: sustituye `REEMPLAZAR_CON_CHAT_ID` por tu Chat ID.
  Para obtenerlo: escribe a tu bot y abre `https://api.telegram.org/bot<TU_TOKEN>/getUpdates`; busca `"chat":{"id":...}`.
- **Credencial**: crea una credencial de Telegram en n8n con el token de tu bot.

### 📊 Nodo "Google Sheets"
- **Document ID**: sustituye `REEMPLAZAR_CON_ID_DE_GOOGLE_SHEET` por el ID de tu hoja (está en la URL, entre `/d/` y `/edit`).
- **Credencial**: configura una credencial de Google Sheets (OAuth2 o Service Account). Si usas Service Account, comparte la hoja con su email como Editor.
- La hoja debe tener en la **fila 1** estos encabezados:

```
Fecha | Título | Organismo | Relevancia | Keywords | Presupuesto | Fecha Límite | Criterios | Encaje | Acción | Justificación | Riesgos | Link
```

## Personalización

### Palabras clave y CPV
El nodo **🔍 Filtro keywords + CPV** trae ~55 términos del sector cultural/digital y varios códigos CPV. Edítalo para adaptarlo a tu actividad: añade o quita condiciones (combinador en modo **OR / any**).

> Cuanto más genéricas sean las palabras (p. ej. "plataforma", "web"), más falsos positivos pasarán. No es grave: la IA los evalúa después y descarta los que no encajan, pero consume tiempo de cómputo en cada uno.

### Pesos del score
El nodo **📊 Score de relevancia** asigna un peso a cada término. Ajústalos para que las oportunidades que más te interesan puntúen alto.

### Ventana temporal
En el nodo **🌀 Descargar+Paginar+Parsear**, la línea:

```javascript
const desde = new Date(Date.now() - 3 * 24 * 60 * 60 * 1000)...
```

controla cuántos días hacia atrás se revisan. Cambia el `3` por los días que quieras (un pequeño solape evita perder licitaciones publicadas a última hora).

### Prompt de la IA
El nodo de IA contiene el prompt que guía el análisis. Adáptalo a tu perfil de empresa para mejorar la precisión del encaje.

## Notas técnicas

- El parseo del XML se hace con JavaScript puro (sin dependencias externas), porque el sandbox de n8n no permite `require()` de módulos como `xml2js`.
- El nodo de IA procesa las licitaciones **una a una**; cada análisis tarda según tu hardware. Si tu filtro deja pasar muchas, la ejecución se alarga. Afina las keywords o añade un umbral de score mínimo antes de la IA.
- Fuente de datos: feed Atom de PLACSP (`licitacionesPerfilesContratanteCompleto3.atom`).

## Aviso

Este proyecto se comparte **sin credenciales ni datos privados**. Antes de subir cualquier modificación a un repositorio público, verifica que no incluyes tokens, IDs de chat, IPs internas ni IDs de credenciales.

## Licencia

Publicado bajo licencia MIT. Úsalo y modifícalo libremente.
