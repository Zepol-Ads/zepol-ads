---
name: diagnostico
description: Estudia la cuenta publicitaria del alumno antes de armar campañas con ZEPOL ADS. Revisa accesos (página, Instagram, píxel y API de Conversiones, landing), agrupa el historial por producto y encuentra los anuncios ganadores con el ID de su publicación para reutilizarlos. Solo lee, no cambia nada en Meta. Úsala cuando el alumno escriba /zepol-ads:diagnostico o antes de /zepol-ads:nueva-campana si no hay un diagnóstico reciente.
---

# Diagnóstico de la cuenta

Cada cuenta publicitaria funciona distinto. Antes de proponer nada, ZEPOL estudia la cuenta: sus accesos, su historial y lo que ya ganó.

Esta skill **solo lee**. No crea, edita ni publica nada en Meta.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tu cuenta"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Asignar permiso", "Ver ganadores".
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre del servidor

- La metodología de ZEPOL vive en el servidor de ZEPOL, no en este plugin. Eso incluye qué cuenta como ganador, qué métricas y cortes leer, y cómo interpretar el historial.
- Pídela con la herramienta `metodologia` del servidor de ZEPOL. TODO(servidor): la herramienta aún no existe; nombre y parámetros por confirmar.
- Si el servidor no está conectado (no ves sus herramientas), no improvises criterios propios. Haz los pasos de lectura y accesos, y en los pasos que dependen del servidor di: "El servidor de ZEPOL aún no está conectado, así que todavía no puedo elegir los ganadores de tu cuenta. Cuando esté listo, vuelve a correr /zepol-ads:diagnostico."

## Antes de empezar

1. Lee el perfil `zepol/perfil-negocio.json`. Si no existe: "No encuentro tu perfil de negocio. Corre /zepol-ads:configurar primero; toma unos 5 minutos."
2. Confirma que el conector de Meta está disponible (por ejemplo, `ads_get_ad_accounts`). Si no: "No encuentro el conector de Meta. Conéctalo desde la sección Conectores de Claude y vuelve a correr /zepol-ads:diagnostico."
3. Pide solo los campos que necesitas y pagina con los cursores. Las respuestas del conector son grandes y consumen el plan del alumno.

## Paso 1. Cuenta

- Usa la cuenta del perfil. Confirma con `ads_get_ad_accounts` que sigue activa, con método de pago, con `is_ads_mcp_enabled = true` y con `is_queryable = true`.
- Si algo falla, dilo con qué pasó + cómo arreglarlo y detente si la cuenta no se puede leer.

## Paso 2. Accesos y destino

1. **Página**: cruza `ads_get_ad_account_pages` con `ads_get_user_pages`. Si falta el permiso del alumno en la página del perfil, explica cómo asignarlo (Configuración del negocio, Cuentas, Páginas, Asignar personas) y vuelve a revisar cuando avise.
2. **Instagram**: confirma con `ads_get_ig_accounts` que la cuenta de Instagram del perfil sigue vinculada.
3. **Píxel y API de Conversiones**: con `ads_get_dataset_details`, `ads_get_dataset_stats` y `ads_get_dataset_quality`, revisa si el píxel del perfil está activo, qué eventos recibe y si llegan eventos del servidor (API de Conversiones).
   - Si no llegan eventos del servidor, avísalo con claridad y di desde cuándo, porque puede pasar meses sin que nadie lo note.
4. **Landing** (si el negocio vende por Shopify): cuando el alumno indique la URL de un producto, revisa que responda (HTTP 200) y que tenga el formulario contra entrega de la app del perfil.
   - TODO(servidor): el resto de la revisión de la landing lo define el servidor.
5. **WhatsApp** (si el negocio vende por WhatsApp): recuerda que la página necesita el número de WhatsApp Business del perfil vinculado y verificado.

## Paso 3. Historial por producto

1. Lee campañas, conjuntos y anuncios con `ads_get_ad_entities` (verifica cada campo con `ads_get_field_context`) y los creativos con `ads_get_creatives`.
2. Agrupa campañas y anuncios **por producto** usando el enlace de destino de cada creativo.
3. Para cada anuncio, guarda el ID de su publicación (`{PAGE_ID}_{POST_ID}`). Es lo que permite reutilizar un ganador con sus reacciones y comentarios (ver la skill `armado-seguro`, regla 5).
4. TODO(servidor): pedir a `metodologia` (etapa `diagnostico`) qué métricas, cortes y periodos leer por producto, y cómo identificar la estructura y los creativos ganadores. Lee solo eso.

## Paso 4. Producto a anunciar

- Pregunta qué producto quiere anunciar.
- TODO(servidor): con lo leído, pedir a `metodologia` la recomendación para ese caso (producto ya anunciado o producto nuevo) y explicarla al alumno con sus números.

## Paso 5. Resultado

1. Muestra un resumen corto:
   - accesos: qué está bien y qué falta, cada cosa con su arreglo;
   - productos encontrados en el historial;
   - ganadores con su ID de publicación, cuando el servidor los haya identificado.
2. Guarda el diagnóstico en `zepol/diagnostico-{cuenta_publicitaria_id}.json` con tu herramienta de escritura de archivos, con la fecha de lectura.
   - TODO: definir el formato del archivo de diagnóstico.
   - TODO(servidor): guardarlo en Zepol Cloud como "perfil de la cuenta" y actualizarlo con cada reporte, para no volver a leer todo cada vez.
3. Termina con el siguiente paso: "Siguiente paso: corre /zepol-ads:nueva-campana para armar la campaña de tu producto."
