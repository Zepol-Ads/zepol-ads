---
name: verificar
description: Compara una campaña publicada en Meta contra su receta guardada por ZEPOL ADS (anuncios por conjunto, estados, presupuestos, hora de inicio, catálogo, mejoras, píxel y UTM) y ofrece recrear lo que falte, siempre con aprobación. Úsala cuando el alumno escriba /zepol-ads:verificar, justo después de publicar o cuando sospeche que algo cambió en una campaña.
---

# Verificar una campaña

Compara lo que está publicado en Meta contra la receta de la campaña. Solo lee; si falta algo, propone cómo arreglarlo y espera la aprobación del alumno.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tu campaña"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Recrear anuncio", "Dejarlo así".
- Errores con la forma qué pasó + cómo arreglarlo.

## Paso 0. Antes de empezar

1. Llama `obtener_perfil`. Si no hay perfiles, pide correr `/zepol-ads:configurar`. Si ZEPOL no está conectado o no hay licencia activa, explícalo como en `/zepol-ads:configurar` y detente.
2. Pregunta qué campaña verificar si no lo dijo. Por ahora las recetas se leen de la copia en `zepol/recetas/`; Zepol Cloud podrá leerlas en la próxima versión.
3. Si no hay receta de esa campaña: "No tengo la receta de esta campaña, así que no puedo compararla. Solo puedo verificar las campañas creadas con /zepol-ads:nueva-campana."
4. Carga la skill `armado-seguro`.

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

- Actualiza la receta con los IDs nuevos, si se recreó algo.
- Si se recreó algo, actualiza también la receta en Zepol Cloud con `guardar_receta` y su `id`, si lo tienes.
