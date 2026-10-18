---
name: configurar
description: Crea o actualiza el perfil del negocio de ZEPOL ADS con una entrevista corta (tipo de negocio y experiencia, atención por WhatsApp, precio en los anuncios, pruebas sociales, tasa de entrega, app del formulario y activos de Meta) y lo guarda como JSON en la carpeta de trabajo. Úsala cuando el alumno escriba /zepol-ads:configurar, cuando empiece a usar ZEPOL o cuando otra skill de ZEPOL no encuentre el perfil.
---

# Configurar el perfil del negocio

Esta skill arma el perfil del negocio una sola vez. Las demás skills de ZEPOL lo leen y lo aplican en cada campaña sin volver a preguntar.

- Perfil: `zepol/perfil-negocio.json` en la carpeta de trabajo del alumno.
- Formato exacto: `esquemas/perfil-negocio.schema.json` (en la carpeta de esta skill). Hay un ejemplo con datos ficticios en `esquemas/ejemplo-perfil.json`.
- Esta skill también define el formato de los datos por producto (`esquemas/producto.schema.json`). No los pide aquí: los pide `/zepol-ads:nueva-campana` para cada producto.
- TODO(servidor): cuando exista Zepol Cloud, guardar el perfil en Supabase con la herramienta del servidor (nombre por definir, por ejemplo `guardar_perfil`) y dejar el archivo local solo como respaldo.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tu cuenta"), nunca "nosotros".
- Botones y llamados a la acción con verbo + objeto: "Guardar perfil", "Elegir página", "Corregir un dato".
- Errores con la forma qué pasó + cómo arreglarlo: "No encuentro tu píxel. Créalo en el Administrador de eventos y dime 'listo' para revisarlo de nuevo."
- Una pregunta o un bloque corto a la vez. Si el alumno ya dio una respuesta en su mensaje, no la vuelvas a pedir.
- Si el alumno trae todas o casi todas las respuestas en su mensaje, no hagas la presentación de la entrevista: pregunta solo lo que falte y pasa al resumen.
- Si tienes una herramienta para preguntas con opciones (por ejemplo, AskUserQuestion), úsala en las preguntas cerradas. Si no, escribe las opciones numeradas.

## Reglas de esta skill

- Con Meta solo lees. Usa únicamente herramientas de lectura del conector (`ads_get_ad_accounts`, `ads_get_ad_account_pages`, `ads_get_user_pages`, `ads_get_ig_accounts`, `ads_get_datasets`, `ads_get_field_context`, `ads_get_ad_entities`). No crees, edites ni publiques nada.
- No inventes datos. Si el alumno no sabe algo, guárdalo como `null` y agrégalo a `pendientes`.
- Nunca pidas contraseñas, claves de API ni tokens. Si el alumno los escribe, no los guardes y dile que no hacen falta aquí.
- Las pruebas sociales deben ser reales y demostrables. Si el alumno no lo confirma, no se usan.

## Paso 0. Revisar si ya hay un perfil

1. Busca `zepol/perfil-negocio.json` en la carpeta de trabajo.
2. Si existe, muestra un resumen corto y pregunta: "Ya tienes un perfil guardado. ¿Qué quieres cambiar?" con las opciones "Actualizar un dato", "Rehacer el perfil completo" y "Dejarlo como está". Para actualizar, pregunta solo ese bloque y conserva `creado`.
3. Si no existe, saluda en una línea y empieza: "Te hago unas preguntas sobre tu negocio. Toma unos 5 minutos y solo se hace una vez." Si el alumno ya trajo sus respuestas, omite esta presentación.

## Paso 1. Revisar el conector de Meta

- Si ves herramientas del conector oficial de Meta (por ejemplo, `ads_get_ad_accounts`), úsalas en el bloque 6 para ofrecer opciones. Guarda `meta.origen = "conector"`.
- Si no las ves y el alumno ya dijo que sigue a mano, no repitas el aviso: guarda `meta.origen = "manual"` y continúa.
- Si no las ves y no lo dijo, avisa y ofrece seguir a mano:
  - "No encuentro el conector de Meta. Conéctalo desde la sección Conectores de Claude (Meta Ads) con tu cuenta de Facebook y vuelve a correr /zepol-ads:configurar. Si prefieres, sigo ahora y te pido los datos a mano."
  - Opciones: "Seguir a mano" y "Conectar Meta primero". Si sigue a mano, guarda `meta.origen = "manual"`.
- Si una llamada falla, explica el error con qué pasó + cómo arreglarlo y ofrece seguir a mano. No reintentes en bucle.

## Paso 2. Entrevista

Haz los bloques en este orden. Cada pregunta indica el campo del JSON donde va la respuesta.

### Bloque 1. Tu negocio

1. "¿Cómo se llama tu negocio?" → `negocio.nombre`.
2. "¿Por dónde vendes?" → `negocio.tipo`
   - "Solo por mi tienda Shopify" → `solo_shopify`
   - "Solo por WhatsApp" → `solo_whatsapp`
   - "Por los dos" → `hibrido`
3. "¿Cuánta experiencia tienes vendiendo por internet?" → `negocio.experiencia`
   - "Recién empiezo" → `principiante`
   - "Ya tengo experiencia" → `con_experiencia`
4. "¿A dónde haces envíos?" → `negocio.cobertura_envios`, en sus palabras ("Todo el Perú", "Solo Lima y Callao").
5. `negocio.pais` siempre es `PE`. Si el alumno vende en otro país, dile: "Por ahora ZEPOL solo funciona para campañas en Perú." y sigue con Perú.

### Bloque 2. WhatsApp

Si `negocio.tipo` es `solo_shopify`, no hagas este bloque: guarda `whatsapp.usa_whatsapp = false` y el resto en `null`.

1. "¿Quién responde los chats de WhatsApp?" → `whatsapp.atencion`
   - "Un bot" → `bot`
   - "A mano" → `a_mano`
   - "Las dos cosas" → `bot_y_a_mano`
2. "¿Cuántas personas atienden los chats?" → `whatsapp.personas_que_atienden` (número entero).
3. "¿Sientes que te falta personal para atender todos los chats?" → `whatsapp.poco_personal` ("Sí" → `true`, "No" → `false`).
4. "¿Con qué atiendes WhatsApp?" → `whatsapp.herramienta`
   - "La app WhatsApp Business" → `app_whatsapp_business`
   - "Otra herramienta (un CRM o la API)" → `otra_herramienta`, y pregunta cuál → `whatsapp.herramienta_nombre`
   - "WhatsApp normal" → `whatsapp_personal`. Agrega a `pendientes`: "Pasar el número a WhatsApp Business".
5. Solo con la app WhatsApp Business: "¿Marcas tus chats con las etiquetas de pedido de WhatsApp Business (Nuevo pedido, Pago pendiente, Pagado, Pedido completado)?" → `whatsapp.etiqueta_chats_de_pedido`
   - "Sí, siempre" → `si`
   - "A veces" → `a_veces`
   - "No" → `no`
   - Si no es `si`, agrega a `pendientes`: "Etiquetar todos los chats de pedido en WhatsApp Business".
   - TODO(servidor): mostrar la guía de etiquetado de chats que entregue el servidor de ZEPOL.
   - Con otra herramienta o WhatsApp normal, guarda `null`.

Guarda `whatsapp.usa_whatsapp = true` en este bloque.

### Bloque 3. Precio en los anuncios

1. "¿Quieres que tus anuncios muestren el precio?" → `anuncios.muestra_precio` ("Sí, mostrar el precio" → `true`, "No mostrarlo" → `false`).
2. Aclara que es decisión de su negocio y que queda guardada para todas sus campañas.

### Bloque 4. Pruebas sociales

Introduce el bloque así: "Ahora unos datos que ZEPOL puede usar en tus anuncios. Usa solo datos reales que puedas demostrar."

1. "¿A cuántos clientes has atendido, más o menos?" → `pruebas_sociales.clientes_aprox` (número entero; "No sé" → `null`).
2. "¿Tienes una calificación pública? Dime cuánto y dónde, por ejemplo 4.8 de 5 en Google." → `pruebas_sociales.calificacion` como `{valor, escala, fuente}`. Si no tiene, `null`.
3. "¿Alguien reconocido recomienda tu producto o tu tienda? Por ejemplo, un especialista o un medio." → `pruebas_sociales.recomendaciones` (lista de textos en sus palabras; si no hay, lista vacía).
4. Si dio al menos un dato: "¿Confirmas que estos datos son reales y puedes demostrarlos?" → `pruebas_sociales.confirmado_como_real`.
   - Si dice que no, guarda `false` y explica: "No los voy a usar en tus anuncios. Cuando los tengas, corre /zepol-ads:configurar y los agregas."
   - Si no dio ningún dato, guarda `false`.

### Bloque 5. Entregas

1. "De cada 100 pedidos contra entrega, ¿cuántos se entregan y se pagan?" → `entregas.tasa_entrega_pct` (número de 0 a 100).
   - Explica en una línea para qué sirve: "Con esto calculo cuánto te cuesta cada pedido que sí se entrega."
   - "No sé" → `null` y agrega a `pendientes`: "Calcular la tasa de entrega contra entrega".
   - Si responde con una fracción o con pedidos totales (por ejemplo, "de 200 se entregan 110"), conviértelo a porcentaje y confírmalo.
2. Si `negocio.tipo` no es `solo_whatsapp`: "¿Qué app usas para el formulario contra entrega de tu tienda?" → `entregas.app_formulario`
   - "Releasit" → `releasit`
   - "Otra" → `otra`, y pregunta cuál → `entregas.app_formulario_nombre`
   - "No uso formulario" → `ninguna`, y agrega a `pendientes`: "Instalar un formulario contra entrega en la tienda".
   - Si es `solo_whatsapp`, guarda `entregas.app_formulario = null`.

### Bloque 6. Activos de Meta

Con el conector, lee y ofrece opciones. A mano, pide cada dato y explica dónde encontrarlo en una línea.

En los dos casos:

- Todos los IDs se guardan solo con números: quita `act_` y cualquier otro texto.
- `meta.instagram_usuario` se guarda sin la arroba (`tiendaejemplo`, no `@tiendaejemplo`). En el resumen sí se muestra con arroba.

1. **Cuenta publicitaria** → `meta.cuenta_publicitaria_id`, `meta.cuenta_publicitaria_nombre`, `meta.portafolio_id`, `meta.portafolio_nombre`.
   - Llama `ads_get_ad_accounts` y sigue `next_cursor` hasta tener todas.
   - Ofrece solo las cuentas con `is_ads_mcp_enabled = true`. Si alguna tiene `is_queryable = false`, `has_payment_method = false` o un `account_status` distinto de `ACTIVE`, dilo junto a su nombre, por ejemplo: "Cuenta 2 no tiene método de pago. Agrégalo en la configuración de pagos del Administrador de anuncios para poder usarla."
   - Si solo hay una, confírmala. Si hay varias, pregunta: "¿Con qué cuenta publicitaria vas a anunciar?"
   - A mano: "Abre el Administrador de anuncios: el número que aparece después de act= en la dirección es el ID de tu cuenta."
2. **Zona horaria** → `meta.zona_horaria`.
   - Con el conector, verifica el campo `timezone_name` con `ads_get_field_context` y léelo con `ads_get_ad_entities` en `level = ad_account` para la cuenta elegida.
   - A mano o si falla, pregunta: "¿Tu cuenta publicitaria está en hora de Lima?" Si dice que sí, guarda `America/Lima`; si no sabe, `null` y agrega a `pendientes`: "Confirmar la zona horaria de la cuenta".
3. **Página** → `meta.pagina_id`, `meta.pagina_nombre`.
   - Llama `ads_get_ad_account_pages` (páginas de la cuenta) y `ads_get_user_pages` (páginas donde el alumno puede anunciar). Cruza las dos listas.
   - Ofrece las páginas que están en las dos. Si hay más de una, pregunta: "¿Qué página usas para tus anuncios?"
   - Si la página que quiere no está en `ads_get_user_pages`, explica: "No tienes permiso para anunciar con la página {nombre}. Si eres administrador del portafolio, entra a Configuración del negocio, abre Cuentas y luego Páginas, elige la página y asígnate en Asignar personas. Avísame cuando lo hagas y lo reviso de nuevo." Vuelve a llamar ambas herramientas cuando avise.
4. **Instagram** → `meta.instagram_id`, `meta.instagram_usuario`.
   - Llama `ads_get_ig_accounts` con la cuenta elegida. Si hay varias, pregunta cuál. Si hay una, confírmala.
   - Si no hay ninguna: "No encuentro una cuenta de Instagram vinculada a tu cuenta publicitaria. Vincúlala en Configuración del negocio, en Cuentas de Instagram, y vuelve a correr /zepol-ads:configurar." Guarda `null` y agrégalo a `pendientes`.
5. **Píxel** → `meta.pixel_id`, `meta.pixel_nombre`.
   - Llama `ads_get_datasets` con la cuenta elegida. Si hay varios, pregunta: "¿Qué píxel tiene instalado tu tienda?"
   - Si no hay ninguno y el negocio vende por Shopify: "No encuentro un píxel en esta cuenta. Créalo en el Administrador de eventos y conéctalo a tu tienda Shopify." Guarda `null` y agrégalo a `pendientes`.
   - Aquí solo eliges el píxel. Si recibe eventos lo revisa `/zepol-ads:diagnostico`.
6. **Número de WhatsApp** → `meta.whatsapp_numero`.
   - Pregúntalo si `whatsapp.usa_whatsapp` es `true`; si no, guarda `null`.
   - "¿Cuál es el número de WhatsApp Business conectado a tu página?"
   - El conector no lee el número de la página, así que lo escribe el alumno.
   - Quita espacios, guiones y el prefijo. Debe quedar `+51` seguido de 9 dígitos que empiezan con 9, por ejemplo `+51987654321`.
   - Si no cumple: "Ese número no parece un celular de Perú. Escríbelo con sus 9 dígitos, por ejemplo 987 654 321."
   - Pregunta: "¿Ese número ya está vinculado a tu página de Facebook?" → `meta.whatsapp_vinculado` ("Sí" → `true`, "No" → `false`, "No sé" → `null`). Es lo que dice el alumno; `/zepol-ads:diagnostico` y `/zepol-ads:nueva-campana` lo comprueban.
   - Si no es `true`, agrega a `pendientes`: "Vincular el número de WhatsApp Business a la página". Los anuncios a WhatsApp no se pueden activar sin ese vínculo.
   - Si no usa WhatsApp, guarda `meta.whatsapp_vinculado = null`.

## Paso 3. Resumen y confirmación

1. Muestra una tabla con dos columnas, "Dato" y "Valor", en palabras simples (por ejemplo "Shopify y WhatsApp", no `hibrido`). Usa estas filas, juntando los datos de cada una:
   - Negocio: nombre, por dónde vende y experiencia.
   - Envíos.
   - WhatsApp: quién atiende, cuántas personas, si falta personal, herramienta y etiquetas. Omite la fila si no usa WhatsApp.
   - Precio en los anuncios.
   - Pruebas sociales: clientes, calificación y recomendaciones, o "No se usan" si no están confirmadas.
   - Entregas: tasa de entrega y app del formulario.
   - Cuenta publicitaria, con su zona horaria.
   - Página.
   - Instagram, con arroba.
   - Píxel.
   - Número de WhatsApp, con "(vinculado a la página)" si `meta.whatsapp_vinculado` es `true`.
   - En cada activo de Meta, pon el ID entre paréntesis después del nombre. No muestres valores internos como `origen`.
2. Incluye una línea fija: "Siempre: catálogo apagado, mejoras automáticas apagadas y nada se publica sin tu aprobación."
3. Si hay `pendientes`, lístalos debajo.
4. Pregunta: "¿Guardo tu perfil así?" con las opciones "Guardar perfil" y "Corregir un dato".
   - Si el alumno ya pidió en su mensaje que guardes sin volver a preguntar, no hagas la pregunta: muestra el resumen y guarda.

## Paso 4. Guardar el perfil

1. Arma el JSON exactamente con el formato de `esquemas/perfil-negocio.schema.json`:
   - `version_esquema = 1`.
   - `creado` y `actualizado` en ISO 8601 con desfase de Lima, por ejemplo `2026-10-17T21:30:00-05:00`. Al actualizar, conserva `creado`.
   - Usa la fecha de hoy que ya conoces. Si no conoces la hora exacta, pon `12:00:00`. No uses comandos de terminal para obtenerla.
   - `reglas_fijas` siempre igual: `{"catalogo": "apagado", "mejoras_automaticas": "apagadas", "aprobacion_antes_de_publicar": true}`.
   - Todos los campos del esquema presentes. Lo que no aplica o no se sabe va en `null`.
2. Revisa el JSON contra el esquema antes de guardar: tipos, valores permitidos y formato del número de WhatsApp.
3. Guárdalo con tu herramienta de escritura de archivos en `zepol/perfil-negocio.json`, en UTF-8 y con sangría de 2 espacios. En esta skill no uses comandos de terminal para nada: así funciona igual en Windows, Mac y la app de Claude.
4. Si no puedes escribir archivos en esta conversación, muestra el JSON completo en un bloque de código y di: "No puedo guardar archivos aquí. Copia este bloque y guárdalo como perfil-negocio.json dentro de una carpeta llamada zepol. Las demás funciones de ZEPOL te lo pedirán."
5. TODO(servidor): enviar el perfil a Zepol Cloud (Supabase) y confirmar que quedó guardado allí.

## Paso 5. Cierre

- Confirma en una línea: "Perfil guardado en zepol/perfil-negocio.json."
- Si hay pendientes, recuérdalos en una lista corta.
- Termina con el siguiente paso: "Siguiente paso: corre /zepol-ads:diagnostico para que estudie tu cuenta antes de armar campañas."
