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

1. Llama `obtener_perfil`. Si no hay perfiles, pide correr `/zepol-ads:configurar`. Si ZEPOL no está conectado o no hay licencia activa, explícalo como en `/zepol-ads:configurar` y detente.
2. Carga la skill `armado-seguro`. Aquí aplican sobre todo sus reglas 1 (candado del editor) y 9 (limpieza).

## Paso 1. Encontrar lo que sobra

1. Lee los borradores con `ads_get_ad_entities` y `object_state=draft`.
2. Cruza con las recetas guardadas. Por ahora se leen de la copia en `zepol/recetas/`; Zepol Cloud podrá leerlas en la próxima versión.
3. Son candidatos a limpiar los objetos que no están en ninguna receta vigente o que quedaron dañados (por ejemplo, anuncios sin creativo o conjuntos sin píxel).
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

- Si una campaña quedó dañada (por ejemplo, porque el editor del Administrador guardó encima), no parches: reconstrúyela desde cero con su receta.
- Antes, confirma qué objetos existen todavía, para no duplicar nada.
- Muestra qué se va a crear y pregunta: "¿Reconstruyo la campaña {nombre} desde su receta?" con las opciones "Reconstruir campaña" y "Todavía no".
- Con un "sí", créala como borrador siguiendo `armado-seguro` en su orden completo: validación, revisión antes de publicar y publicación solo con otro "sí" explícito.
- Marca la versión dañada con "BORRAR -" (pasos 2 y 3).
- Guarda la campaña reconstruida como receta nueva con `guardar_receta`.

## Paso 5. Cierre

- Resume qué quedó marcado, qué ya se descartó y qué se reconstruyó.
- Actualiza la receta con los IDs nuevos.
