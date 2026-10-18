---
name: configurar
description: Crea o actualiza el perfil del negocio de ZEPOL ADS con una entrevista corta (tipo de negocio y experiencia, atención por WhatsApp, precio en los anuncios, pruebas sociales, tasa de entrega, app del formulario y activos de Meta) y lo guarda en Zepol Cloud. Úsala cuando el alumno escriba /zepol-ads:configurar, cuando empiece a usar ZEPOL o cuando otra skill de ZEPOL no encuentre el perfil.
---

# Configurar el perfil del negocio

Esta skill arma el perfil del negocio una sola vez. Las demás skills de ZEPOL lo leen desde Zepol Cloud y lo aplican en cada campaña sin volver a preguntar.

- El perfil se guarda en Zepol Cloud con la herramienta `guardar_perfil` y se lee con `obtener_perfil`. No se guarda ningún archivo local.
- Los datos por producto no se piden aquí: los pide `/zepol-ads:nueva-campana` para cada producto.

## Antes de empezar: conexión con Zepol Cloud

- Si no ves las herramientas de Zepol Cloud (`obtener_perfil`, `guardar_perfil`), ZEPOL no está conectado. Dile al alumno: "ZEPOL ADS no está conectado. En la app de Claude, entra a Configuración → Conectores y conecta ZEPOL ADS. En Claude Code, escribe /mcp y autentica zepol-cloud. Inicia sesión con la cuenta de tu licencia y vuelve a correr /zepol-ads:configurar." No sigas sin la conexión.
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.

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
- No inventes datos. Si el alumno no sabe algo, no lo envíes y agrégalo a la lista de pendientes.
- Nunca pidas contraseñas, claves de API ni tokens. Si el alumno los escribe, no los guardes y dile que no hacen falta aquí.
- Las pruebas sociales deben ser reales y demostrables. Si el alumno no lo confirma, no se envían.

## Paso 0. Revisar si ya hay un perfil

1. Llama `obtener_perfil` sin parámetros.
2. Si devuelve uno o más perfiles:
   - Con uno, muestra un resumen corto y pregunta: "Ya tienes un perfil guardado. ¿Qué quieres cambiar?" con las opciones "Actualizar un dato", "Rehacer el perfil completo" y "Dejarlo como está".
   - Con varios, pregunta primero con qué negocio trabajas y luego lo mismo.
   - Para actualizar, pregunta solo ese bloque y conserva el resto de lo guardado.
   - Para registrar otro negocio, sigue la entrevista completa con otro nombre.
3. Si no hay perfiles, saluda en una línea y empieza: "Te hago unas preguntas sobre tu negocio. Toma unos 5 minutos y solo se hace una vez." Si el alumno ya trajo sus respuestas, omite esta presentación.

## Paso 1. Revisar el conector de Meta

- Si ves herramientas del conector oficial de Meta (por ejemplo, `ads_get_ad_accounts`), úsalas en el bloque 6 para ofrecer opciones.
- Si no las ves y el alumno ya dijo que sigue a mano, no repitas el aviso y continúa.
- Si no las ves y no lo dijo, avisa y ofrece seguir a mano:
  - "No encuentro el conector de Meta. Conéctalo desde la sección Conectores de Claude (Meta Ads) con tu cuenta de Facebook y vuelve a correr /zepol-ads:configurar. Si prefieres, sigo ahora y te pido los datos a mano."
  - Opciones: "Seguir a mano" y "Conectar Meta primero".
- Si una llamada falla, explica el error con qué pasó + cómo arreglarlo y ofrece seguir a mano. No reintentes en bucle.

## Paso 2. Entrevista

Haz los bloques en este orden. Cada pregunta indica el dato que se guarda en Zepol Cloud (parámetro de `guardar_perfil`). Los datos marcados `extras.…` van dentro del parámetro `extras` (por ejemplo `extras.envios`).

### Bloque 1. Tu negocio

1. "¿Cómo se llama tu negocio?" → `nombre_negocio`.
2. "¿Por dónde vendes?" → `tipo_negocio`
   - "Solo por mi tienda Shopify" → `shopify`
   - "Solo por WhatsApp" → `whatsapp`
   - "Por los dos" → `hibrido`
3. "¿Cuánta experiencia tienes vendiendo por internet?" → `experiencia`
   - "Recién empiezo" → `principiante`
   - "Ya tengo experiencia" → `con_experiencia`
4. "¿A dónde haces envíos?" En sus palabras ("Todo el Perú", "Solo Lima y Callao") → `extras.envios`.
5. ZEPOL funciona solo para Perú. Si el alumno vende en otro país, dile: "Por ahora ZEPOL solo funciona para campañas en Perú." y sigue con Perú.

### Bloque 2. WhatsApp

Si `tipo_negocio` es `shopify`, no hagas este bloque.

1. "¿Quién responde los chats de WhatsApp?" → `whatsapp_atencion`
   - "Un bot" → `bot`
   - "A mano" → `manual`
   - "Las dos cosas" → `bot`
2. "¿Cuántas personas atienden los chats?" → `extras.personas_atencion` (número entero).
3. "¿Sientes que te falta personal para atender todos los chats?" → `whatsapp_poco_personal` ("Sí" → `true`, "No" → `false`).
4. "¿Con qué atiendes WhatsApp?" → `extras.herramienta_whatsapp`, en palabras simples.
   - "La app WhatsApp Business"
   - "Otra herramienta (un CRM o la API)": pregunta cuál.
   - "WhatsApp normal": agrega a pendientes "Pasar el número a WhatsApp Business".
5. Solo con la app WhatsApp Business: "¿Marcas tus chats con las etiquetas de pedido de WhatsApp Business (Nuevo pedido, Pago pendiente, Pagado, Pedido completado)?" → `whatsapp_etiqueta_chats`
   - "Sí, siempre" → `true`
   - "A veces" o "No" → `false`, y agrega a pendientes: "Etiquetar todos los chats de pedido en WhatsApp Business".
   - Con otra herramienta o WhatsApp normal, no envíes este dato.

### Bloque 3. Precio en los anuncios

1. "¿Quieres que tus anuncios muestren el precio?" → `muestra_precio` ("Sí, mostrar el precio" → `true`, "No mostrarlo" → `false`).
2. Aclara que es decisión de su negocio y que queda guardada para todas sus campañas.

### Bloque 4. Pruebas sociales

Introduce el bloque así: "Ahora unos datos que ZEPOL puede usar en tus anuncios. Usa solo datos reales que puedas demostrar."

1. "¿A cuántos clientes has atendido, más o menos?" → `prueba_social_clientes` (número entero; "No sé" → no lo envíes).
2. "¿Tienes una calificación pública? Dime cuánto, de cuánto y dónde, por ejemplo 4.8 de 5 en Google." → `prueba_social_calificacion`, en escala de 0 a 5. Si su escala es otra, conviértela (por ejemplo 9 de 10 → 4.5) y confírmalo. Dónde la tiene → `extras.calificacion_fuente`.
3. Si tiene calificación: "¿Cuántas reseñas tiene esa calificación?" → `prueba_social_resenas` (número entero; "No sé" → no lo envíes).
4. "¿Alguien reconocido recomienda tu producto o tu tienda? Por ejemplo, un especialista o un medio." → `prueba_social_recomendaciones` (lista de textos cortos en sus palabras).
5. Si dio al menos un dato: "¿Confirmas que estos datos son reales y puedes demostrarlos?"
   - Si dice que no, no envíes ninguna prueba social y explica: "No los voy a usar en tus anuncios. Cuando los tengas, corre /zepol-ads:configurar y los agregas."

### Bloque 5. Entregas

1. "De cada 100 pedidos contra entrega, ¿cuántos se entregan y se pagan?" → `tasa_entrega_pct` (número de 0 a 100).
   - Explica en una línea para qué sirve: "Con esto calculo cuánto te cuesta cada pedido que sí se entrega."
   - "No sé" → no lo envíes y agrega a pendientes: "Calcular la tasa de entrega contra entrega".
   - Si responde con una fracción o con pedidos totales (por ejemplo, "de 200 se entregan 110"), conviértelo a porcentaje y confírmalo.
2. Si `tipo_negocio` no es `whatsapp`: "¿Qué app usas para el formulario contra entrega de tu tienda?" → `app_formulario`
   - "Releasit" → `releasit`
   - "Otra" → pregunta cuál y envía su nombre en minúsculas.
   - "No uso formulario" → `ninguna`, y agrega a pendientes: "Instalar un formulario contra entrega en la tienda".

### Bloque 6. Activos de Meta

Con el conector, lee y ofrece opciones. A mano, pide cada dato y explica dónde encontrarlo en una línea.

En los dos casos, todos los IDs se envían solo con números: quita `act_` y cualquier otro texto.

1. **Cuenta publicitaria** → `meta_cuenta_publicitaria_id`.
   - Llama `ads_get_ad_accounts` y sigue `next_cursor` hasta tener todas.
   - Ofrece solo las cuentas con `is_ads_mcp_enabled = true`. Si alguna tiene `is_queryable = false`, `has_payment_method = false` o un `account_status` distinto de `ACTIVE`, dilo junto a su nombre, por ejemplo: "Cuenta 2 no tiene método de pago. Agrégalo en la configuración de pagos del Administrador de anuncios para poder usarla."
   - Si solo hay una, confírmala. Si hay varias, pregunta: "¿Con qué cuenta publicitaria vas a anunciar?"
   - A mano: "Abre el Administrador de anuncios: el número que aparece después de act= en la dirección es el ID de tu cuenta."
2. **Zona horaria** → `extras.zona_horaria_cuenta` (ZEPOL trabaja en hora de Lima).
   - Con el conector, verifica el campo `timezone_name` con `ads_get_field_context` y léelo con `ads_get_ad_entities` en `level = ad_account` para la cuenta elegida.
   - Si no es `America/Lima`, avisa: "Tu cuenta publicitaria no está en hora de Lima. Los horarios de lanzamiento se calculan en la hora de tu cuenta."
3. **Página** → `meta_pagina_id`.
   - Llama `ads_get_ad_account_pages` (páginas de la cuenta) y `ads_get_user_pages` (páginas donde el alumno puede anunciar). Cruza las dos listas.
   - Ofrece las páginas que están en las dos. Si hay más de una, pregunta: "¿Qué página usas para tus anuncios?"
   - Si la página que quiere no está en `ads_get_user_pages`, explica: "No tienes permiso para anunciar con la página {nombre}. Si eres administrador del portafolio, entra a Configuración del negocio, abre Cuentas y luego Páginas, elige la página y asígnate en Asignar personas. Avísame cuando lo hagas y lo reviso de nuevo." Vuelve a llamar ambas herramientas cuando avise.
4. **Instagram** → `meta_instagram_id`.
   - Llama `ads_get_ig_accounts` con la cuenta elegida. Si hay varias, pregunta cuál. Si hay una, confírmala.
   - Si no hay ninguna: "No encuentro una cuenta de Instagram vinculada a tu cuenta publicitaria. Vincúlala en Configuración del negocio, en Cuentas de Instagram, y vuelve a correr /zepol-ads:configurar." Agrégalo a pendientes.
5. **Píxel** → `meta_pixel_id`.
   - Llama `ads_get_datasets` con la cuenta elegida. Si hay varios, pregunta: "¿Qué píxel tiene instalado tu tienda?"
   - Si no hay ninguno y el negocio vende por Shopify: "No encuentro un píxel en esta cuenta. Créalo en el Administrador de eventos y conéctalo a tu tienda Shopify." Agrégalo a pendientes.
   - Aquí solo eliges el píxel. Si recibe eventos lo revisa `/zepol-ads:diagnostico`.
6. **Número de WhatsApp** → `whatsapp_numero`.
   - Pregúntalo si el negocio usa WhatsApp.
   - "¿Cuál es el número de WhatsApp Business conectado a tu página?"
   - El conector no lee el número de la página, así que lo escribe el alumno.
   - Quita espacios, guiones y el prefijo. Debe quedar `+51` seguido de 9 dígitos que empiezan con 9, por ejemplo `+51987654321`.
   - Si no cumple: "Ese número no parece un celular de Perú. Escríbelo con sus 9 dígitos, por ejemplo 987 654 321."
   - Pregunta: "¿Ese número ya está vinculado a tu página de Facebook?" → `extras.whatsapp_vinculado` ("Sí" → `true`, "No" o "No sé" → `false`). Si no es "Sí", agrega a pendientes: "Vincular el número de WhatsApp Business a la página". Los anuncios a WhatsApp no se pueden activar sin ese vínculo.

## Paso 3. Resumen y confirmación

1. Muestra una tabla con dos columnas, "Dato" y "Valor", en palabras simples (por ejemplo "Shopify y WhatsApp", no `hibrido`). Usa estas filas, juntando los datos de cada una:
   - Negocio: nombre, por dónde vende y experiencia.
   - Envíos.
   - WhatsApp: quién atiende, cuántas personas, si falta personal, herramienta y etiquetas. Omite la fila si no usa WhatsApp.
   - Precio en los anuncios.
   - Pruebas sociales: clientes, calificación con reseñas y fuente, y recomendaciones, o "No se usan" si no están confirmadas.
   - Entregas: tasa de entrega y app del formulario.
   - Cuenta publicitaria, con su zona horaria.
   - Página.
   - Instagram, con arroba.
   - Píxel.
   - Número de WhatsApp, con "(vinculado a la página)" si el alumno lo confirmó.
   - En cada activo de Meta, pon el ID entre paréntesis después del nombre.
2. Incluye una línea fija: "Siempre: catálogo apagado, mejoras automáticas apagadas y nada se publica sin tu aprobación."
3. Si hay pendientes, lístalos debajo.
4. Pregunta: "¿Guardo tu perfil así?" con las opciones "Guardar perfil" y "Corregir un dato".
   - Si el alumno ya pidió en su mensaje que guardes sin volver a preguntar, no hagas la pregunta: muestra el resumen y guarda.

## Paso 4. Guardar el perfil en Zepol Cloud

1. Llama `guardar_perfil` con `nombre_negocio`, `tipo_negocio`, `experiencia` y los demás datos que tengas, y en `extras` los datos marcados así y la lista de pendientes (`extras.pendientes`). No envíes los datos que el alumno no sabe.
2. El perfil se identifica por `nombre_negocio`: si ya existía con ese nombre, se actualiza y lo que no envíes se conserva. Los `extras` se combinan con los que ya estaban guardados.
3. Si `guardar_perfil` responde con un error, muéstralo con qué pasó + cómo arreglarlo y ofrece corregir el dato. No reintentes en bucle. Si el error no es de un dato del alumno, regístralo con `registrar_error` (`comando = configurar`, sin claves, tokens ni datos de clientes).
4. Confirma que quedó guardado con lo que devuelve la herramienta (el perfil con su `id`).

## Paso 5. Cierre

- Confirma en una línea: "Perfil de {nombre_negocio} guardado en ZEPOL."
- Si hay pendientes, recuérdalos en una lista corta.
- Termina con el siguiente paso: "Siguiente paso: corre /zepol-ads:diagnostico para que estudie tu cuenta antes de armar campañas."
