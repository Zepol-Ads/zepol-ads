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

## Metodología: siempre de Zepol Cloud

- La metodología de ZEPOL vive en Zepol Cloud, no en este plugin. Eso incluye qué es un buen resultado y cómo se reutilizan los ganadores.
- Qué es un buen resultado y qué anuncio es ganador lo calcula Zepol Cloud: usa `evaluar_conjuntos` con `modo = historial` (Paso 3). Para reutilizar ganadores, pide `obtener_metodologia` con `etapa = creativos` y el canal de cada producto.
- No improvises criterios propios.
- Si no ves las herramientas de Zepol Cloud (por ejemplo `obtener_perfil`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Customize (Personalizar) → Plugins → ZEPOL ADS, abre la pestaña Connectors (Conectores) y toca Connect (Conectar). En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:diagnostico." y detente.
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.
- Si Zepol Cloud responde que el alumno llegó al límite de consultas de hoy, muéstrale ese mensaje tal cual y detente. No es un error: no lo registres con `registrar_error`.
- Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = diagnostico`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Antes de empezar

1. Llama `obtener_perfil`. Si no hay perfiles o el perfil no tiene cuenta publicitaria (`meta_cuenta_publicitaria_id`), di: "Antes del diagnóstico falta tu arranque: escribe /zepol-ads:empezar. Encuentra tus activos de Meta y estudia tu cuenta en el mismo paso." y detente. Si hay varios perfiles, pregunta con qué negocio trabajas. Usa sus activos: `meta_cuenta_publicitaria_id`, `meta_pagina_id`, `meta_instagram_id`, `meta_pixel_id`, `app_formulario` y `whatsapp_numero`.
2. Confirma que el conector de Meta está disponible (por ejemplo, `ads_get_ad_accounts`). Si no: "No encuentro el conector de Meta. En la app de Claude, entra a Customize (Personalizar) → Connectors (Conectores), busca Meta Ads y toca Connect (Conectar). En Claude Code, escribe /mcp y autentica el conector de Meta. Después vuelve a correr /zepol-ads:diagnostico."
3. Pide solo los campos que necesitas y pagina con los cursores. Las respuestas del conector son grandes y consumen el plan de Claude del alumno.

## Paso 1. Cuenta

- Usa la cuenta del perfil. Confirma con `ads_get_ad_accounts` que sigue activa, con método de pago, con `is_ads_mcp_enabled = true` y con `is_queryable = true`.
- Si algo falla, dilo con qué pasó + cómo arreglarlo y detente si la cuenta no se puede leer.

## Paso 2. Accesos y destino

1. **Página**: cruza `ads_get_ad_account_pages` con `ads_get_user_pages`. Si falta el permiso del alumno en la página del perfil, explica cómo asignarlo (Configuración del negocio, Cuentas, Páginas, Asignar personas) y vuelve a revisar cuando avise.
2. **Instagram**: confirma con `ads_get_ig_accounts` que la cuenta de Instagram del perfil sigue vinculada.
3. **Píxel y API de Conversiones**: con `ads_get_dataset_details`, `ads_get_dataset_stats` y `ads_get_dataset_quality`, revisa si el píxel del perfil está activo, qué eventos recibe y si llegan eventos del servidor (API de Conversiones).
   - Si no llegan eventos del servidor, avísalo con claridad y di desde cuándo, porque puede pasar meses sin que nadie lo note.
4. **Landing** (si el negocio vende por una web: Shopify, WooCommerce u otra): cuando el alumno indique la URL de un producto, revisa que responda (HTTP 200) y, si vende contra entrega, que tenga un formulario contra entrega. Mira tú qué app usa (Releasit u otra) y, si el perfil no la tiene, guárdala con `guardar_perfil` (`perfil_id`, `nombre_negocio` y `app_formulario`); no se la preguntes al alumno.
   - El resto de la revisión de la landing está en la metodología (`configuracion`, canal `shopify`).
5. **WhatsApp** (si el negocio vende por WhatsApp): recuerda que la página necesita el número de WhatsApp Business del perfil vinculado y verificado.

## Paso 3. Historial por producto

1. Lee campañas, conjuntos y anuncios con `ads_get_ad_entities` (verifica cada campo con `ads_get_field_context`) y los creativos con `ads_get_creatives`.
2. Agrupa campañas y anuncios **por producto** usando el enlace de destino de cada creativo.
3. Para cada anuncio, guarda el ID de su publicación (`{PAGE_ID}_{POST_ID}`). Es lo que permite reutilizar un ganador con sus reacciones y comentarios (ver la skill `armado-seguro`, regla 5).
4. Por cada producto y canal, envía todos sus conjuntos del historial a `evaluar_conjuntos` con `modo = historial`, en una sola llamada:
   - de cada conjunto: gasto, resultados, costo por resultado, días activo y, si los hay, el ROAS o el valor de las compras;
   - dentro de cada conjunto, sus anuncios (`anuncios`), cada uno con sus propios números (gasto, resultados, costo por resultado y, si los hay, el ROAS o el valor de las compras), su ID (`anuncio_id`) y el ID de su publicación (`object_story_id`);
   - el precio del producto (`precio_producto`) siempre que lo sepas: sin él, Zepol Cloud no puede saber si un anuncio es rentable.
   - Si son más de 50 conjuntos, envía los 50 con más gasto; de cada conjunto, hasta 30 anuncios, los de más gasto.
   - En el diagnóstico no propongas acciones: solo lees.
5. Lee lo que devuelve, sin calcular nada por tu cuenta:
   - un conjunto ganó si trae `ganador: true`;
   - **un anuncio es ganador solo si trae `ganador: true`**: Zepol Cloud ya revisó que su conjunto haya ganado, que su calificación sea buena y que sea rentable para el alumno (`rentable`). Esos son los únicos que se reutilizan;
   - si un anuncio trae `ganador: null` y una `pregunta_al_usuario`, hazle esa pregunta al alumno una sola vez (normalmente, el precio del producto) y vuelve a llamar a `evaluar_conjuntos` con la respuesta. Si no la sabe, no lo cuentes como ganador y dilo en el resumen;
   - un anuncio con `ganador: false` no se reutiliza, aunque tenga muchos resultados. Usa su `motivo` si el alumno pregunta por qué.
6. **Marca propia** (`categoria = marca_propia`): no califiques con `evaluar_conjuntos`, porque sus reglas son de contra entrega. Muestra los anuncios con más resultados y su ID de publicación sin llamarlos ganadores, y di que la lectura para marcas llega con su metodología.

## Paso 4. Producto a anunciar

- Marca propia: salta este paso (sus campañas llegan pronto) y ve al Paso 5.
- Pregunta qué producto quiere anunciar.
- Con lo leído y las calificaciones de `evaluar_conjuntos`, explícale al alumno con sus números si el producto ya tiene ganadores para reutilizar o si es un producto nuevo.
- Si el producto ya está en el perfil (`obtener_perfil` trae los productos), usa su `id` como `producto_id` en el diagnóstico.

## Paso 5. Resultado

1. Muestra un resumen corto:
   - accesos: qué está bien y qué falta, cada cosa con su arreglo;
   - productos encontrados en el historial;
   - anuncios ganadores (los que trajeron `ganador: true`) con su ID de publicación, y los que quedaron sin decidir porque faltó un dato.
2. Guarda el diagnóstico en Zepol Cloud con `guardar_diagnostico`, con el `perfil_id` del perfil. La herramienta trae el formato:
   - `fecha_lectura` y la `cuenta` (ID solo con números, estado, método de pago y zona horaria);
   - `accesos`: uno por activo, con su estado y, si falta algo, cómo arreglarlo;
   - `productos` encontrados en el historial, con los IDs de sus campañas;
   - `ganadores`: solo los anuncios con `ganador: true`, con el ID del anuncio, el de su publicación (`object_story_id`), su `calificacion`, `rentable` y sus números.
   - Reemplaza el diagnóstico anterior. Si la herramienta rechaza un dato, corrígelo y vuelve a enviarlo.
3. Termina con el siguiente paso:
   - contra entrega: "Siguiente paso: escribe /zepol-ads:nueva-campana para armar la campaña de tu producto.";
   - marca propia (`categoria = marca_propia`): "Las campañas para marcas propias llegan pronto. Mientras tanto, revisa tus campañas actuales con /zepol-ads:reporte."
