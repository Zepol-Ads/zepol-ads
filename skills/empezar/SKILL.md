---
name: empezar
description: Arranque completo de ZEPOL ADS en un solo comando. Revisa que ZEPOL y Meta estén conectados, lee lo que el alumno llenó en la web (su negocio y su producto), encuentra solo sus activos de Meta (cuenta publicitaria, página, Instagram, píxel), estudia su cuenta, le entrega su plan de arranque y ofrece armar el borrador de su primera campaña. Úsala cuando el alumno escriba /zepol-ads:empezar, "empieza con ZEPOL ADS", "empezar con ZEPOL", "configura ZEPOL", "configura ZEPOL ADS" o "quiero empezar", cuando use ZEPOL por primera vez o cuando otra skill de ZEPOL diga que falta el arranque.
---

# Empezar con ZEPOL ADS

Un solo comando lleva al alumno desde la conexión hasta el borrador de su primera campaña. Haces el trabajo por él, de principio a fin, y solo te detienes en dos casos:

- **cuando hay que elegir**, por ejemplo si tiene dos cuentas publicitarias;
- **cuando hay que aprobar**: nada se publica sin su "sí".

Antes de cada paso largo, di en una línea qué estás haciendo ("Busco tus activos de Meta…"). No le pidas al alumno números de Meta: los encuentras tú.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tu cuenta"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Armar el borrador", "Elegir cuenta", "Llenarlo en la web".
- Errores con la forma qué pasó + cómo arreglarlo.
- Si tienes una herramienta para preguntas con opciones (por ejemplo, AskUserQuestion), úsala en las preguntas cerradas. Si no, escribe las opciones numeradas.
- Palabras simples: "tu cuenta publicitaria", no `act_…`; "el píxel de tu tienda", no "dataset".

## Reglas de esta skill

- En el arranque, con Meta solo lees. Usa solo herramientas de lectura del conector. El borrador de la campaña lo arma la skill `nueva-campana`, con sus propias reglas.
- La metodología vive en Zepol Cloud. Las skills que usas aquí (`configurar`, `diagnostico` y `nueva-campana`) ya saben pedirla. No improvises criterios propios.
- No inventes datos. Si algo falta y no se puede leer de Meta, pregúntalo una sola vez o anótalo como pendiente.
- Nunca pidas contraseñas, claves de API ni tokens. La clave de OpenAI se guarda en el panel de ZEPOL ADS, no en el chat.
- Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = empezar`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes. No reintentes en bucle.

## Paso 1. Revisar las conexiones

1. **ZEPOL.** Si no ves las herramientas de Zepol Cloud (por ejemplo `estado_cuenta`), di y detente: "ZEPOL ADS todavía no está conectado. En la app de Claude, entra a Customize → Plugins → ZEPOL ADS, abre la pestaña Conectores y toca Conectar. Inicia sesión con la cuenta de tu licencia y toca Permitir acceso. En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a escribir /zepol-ads:empezar."
2. **Licencia.** Llama `estado_cuenta` con `version_plugin = "0.5.0"` (la versión de este plugin).
   - Si la licencia no está activa, muestra el aviso de la herramienta tal cual y detente.
   - Si `versiones.actualizar_plugin` es `true`, avísalo en una línea ("Hay una versión nueva de ZEPOL: actualízala en Customize → Plugins cuando termines.") y sigue.
3. **Meta.** Si no ves las herramientas del conector oficial de Meta (por ejemplo `ads_get_ad_accounts`), di y detente: "Falta conectar Meta. En la app de Claude, entra a Customize → Connectors, busca Meta Ads, toca Conectar y entra con tu cuenta de Facebook. En Claude Code, escribe /mcp y autentica el conector de Meta. Después vuelve a escribir /zepol-ads:empezar."
4. **Arranque ya hecho.** Si `onboarding.arranque_completo` es `true`, di: "Tu arranque ya está hecho: tu cuenta está estudiada desde el {fecha de `onboarding.fechas.diagnostico`, en palabras}." y pregunta qué quiere hacer:
   - "Armar una campaña" → ve al Paso 6.
   - "Estudiar mi cuenta de nuevo" → sigue con el Paso 2.
   - "Ver mi rutina diaria" → ve al Paso 7.
5. Si no, di en una línea qué vas a hacer: "Te dejo listo en unos minutos: leo tu negocio, busco tus activos de Meta, estudio tu cuenta y te propongo tu primera campaña."

## Paso 2. Leer lo que llenaste en la web

1. Llama `obtener_perfil` sin parámetros.
2. Si hay varios perfiles, pregunta una vez con qué negocio empiezas.
3. **Sin perfil:**
   - Di: "No veo los datos de tu negocio. Lo más cómodo es llenarlos en la web, con botones, en https://zepol-ad-beacon.lovable.app/bienvenida. Toma unos 3 minutos. Si prefieres, te hago las preguntas aquí."
   - Opciones: "Responder aquí" y "Llenarlo en la web".
   - Con "Llenarlo en la web": "Cuando termines, vuelve aquí y escribe /zepol-ads:empezar." Detente.
   - Con "Responder aquí":
     - sigue la skill `configurar`, Paso 2, solo los bloques 1 a 5. No hagas el bloque 6: los activos de Meta los encuentras tú en el Paso 3;
     - después muestra su resumen y guarda con `guardar_perfil`, como en sus pasos 3 y 4;
     - no muestres su cierre y sigue con el punto 4.
4. **Producto.** Los productos llegan dentro del perfil (`productos`).
   - Si hay uno, ese es el producto del arranque.
   - Si hay varios, usa el más reciente y dilo: "Uso {nombre} para tu primera campaña; si quieres otro, me dices."
   - Si no hay ninguno, no lo pidas ahora: lo pide `nueva-campana` en el Paso 6.
5. Resume en dos o tres líneas lo que ya sabes, sin volver a preguntarlo. Por ejemplo: "Tengo tu negocio, Tienda Lima: vendes por WhatsApp y Shopify, con 2 personas atendiendo. Tu producto es la faja térmica, con su precio y su foto."

## Paso 3. Encontrar tus activos de Meta

Di: "Busco tus activos de Meta. No necesitas saber ningún número."

Lee con el conector y **elige solo** cuando hay una sola opción válida. Si hay varias, pregunta **una sola vez** por activo, con opciones que muestren el nombre (y el ID entre paréntesis). Todos los IDs se guardan solo con números: quita `act_` y cualquier otro texto.

Si el perfil ya tiene un activo guardado y sigue siendo válido, consérvalo sin preguntar.

1. **Cuenta publicitaria** → `meta_cuenta_publicitaria_id`.
   - Llama `ads_get_ad_accounts` y sigue `next_cursor` hasta tenerlas todas.
   - Una cuenta sirve si tiene `is_ads_mcp_enabled = true`, `is_queryable = true`, `has_payment_method = true` y `account_status = ACTIVE`.
   - Si ninguna sirve, explica por qué con cada cuenta y cómo arreglarlo. Por ejemplo: "Tu cuenta Tienda 2 no tiene método de pago. Agrégalo en la configuración de pagos del Administrador de anuncios." Después detente: "Cuando lo arregles, vuelve a escribir /zepol-ads:empezar."
2. **Zona horaria** → `extras.zona_horaria_cuenta`.
   - Lee `timezone_name` de la cuenta elegida: verifica el campo con `ads_get_field_context` y léelo con `ads_get_ad_entities` en `level = ad_account`.
   - Si no es `America/Lima`, avisa: "Tu cuenta publicitaria no está en hora de Lima. Los horarios de lanzamiento se calculan en la hora de tu cuenta."
3. **Página** → `meta_pagina_id`.
   - Cruza `ads_get_ad_account_pages` con `ads_get_user_pages`. Solo valen las páginas que están en las dos listas.
   - Si la página que el alumno quiere no está en `ads_get_user_pages`, explica cómo asignarse: "Entra a Configuración del negocio, abre Cuentas y luego Páginas, elige la página y asígnate en Asignar personas." Ofrece "Ya lo hice" para volver a revisar.
4. **Instagram** → `meta_instagram_id`.
   - Usa `ads_get_ig_accounts` con la cuenta elegida.
   - Si no hay ninguna, agrégalo a pendientes: "Vincular tu cuenta de Instagram en Configuración del negocio, en Cuentas de Instagram".
5. **Píxel** → `meta_pixel_id`.
   - Usa `ads_get_datasets` con la cuenta elegida. Si hay varios, pregunta: "¿Qué píxel tiene instalado tu tienda?"
   - Si no hay ninguno y el negocio vende por Shopify, agrégalo a pendientes: "Crear el píxel en el Administrador de eventos y conectarlo a tu tienda Shopify".
6. **Número de WhatsApp** → `whatsapp_numero`. Solo si el negocio vende por WhatsApp y el perfil no lo tiene.
   - El conector no lee el número de la página, así que es lo único que preguntas: "¿Cuál es el número de WhatsApp Business conectado a tu página?"
   - Valídalo y pregunta si está vinculado a la página, como en la skill `configurar`, bloque 6, punto 6.
7. **Guardar.** Llama `guardar_perfil` con:
   - `nombre_negocio`, `tipo_negocio` y `experiencia` del perfil;
   - los IDs que encontraste;
   - en `extras`: la zona horaria y, si las hay, la lista de pendientes nuevos, sumados a `extras.pendientes`.

   No pidas confirmación para guardar: no cambia nada en Meta. Muestra una tabla corta con dos columnas, "Activo" y "Cuál", con el nombre y el ID entre paréntesis.

## Paso 4. Estudiar tu cuenta

Di: "Estudio tu cuenta: accesos, píxel, historial por producto y anuncios ganadores. Toma unos minutos."

- Si el perfil ya tiene un diagnóstico guardado (`diagnostico_actualizado_en`), pregunta: "Ya estudié tu cuenta el {fecha}. ¿Uso ese estudio?" Opciones: "Usar ese estudio" (sigue con el Paso 5, con lo guardado en `diagnostico`) y "Estudiarla de nuevo".
- Sigue la skill `diagnostico`, pasos 1, 2, 3 y 5:
  - no hagas su "Antes de empezar": ya lo hiciste;
  - en su Paso 4 no preguntes qué producto va a anunciar: usa el producto del Paso 2 (`producto_id`). Si no hay producto, sáltalo;
  - guarda con `guardar_diagnostico` como dice su Paso 5, pero no muestres su resumen ni su "Siguiente paso": el resumen va en tu Paso 5.

## Paso 5. Tu plan de arranque

Muestra todo en una sola respuesta, con estos títulos y en palabras simples:

1. **Lo que funciona en tu cuenta:** de 2 a 4 puntos, con sus números.
2. **Tus ganadores:** hasta 3 anuncios, con su producto, cuánto gastaron, sus resultados y por qué ganaron. Si no hay: "Todavía no hay ganadores para reutilizar: tu producto arranca como nuevo."
3. **Alertas:** cada una con su arreglo. Por ejemplo: "Tu píxel no recibe ventas del servidor desde agosto. Revisa la conexión de la API de Conversiones en tu tienda." Si no hay alertas, dilo en una línea.
4. **Pendientes:** los de `extras.pendientes`, si hay.
5. **Tu primera acción recomendada:** una sola frase.
   - Si una alerta impide anunciar (sin método de pago, sin permiso en la página, WhatsApp sin vincular a la página), la primera acción es arreglarla.
   - Si no, es armar el borrador de la primera campaña con el producto del Paso 2, y reutilizar un ganador si lo hay.

## Paso 6. Tu primera campaña, como borrador

1. Pregunta: "¿Armo ahora el borrador de tu primera campaña con {producto}?" Opciones: "Armar el borrador" y "Más tarde".
   - Si no hay producto: "¿Armo ahora el borrador de tu primera campaña?"
   - Si la primera acción del Paso 5 es arreglar algo que impide anunciar, dilo antes de preguntar: "Puedo dejar el borrador listo, pero no se podrá publicar hasta que arregles {alerta}."
2. Con "Armar el borrador", sigue la skill `nueva-campana` desde su Paso 0:
   - el perfil, el producto y el diagnóstico ya están: no vuelvas a preguntar lo que ya está guardado;
   - en su Paso 1, muestra el producto y pregunta solo los datos que falten;
   - todo queda como borrador. Publicar sigue pidiendo el "sí" explícito del alumno, como dice esa skill.
3. Con "Más tarde": "Cuando quieras, escribe /zepol-ads:nueva-campana y lo armo con lo que ya sé de tu negocio."

## Paso 7. Tu rutina

Cierra siempre con esto, en pocas líneas:

- "Listo: ZEPOL ya conoce tu negocio y tu cuenta."
- "Cada mañana escribe /zepol-ads:optimizar. Te digo qué apagar, qué escalar y qué dejar quieto. Nada cambia sin tu sí."
- "Para ver cómo van tus campañas, escribe /zepol-ads:reporte."
- "Si algo no funciona como esperas, cuéntaselo a Alexis con /zepol-ads:feedback."
