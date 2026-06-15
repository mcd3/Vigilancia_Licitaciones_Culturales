# Vigilancia de Licitaciones Culturales (n8n + IA local)

Workflow de [n8n](https://n8n.io) que cada mañana revisa las licitaciones públicas del Estado español (PLACSP), filtra las del sector cultural y digital, las analiza con un modelo de IA local y envía las relevantes a Telegram y a una hoja de Google Sheets.

Pensado para empresas y autónomos del sector de la digitalización cultural, patrimonio, turismo, audiovisual, fotografía y contenidos digitales que quieran detectar oportunidades de contratación pública sin revisar el portal a mano.

## Qué hace

Cada día a las 7:00 (configurable):

1. Descarga el feed Atom de licitaciones de la Plataforma de Contratación del Sector Público, paginando hasta cubrir los últimos 3 días.
2. Se queda solo con las licitaciones en estado `PUB` (plazo de presentación abierto) y limpia el texto.
3. Filtra por palabras clave del sector cultural/digital y calcula una puntuación de relevancia.
4. Descarta las que no superan un umbral y las que ya se procesaron en días anteriores (deduplicación por identificador único de cada expediente, `idEvl`).
5. Analiza cada licitación con un modelo de IA local (vía Ollama), que devuelve objeto, presupuesto, criterios, encaje y una recomendación (presentarse / estudiar / descartar).
6. Envía las relevantes a Telegram y las registra en Google Sheets.

## Requisitos

- n8n (self-hosted o cloud).
- [Ollama](https://ollama.com) corriendo en local o en red, con un modelo descargado (por defecto `gemma3:27b`; cualquier modelo de instrucciones vale, ajustando el nombre).
- Un bot de Telegram y el chat ID donde recibir las alertas.
- Una hoja de Google Sheets y una credencial de cuenta de servicio con acceso a ella.

## Instalación

1. **Importa el workflow.** En n8n: menú -> *Import from File* -> selecciona el `.json` de este repositorio.

2. **Configura las credenciales** (los nodos vienen con marcadores `REEMPLAZAR_CON_...`):
   - Nodo de IA: cambia la URL del endpoint a la de tu Ollama. Si Ollama corre en la misma máquina que n8n, `http://localhost:11434/api/generate`; si está en otra máquina de tu red, usa su IP.
   - Nodos de Google Sheets: asigna tu credencial de cuenta de servicio y el ID de tu documento.
   - Nodo de Telegram: asigna tu credencial de bot y tu chat ID.

3. **Prepara la hoja de Google Sheets.** Crea (o usa) una hoja y anade estos encabezados en la primera fila, **incluida la columna `idEvl`**, que es la que permite la deduplicacion:

   ```
   idEvl | Fecha | Titulo | Organismo | Relevancia | Keywords | Presupuesto | Fecha Limite | Criterios | Encaje | Accion | Justificacion | Riesgos | Link
   ```

4. **Activa el workflow.** La primera ejecucion guardara todas las licitaciones encontradas (es la linea base); a partir de la segunda, solo te llegaran las nuevas.

## Personalizacion

- **Palabras clave y pesos:** en el nodo *Score de relevancia*. Cada palabra tiene un peso; la suma da la puntuacion. Anade o quita terminos segun tu sector.
- **Umbral minimo:** la constante `UMBRAL_MINIMO` en ese mismo nodo. Subelo para recibir menos y mas relevante, bajalo para recibir mas.
- **Ventana temporal:** en el nodo de descarga, la variable que resta dias a la fecha actual (por defecto 3).
- **Estados:** `ESTADOS_VALIDOS` en el nodo de descarga. Por defecto solo `PUB` (plazo abierto); anade `PRE` para incluir anuncios previos.
- **Modelo de IA:** cambia el nombre del modelo en el cuerpo de la peticion del nodo de IA.

## Notas

- El workflow no incluye credenciales ni datos privados. Todos los identificadores son marcadores que debes sustituir por los tuyos.
- La columna `idEvl` se guarda como texto (con un apostrofo inicial invisible) para evitar que Google Sheets interprete como formula los identificadores que empiezan por `+`, `=`, `-` o `@`.
- La IA se ejecuta de forma secuencial; con muchos resultados, una ejecucion puede tardar varios minutos. En el uso diario habitual, el numero de licitaciones nuevas es pequeno.
- Fuente de datos: Plataforma de Contratacion del Sector Publico (PLACSP), feed de sindicacion de perfiles de contratante.

## Licencia

MIT.
