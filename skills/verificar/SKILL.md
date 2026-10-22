---
name: verificar
description: Compara una campaña publicada en Meta contra su receta guardada en Zepol Cloud (anuncios por conjunto, estados, presupuestos, hora de inicio, catálogo, mejoras, píxel y UTM) y ofrece recrear lo que falte, siempre con aprobación. Si la campaña no tiene receta, ofrece adoptarla como receta. Úsala cuando el alumno escriba /zepol-ads:verificar, justo después de publicar, cuando sospeche que algo cambió en una campaña o cuando quiera que ZEPOL siga una campaña que creó por su cuenta.
---

# Verificar una campaña

Compara lo que está publicado en Meta contra la receta de la campaña. Solo lee; si falta algo, propone cómo arreglarlo y espera la aprobación del alumno.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tu campaña"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Recrear anuncio", "Dejarlo así".
- Errores con la forma qué pasó + cómo arreglarlo.

## Paso 0. Antes de empezar

1. Si ZEPOL no está conectado o no hay licencia activa, explícalo como en `/zepol-ads:empezar` y detente.
2. Llama `estado_cuenta` con `version_plugin = "0.6.1"`. Mira `onboarding.pasos`:
   - si falta el perfil o la cuenta publicitaria (`perfil` o `activos_meta` en `false`), di: "Antes de verificar falta tu arranque: escribe /zepol-ads:empezar. En unos minutos encuentra tus activos de Meta y estudia tu cuenta." y detente;
   - si solo falta el diagnóstico, sigue y, al terminar, recuerda: "Te falta el arranque: escribe /zepol-ads:empezar para que estudie tu cuenta."
3. Llama `obtener_perfil`. Si hay varios perfiles, pregunta con qué negocio trabajas.
4. Si el alumno no dijo qué campaña verificar, lista sus recetas con `listar_recetas` (estados `borrador`, `publicada` y `pausada`) y pregunta cuál.
5. Trae la receta completa con `obtener_receta`, por su `id` o por el ID de la campaña en Meta (`meta_campaign_id`).
6. Si no hay receta de esa campaña, dilo y ofrece adoptarla: "No tengo la receta de esta campaña, así que no puedo compararla. ¿La adopto como receta para que ZEPOL la siga desde hoy?" con las opciones "Adoptar campaña" y "No por ahora". Si acepta, sigue el "Paso 5. Adoptar una campaña como receta" y termina.
7. Carga la skill `armado-seguro`.
8. Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = verificar`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Paso 1. Leer lo publicado

- Lee la campaña, sus conjuntos y sus anuncios con `ads_get_ad_entities` (verifica los campos con `ads_get_field_context`). Pide solo los campos necesarios.
- Lee los creativos con `ads_get_creatives` para revisar las mejoras y las UTM.

## Paso 2. Comparar contra la receta

Regla 11 de `armado-seguro`. Arma una tabla con cada punto, lo esperado, lo publicado y el estado:

- número de anuncios por conjunto;
- estados (`effective_status`) de campaña, conjuntos y anuncios;
- presupuestos;
- hora de inicio, en la zona horaria de la cuenta;
- catálogo vacío en la campaña;
- píxel y evento de compra en cada conjunto (Shopify);
- mejoras en `OPT_OUT` en cada creativo, con la lista de la regla 3;
- UTM en cada anuncio (Shopify), con la regla 6.

## Paso 3. Proponer arreglos

- Si todo coincide: "Tu campaña quedó igual que su receta." y termina.
- Si falta algo o no coincide, dilo en concreto: "Falta el anuncio IMG 1 en el conjunto 3."
- Antes de proponer recrear, confirma que el objeto de verdad no existe (regla 9 de `armado-seguro`, con la vista previa).
- Pregunta: "¿Recreo lo que falta?" con las opciones "Recrear lo que falta" y "Dejarlo así".
- Con un "sí", recrea solo lo que falta siguiendo `armado-seguro` (candado del editor, mejoras en `OPT_OUT`, UTM, validación y publicación solo con otro "sí" explícito).
- Si el daño es grande (por ejemplo, anuncios sin creativo o conjuntos sin píxel), no parches: propone reconstruir la campaña desde la receta con `/zepol-ads:limpiar`.

## Paso 4. Cierre

- Si se recreó algo, actualiza la receta en Zepol Cloud con `guardar_receta`: su `id` y la `receta` completa con los IDs nuevos.
- Resume en una línea qué coincidió y qué se arregló.

## Paso 5. Adoptar una campaña como receta

Para las campañas que ya existen en Meta y no se crearon con ZEPOL. Solo lee Meta: no cambia nada en la campaña.

1. Lee la campaña, sus conjuntos, sus anuncios y sus creativos como en el paso 1.
2. Identifica el producto por el enlace de destino o por el nombre. Si no está en el perfil, pide sus datos y guárdalo con `guardar_producto`, como en el paso 1 de `/zepol-ads:nueva-campana`.
3. Canal: `destination_type` de WhatsApp o mensajes → `whatsapp`; sitio web → `shopify`. Si no está claro, pregunta.
4. Guarda la receta con `guardar_receta`, con el formato que trae la herramienta:
   - `origen = adoptada`;
   - el `estado` según Meta: `publicada` si la campaña está activa, `pausada` si está pausada y `borrador` si nunca se publicó;
   - lo que está publicado tal cual, con los IDs de Meta de cada objeto.
5. Si al leerla ves algo que no sigue `armado-seguro` (por ejemplo, el catálogo puesto o mejoras activas), dilo en una lista corta, pero no lo arregles sin un "sí" del alumno.
6. Termina con: "Desde ahora ZEPOL sigue esta campaña. Corre /zepol-ads:optimizar cada mañana y /zepol-ads:reporte para ver tus números."
