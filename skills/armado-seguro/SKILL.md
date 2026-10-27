---
name: armado-seguro
description: Reglas técnicas obligatorias de ZEPOL ADS para crear, editar, validar, publicar, verificar y limpiar objetos en Meta con el conector oficial. Cubre el candado del editor, quitar el catálogo, las mejoras automáticas en OPT_OUT, anuncios de imagen con link_data en línea, reutilizar ganadores por object_story_id, UTM, inicio programado, validación de borradores, limpieza con el prefijo BORRAR y publicación solo con un sí explícito. Cárgala siempre antes de llamar a una herramienta del conector de Meta que cree, edite o publique, y úsala como referencia en nueva-campana, verificar, optimizar y limpiar.
---

# Armado seguro en Meta

Estas reglas salen de la prueba real con el conector oficial de Meta (`mcp.facebook.com/ads`) del 16 de octubre de 2026. Son técnicas: no dependen de la metodología, se aplican siempre y no se saltan aunque el alumno lo pida.

La estructura, los presupuestos, los públicos, los textos, los creativos y los horarios no están aquí: los entrega el servidor de ZEPOL. Esta skill solo dice cómo armarlos en Meta sin romper nada.

## Orden de armado

1. Candado del editor (regla 1).
2. Crear la campaña y quitarle el catálogo (regla 2).
3. Crear los conjuntos con inicio programado (regla 7).
4. Subir las imágenes que el alumno aprobó, por URL pública, con `ads_creative_upload_media` (regla 12).
5. Crear los anuncios: imagen con `link_data` en línea (regla 4) o ganadores por publicación (regla 5), siempre con las mejoras en `OPT_OUT` (regla 3) y las UTM (regla 6).
6. Validar el borrador (regla 8).
7. Revisión con el alumno y publicación solo con un "sí" explícito (regla 10).
8. Verificación después de publicar (regla 11).
9. Limpieza de lo que sobró (regla 9).

## Reglas de oro

- Todo se crea como borrador. Nada se publica sin un "sí" explícito del alumno.
- Candado del editor antes de cada cambio.
- Valida antes de crear: el conector no puede borrar borradores, así que cada error deja basura.
- Nunca recrees algo sin confirmar antes que ya no existe.
- Respeta las políticas de publicidad de Meta. Si Meta rechaza un anuncio, no lo reenvíes con cambios mínimos.

## 1. Candado del editor

- **Antes** de crear o editar cualquier cosa por el conector, pide al alumno que cierre el editor del Administrador de anuncios y espera su confirmación:
  - "Antes de hacer cambios, cierra el editor del Administrador de anuncios si lo tienes abierto. Si queda abierto, puede guardar su versión encima y borrar los anuncios. Avísame cuando lo cierres."
- **Después** de editar, pide que refresque antes de abrirlo:
  - "Listo. Si vas a abrir la campaña en el Administrador, refresca la página primero."
- **Por qué**: el 16 de octubre de 2026 se cambiaron presupuestos por el conector con el editor abierto. Un minuto después, el editor guardó su propia versión del borrador: borró los creativos de todos los anuncios y el píxel de todos los conjuntos, y puso como inicio "ahora".
- **Si ya pasó**: no parches. Reconstruye desde cero con la receta de la campaña, porque los creativos no se pueden editar por el conector. Marca lo dañado con la regla 9.
- Cada vez que pidas al alumno abrir algo en el Administrador para revisarlo (reglas 3 y 8), recuérdale cerrarlo antes del siguiente cambio.

## 2. Campaña: quitar el catálogo

- Justo después de `ads_create_campaign` y **antes** de crear los conjuntos, llama `ads_update_entity` sobre la campaña:
  - `entity_type`: `campaign`
  - `fields`: `{"promoted_object":{}}`
- **Por qué**: al crear la campaña, el conector le asocia solo un catálogo de productos de la cuenta (`promoted_object.product_catalog_id`). Las campañas de ZEPOL nunca usan catálogo.
- **Verifica** en el borrador (regla 8): el `promoted_object` de la campaña debe quedar vacío (`[]`). Si no, repite una vez. Si sigue, detente y avisa al alumno.
- No lo confundas con el `promoted_object` del conjunto (por ejemplo, el píxel y el evento de compra en Shopify): ese se configura normal.

## 3. Mejoras automáticas: todas en OPT_OUT

Cada creativo, y cada creativo en línea dentro de `ads_create_ad`, lleva `degrees_of_freedom_spec.creative_features_spec` con cada función en `{"enroll_status":"OPT_OUT"}`. Nunca envíes `advantage_plus_creative: true` ni `advantage_plus_creative_features`.

Estos nombres se probaron contra Meta el 16 de octubre de 2026.

### Lista común (todos los anuncios, 17)

`product_browsing`, `product_extensions`, `standard_enhancements_catalog`, `site_extensions`, `show_summary`, `reveal_details_over_time`, `show_destination_blurbs`, `catalog_feed_tag`, `inline_comment`, `image_templates`, `enhance_cta`, `description_automation`, `media_type_automation`, `text_optimizations`, `product_metadata_automation`, `profile_card`, `text_overlay_translation`.

- `product_browsing` es la exploración de productos.
- `product_extensions` es el origen Productos, el más importante de apagar. Meta lo guarda además como `creative_sourcing_spec.catalog: OPT_OUT`; es normal.
- `standard_enhancements_catalog` probablemente corresponde a Secuencia y Colección.

### Anuncio de imagen (lista común + 6 = 23)

Además de la lista común: `image_animation`, `image_touchups`, `image_uncrop`, `image_brightness_and_contrast`, `adapt_to_placement`, `add_text_overlay`.

```json
{
  "creative_features_spec": {
    "product_browsing": {"enroll_status": "OPT_OUT"},
    "product_extensions": {"enroll_status": "OPT_OUT"},
    "standard_enhancements_catalog": {"enroll_status": "OPT_OUT"},
    "site_extensions": {"enroll_status": "OPT_OUT"},
    "show_summary": {"enroll_status": "OPT_OUT"},
    "reveal_details_over_time": {"enroll_status": "OPT_OUT"},
    "show_destination_blurbs": {"enroll_status": "OPT_OUT"},
    "catalog_feed_tag": {"enroll_status": "OPT_OUT"},
    "inline_comment": {"enroll_status": "OPT_OUT"},
    "image_templates": {"enroll_status": "OPT_OUT"},
    "enhance_cta": {"enroll_status": "OPT_OUT"},
    "description_automation": {"enroll_status": "OPT_OUT"},
    "media_type_automation": {"enroll_status": "OPT_OUT"},
    "text_optimizations": {"enroll_status": "OPT_OUT"},
    "product_metadata_automation": {"enroll_status": "OPT_OUT"},
    "profile_card": {"enroll_status": "OPT_OUT"},
    "text_overlay_translation": {"enroll_status": "OPT_OUT"},
    "image_animation": {"enroll_status": "OPT_OUT"},
    "image_touchups": {"enroll_status": "OPT_OUT"},
    "image_uncrop": {"enroll_status": "OPT_OUT"},
    "image_brightness_and_contrast": {"enroll_status": "OPT_OUT"},
    "adapt_to_placement": {"enroll_status": "OPT_OUT"},
    "add_text_overlay": {"enroll_status": "OPT_OUT"}
  }
}
```

### Video que reutiliza una publicación existente (lista común + 2 = 19)

Además de la lista común: `video_auto_crop`, `video_filtering`. El `degrees_of_freedom_spec` se acepta junto con `object_story_id`.

```json
{
  "creative_features_spec": {
    "product_browsing": {"enroll_status": "OPT_OUT"},
    "product_extensions": {"enroll_status": "OPT_OUT"},
    "standard_enhancements_catalog": {"enroll_status": "OPT_OUT"},
    "site_extensions": {"enroll_status": "OPT_OUT"},
    "show_summary": {"enroll_status": "OPT_OUT"},
    "reveal_details_over_time": {"enroll_status": "OPT_OUT"},
    "show_destination_blurbs": {"enroll_status": "OPT_OUT"},
    "catalog_feed_tag": {"enroll_status": "OPT_OUT"},
    "inline_comment": {"enroll_status": "OPT_OUT"},
    "image_templates": {"enroll_status": "OPT_OUT"},
    "enhance_cta": {"enroll_status": "OPT_OUT"},
    "description_automation": {"enroll_status": "OPT_OUT"},
    "media_type_automation": {"enroll_status": "OPT_OUT"},
    "text_optimizations": {"enroll_status": "OPT_OUT"},
    "product_metadata_automation": {"enroll_status": "OPT_OUT"},
    "profile_card": {"enroll_status": "OPT_OUT"},
    "text_overlay_translation": {"enroll_status": "OPT_OUT"},
    "video_auto_crop": {"enroll_status": "OPT_OUT"},
    "video_filtering": {"enroll_status": "OPT_OUT"}
  }
}
```

### No enviar nunca

- `website_highlights`, `show_spotlight` y `related_media`: Meta los rechazó, no existen.
- `standard_enhancements` (distinto de `standard_enhancements_catalog`): según el conector, la creación lo rechaza aunque vaya en `OPT_OUT`.
- `cv_transformation`: según el conector, solo existe en el Administrador.

### Detalles del conector

- En `ads_create_creative`, `degrees_of_freedom_spec` va como texto JSON, no como objeto. En `ads_create_ad`, todo el `creative` va como texto JSON.
- `contextual_multi_ads.enroll_status`: el conector lo deja en `OPT_OUT` en los creativos nuevos en línea. No lo cambies.
- TODO(prueba): video nuevo subido a la cuenta, que no es una publicación existente. La lista no está probada. Usa la lista común + las 2 de video, valida el borrador y confírmalo en el panel "Mejoras".

### Pendiente: "Lo más destacado del sitio web"

No hay certeza de qué nombre de la API apaga "Lo más destacado del sitio web". Con el primer anuncio de cada cuenta:

1. Pide al alumno abrir el panel "Mejoras" del anuncio en el Administrador.
2. Si "Lo más destacado del sitio web" u otro origen sigue activado, que lo apague ahí una vez. Meta guarda esa preferencia para todos los anuncios futuros de la cuenta.
3. Recuérdale cerrar el editor antes del siguiente cambio (regla 1).

## 4. Anuncios de imagen: link_data en línea, nunca PAC

- Crea el anuncio de imagen con un creativo en línea en `ads_create_ad`: `object_story_spec` con `page_id`, `instagram_user_id` (si el perfil tiene Instagram) y `link_data`.
- **Una sola imagen por anuncio** (`image_hash` de la imagen ya subida a la cuenta).
- Título en `link_data.name`, descripción en `link_data.description`, texto principal en `link_data.message` y enlace visible en `link_data.caption`.
- El botón (`link_data.call_to_action`) lleva solo `type` y `value.link`. **Nunca** `value.link_title` ni `value.link_description`.
- **Nunca PAC por el conector**: nada de `placement_customizations` ni de varios formatos por ubicación (por ejemplo, 4:5 + 9:16).
  - **Por qué**: el conector arma esos anuncios con el título y la descripción dentro del botón, un formato que Meta ya no acepta. La API los dio por `VALIDATED`, pero el Administrador los marcó con el error `#1815589`. Además, Meta agregó una descripción que nadie envió.
- Formato del `creative` (el texto, la imagen y el botón los define el servidor):

```json
{
  "object_story_spec": {
    "page_id": "{PAGE_ID}",
    "instagram_user_id": "{INSTAGRAM_ID}",
    "link_data": {
      "link": "{URL_DESTINO}",
      "image_hash": "{IMAGE_HASH}",
      "message": "{TEXTO_PRINCIPAL}",
      "name": "{TITULO}",
      "description": "{DESCRIPCION}",
      "caption": "{DOMINIO_VISIBLE}",
      "call_to_action": {
        "type": "{TIPO_DE_BOTON}",
        "value": {"link": "{URL_DESTINO}"}
      }
    }
  },
  "url_tags": "{UTM}",
  "degrees_of_freedom_spec": {"creative_features_spec": {"...": "lista de imagen de la regla 3"}}
}
```

- Declaración de IA (`self_ai_disclosure`): no la decidas tú ni se la preguntes al alumno. Ya está decidida en la metodología (`obtener_metodologia`, etapa `configuracion`, entre sus ajustes fijos): aplica ese valor en cada creativo tal cual. Solo si la metodología no lo trae, pregúntaselo al alumno. No se puede cambiar después de crear el creativo.
- TODO(prueba): anuncios de imagen a WhatsApp. El botón es `WHATSAPP_MESSAGE` y no lleva enlace web (ver regla 12). Confirmar en la primera prueba si se arma en línea igual que este formato o con `ads_create_creative`.

## 5. Reutilizar ganadores: object_story_id + url_tags

- Para relanzar un anuncio que ya funcionó, crea el anuncio con su **publicación**, no con su `creative_id`:

```json
{
  "object_story_id": "{PAGE_ID}_{POST_ID}",
  "url_tags": "{UTM}",
  "degrees_of_freedom_spec": {"creative_features_spec": {"...": "lista de video de la regla 3"}}
}
```

- **Por qué**: conserva las reacciones y los comentarios, y evita el error `ObjectStorySpecRedundant` de los creativos antiguos.
- El ID de la publicación (`{PAGE_ID}_{POST_ID}`) sale de `/zepol-ads:diagnostico`.
- El texto y el enlace de una publicación existente no se pueden cambiar. Las UTM van en `url_tags`.
- No combines `object_story_id` con `image_hash`, `video_id`, `link_url`, `message`, `headline`, `description` ni `call_to_action_type`.
- Antes de usarla, confirma que la publicación sigue existiendo y es de la misma página. Si no, dilo y no reintentes con el mismo ID.

## 6. UTM: utm_content por anuncio y utm_term por conjunto

- Todo anuncio con destino web lleva las UTM en `url_tags` del creativo:
  - Fijas: `utm_source=facebook`, `utm_medium=paid` y `utm_campaign={slug de la campaña}`.
  - `utm_content` = slug del **anuncio**. Cambia en cada anuncio.
  - `utm_term` = slug del **conjunto**. Cambia en cada conjunto.
- Así Shopify dice qué anuncio y qué conjunto trajo cada pedido.
- Slug: minúsculas, sin tildes, la ñ como n, espacios y símbolos como guiones, sin guiones repetidos. Ejemplo: "Video 1 - Oferta" → `video-1-oferta`.
- Ejemplo: `utm_source=facebook&utm_medium=paid&utm_campaign=test-producto&utm_content=video-1-oferta&utm_term=conjunto-1`.
- `url_tags` vive en el creativo. Si el mismo anuncio va en varios conjuntos, cada copia lleva su propio creativo con su `utm_term`.
- Los nombres de campaña, conjuntos y anuncios los define el servidor. Los slugs salen de esos nombres.
- Las campañas a WhatsApp no tienen destino web: no llevan UTM.
- Antes de publicar, confirma que la URL de destino responde (HTTP 200).

## 7. Inicio programado en la zona horaria de la cuenta

- Lee la zona horaria de la cuenta: `zona_horaria` del perfil o `timezone_name` en `level = ad_account` con `ads_get_ad_entities`.
- La fecha y la hora de inicio las elige el alumno. La recomendación de cuándo lanzar sale de la metodología (etapa `estructura`).
- Envía la hora en ISO 8601 con el desfase de esa zona. Perú (`America/Lima`) es `-05:00` todo el año, sin horario de verano. Ejemplo: `2026-10-23T08:00:00-05:00`.
- Pon `start_time` en cada conjunto. Nunca dejes el inicio en "ahora" por omisión.
- Verifícalo en el borrador (regla 8) y después de publicar (regla 11): el editor abierto puede cambiarlo a "ahora".

## 8. Validar el borrador con object_state=draft

- **Antes de crear**: confirma que tienes todo (IDs, imágenes subidas, URL que responden, publicaciones que existen). Así no queda basura.
- **Después de crear**: llama `ads_get_ad_entities` con `object_state=draft` y la campaña en `object_ids`.
  - Los resultados llegan en `ad_drafts` y el ID de cada objeto está en `ad_object_id`.
  - Propón publicar solo si todo está `VALIDATED` y sin `active_errors`.
  - Si la llamada con `object_state=draft` falla (la función todavía se está habilitando por cuenta), no reintentes: avisa y pide revisar en el Administrador.
- **La validación de la API no basta**:
  - Pide la vista previa de cada anuncio con `ads_get_ad_preview` y comparte el `preview_url`.
  - Pide al alumno abrir en el Administrador un anuncio de cada tipo y confirmar que no aparece ningún error, como el `#1815589`.
  - Después, que cierre el editor (regla 1).

## 9. Limpieza: prefijo "BORRAR -"

- El conector **no puede borrar borradores**: `status=DELETED` sobre un objeto que nunca se publicó queda pendiente y Meta lo rechaza. No lo intentes.
- Renombra lo obsoleto con `ads_update_entity` y `{"name":"BORRAR - {nombre actual}"}`, con el candado del editor (regla 1).
- Entrega la lista exacta, con nombre e ID de cada objeto, y guía al alumno para descartarlos en el Administrador: buscar "BORRAR -", seleccionar solo esos objetos y descartarlos.
- **Advertencia obligatoria**: "No uses el botón 'Descartar borradores': puede descartar también los borradores que sí sirven, no solo los marcados con BORRAR -."
- La lista de borradores del conector puede quedar desactualizada después de descartar en el Administrador. Para confirmar si un objeto sigue existiendo, pide su vista previa con `ads_get_ad_preview`: si responde "no encontrado", ya no existe.
- Nunca recrees algo sin confirmar antes que no existe, para no duplicar.

## 10. Publicar: nunca sin un "sí" explícito

- **Nunca llames `ads_activate_entity` sin un "sí" explícito del alumno** a la pregunta concreta de publicar, en ese momento.
  - No cuentan: un "ok" a otra pregunta, una aprobación anterior del plan, la aprobación de otra campaña ni el silencio.
  - Cada publicación necesita su propio "sí".
- Antes de preguntar, muestra la revisión antes de publicar (ver `/zepol-ads:nueva-campana`).
- Pregunta así: "¿Publico la campaña {nombre}? Empezará a gastar S/{total} por día desde el {fecha y hora}." Opciones: "Publicar campaña" y "Todavía no".
- **Publica solo esta campaña**: arma `object_ids` con los `ad_object_id` de esta campaña (campaña, conjuntos y anuncios) leídos con `object_state=draft`. Nunca incluyas otros borradores.
- No uses `ignore_validation_errors=true`, salvo que el alumno haya visto los errores y lo pida.
- Si el alumno publica desde el Administrador: "Revisar y publicar" publica **todos** los borradores pendientes. Recuérdale: "Publica solo esta campaña, no otros borradores."
- `PUBLISHING` no significa publicado. Confirma solo con la lectura de los objetos publicados (regla 11).
- **Cambios en objetos ya publicados** (por ejemplo, desde `/zepol-ads:optimizar`):
  - Pausar: `ads_update_entity` con solo `{"status":"PAUSED"}`. Cambia el objeto publicado al instante (`is_draft=false`). No llames `ads_activate_entity` después, porque puede deshacer la pausa.
  - Otros cambios, como un presupuesto, quedan en borrador (`is_draft=true`). Se publican con `ads_activate_entity`, con solo ese objeto en `object_ids` y `publish_as_active=false`, y solo con el "sí" del alumno.
  - Los presupuestos van en céntimos: multiplica los soles por 100 (S/1 es `100`).

## 11. Verificación después de publicar

Justo después de publicar, lee los objetos publicados con `ads_get_ad_entities` y compáralos con la receta de la campaña:

- número de anuncios por conjunto;
- estados (`effective_status`);
- presupuestos;
- hora de inicio;
- además: catálogo vacío en la campaña, píxel y evento en cada conjunto (Shopify), mejoras en `OPT_OUT` (con `ads_get_creatives`) y UTM en cada anuncio.

Si falta algo o no coincide:

- Muéstralo en concreto, por ejemplo: "Falta el anuncio IMG 1 en el conjunto 3."
- Ofrece recrear solo lo que falta, con estas mismas reglas y con aprobación. Antes, confirma que de verdad no existe (regla 9).
- En la prueba real, esta verificación detectó un anuncio de imagen que faltaba en uno de los conjuntos.

## 12. Otros datos técnicos del conector

- Pide solo los campos que necesitas en `ads_get_ad_entities`: las respuestas son grandes y consumen el plan de Claude del alumno.
- Las imágenes se suben por URL pública con `ads_creative_upload_media`, con estos datos exactos:

  ```json
  {
    "ad_account_id": "{ID DE LA CUENTA, SIN act_}",
    "upload_source": "URL",
    "media_type": "IMAGE",
    "media_url": "{LINK DE LA IMAGEN}"
  }
  ```

  - Las de `generar_imagen` ya vienen alojadas en Zepol Cloud: `media_url` es `creativo.url` (la respuesta lo trae listo en `subir_a_meta`; si se cortó, búscala con `ver_creativos`). Súbela solo después de que el alumno la vea y la apruebe.
  - De la respuesta toma el `image_hash` para el anuncio (regla 4).
  - Las imágenes de ZEPOL pueden venir en WebP. TODO(prueba): confirmar en la primera prueba real que Meta las acepta; si no, ZEPOL las pasa a JPG desde el servidor. Si la subida falla por el formato, regístralo con `registrar_error` y no la conviertas tú.
  - No uses alojamientos temporales públicos.
- WhatsApp. TODO(prueba): todavía sin probar de punta a punta.
  - El conjunto lleva `destination_type: WHATSAPP` y el botón es `WHATSAPP_MESSAGE`, sin enlace web.
  - La página necesita un número de WhatsApp Business verificado y vinculado. Sin eso, el borrador se crea, pero la publicación falla.
  - El conector no configura el mensaje de bienvenida en el creativo: aplica el saludo predeterminado de la página. TODO(prueba): confirmar cómo cargar el mensaje de bienvenida.
- Permisos en Claude Code: un plugin no puede traer reglas de permiso. TODO: definir en la guía de instalación qué herramientas de lectura se aprueban de una vez. Publicar siempre pide el "sí" del alumno.
