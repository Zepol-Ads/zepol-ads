---
name: limpiar
description: Limpia los borradores obsoletos de Meta que dejó una campaña de ZEPOL ADS (el conector no puede borrarlos), marcándolos con el prefijo BORRAR y guiando al alumno para descartarlos en el Administrador sin tocar los que sirven. También reconstruye una campaña dañada desde su receta, con aprobación. Úsala cuando el alumno escriba /zepol-ads:limpiar, cuando sobren borradores o cuando una campaña quedó dañada.
---

# Limpiar borradores y reconstruir

El conector de Meta no puede borrar borradores. Esta skill marca lo que sobra para que el alumno lo descarte en el Administrador, y reconstruye desde la receta lo que se dañó.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Marco los borradores"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Marcar borradores", "Reconstruir campaña".
- Errores con la forma qué pasó + cómo arreglarlo.

## Paso 0. Antes de empezar

1. Si ZEPOL no está conectado o no hay licencia activa, explícalo como en `/zepol-ads:empezar` y detente.
2. Llama `estado_cuenta` con `version_plugin = "0.6.2"`. Mira `onboarding.pasos`:
   - si falta el perfil o la cuenta publicitaria (`perfil` o `activos_meta` en `false`), di: "Antes de limpiar falta tu arranque: escribe /zepol-ads:empezar. En unos minutos encuentra tus activos de Meta y estudia tu cuenta." y detente;
   - si solo falta el diagnóstico, sigue y, al terminar, recuerda: "Te falta el arranque: escribe /zepol-ads:empezar para que estudie tu cuenta."
3. Llama `obtener_perfil`. Si hay varios perfiles, pregunta con qué negocio trabajas.
4. Carga la skill `armado-seguro`. Aquí aplican sobre todo sus reglas 1 (candado del editor) y 9 (limpieza).
5. Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = limpiar`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Paso 1. Encontrar lo que sobra

1. Lee los borradores con `ads_get_ad_entities` y `object_state=draft`.
2. Cruza con las recetas guardadas en Zepol Cloud: `listar_recetas` con los estados `borrador`, `publicada` y `pausada` e `incluir_ids_meta = true` trae los IDs de Meta de cada receta vigente.
3. Son candidatos a limpiar los objetos que no están en ninguna receta vigente o que quedaron dañados (por ejemplo, anuncios sin creativo o conjuntos sin píxel). Para revisar si un objeto está dañado, compáralo con su receta completa (`obtener_receta`).
4. La lista del conector puede estar desactualizada: confirma cada candidato con su vista previa (`ads_get_ad_preview`). Si responde "no encontrado", ya no existe y no va en la lista.
5. Muestra la lista con nombre, ID, tipo y motivo. Pregunta: "¿Marco estos borradores con BORRAR -?" con las opciones "Marcar borradores" y "No marcar nada".

## Paso 2. Marcar

1. Candado del editor (regla 1).
2. Con el "sí" del alumno, renombra cada objeto con `ads_update_entity` y `{"name":"BORRAR - {nombre actual}"}`.
3. Nunca uses `status=DELETED` sobre un borrador: Meta lo rechaza.

## Paso 3. Guiar el descarte en el Administrador

Da la lista exacta y estos pasos:

1. "Refresca el Administrador de anuncios."
2. "Busca BORRAR - en el buscador."
3. "Selecciona solo esos objetos y descártalos."
4. Advertencia obligatoria: "No uses el botón 'Descartar borradores': puede descartar también los borradores que sí sirven, no solo los marcados con BORRAR -."
5. Cuando avise que terminó, confirma cada objeto con su vista previa. Si alguno sigue existiendo, dile cuál.

## Paso 4. Reconstruir desde la receta (si hace falta)

- Si una campaña quedó dañada (por ejemplo, porque el editor del Administrador guardó encima), no parches: reconstrúyela desde cero con su receta (`obtener_receta`).
- Antes, confirma qué objetos existen todavía, para no duplicar nada.
- Muestra qué se va a crear y pregunta: "¿Reconstruyo la campaña {nombre} desde su receta?" con las opciones "Reconstruir campaña" y "Todavía no".
- Con un "sí", créala como borrador siguiendo `armado-seguro` en su orden completo: validación, revisión antes de publicar y publicación solo con otro "sí" explícito.
- Marca la versión dañada con "BORRAR -" (pasos 2 y 3).
- Guarda la campaña reconstruida como receta nueva con `guardar_receta`: `origen = reconstruida`, `reconstruida_de` con el `id` de la receta anterior y los IDs nuevos de Meta.
- Pasa la receta anterior a `estado = archivada` con `guardar_receta` y su `id`.

## Paso 5. Cierre

- Resume qué quedó marcado, qué ya se descartó y qué se reconstruyó.
- Confirma que la receta nueva quedó guardada con los IDs nuevos.
