---
name: nueva-campana
description: Arma una campaña nueva de Meta para un producto con ZEPOL ADS, a WhatsApp o a Shopify (web contra entrega). Pide los datos del producto, trae la metodología de Zepol Cloud, crea todo como borrador con las reglas de armado seguro, guarda la receta, hace la revisión antes de publicar y publica solo con un sí explícito del alumno. Úsala cuando el alumno escriba /zepol-ads:nueva-campana o pida lanzar, crear o testear un producto en Meta.
---

# Nueva campaña

Arma la campaña de un producto. Todo se crea como borrador y nada se publica ni gasta sin el "sí" explícito del alumno.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Armo tu campaña"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Crear borrador", "Publicar campaña", "Cambiar un texto".
- Errores con la forma qué pasó + cómo arreglarlo.
- Excepción: los textos de los anuncios para Meta sí pueden llevar emojis. Salen de la metodología de Zepol Cloud.

## Metodología: siempre de Zepol Cloud

- La metodología de ZEPOL vive en Zepol Cloud, no en este plugin: estructura, objetivo, presupuestos, públicos, ubicaciones, nombres, horarios, configuración, creativos y textos.
- Pídela con `obtener_metodologia`, por etapa (`estructura`, `configuracion`, `creativos`, `textos`) y con el canal de la campaña (`whatsapp` o `shopify`).
- No improvises: no inventes estructuras, presupuestos, públicos, textos ni horarios con tu propio criterio. Si a las reglas les falta algo para tu caso, pregúntale al alumno.
- Si no ves las herramientas de Zepol Cloud (por ejemplo `obtener_perfil`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Configuración → Conectores y conecta ZEPOL ADS. En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:nueva-campana." y detente.
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.

## Paso 0. Antes de empezar

1. Llama `obtener_perfil`. Si no hay perfiles: "No encuentro tu perfil de negocio. Corre /zepol-ads:configurar primero; toma unos 5 minutos." Si hay varios, pregunta con qué negocio trabajas.
2. Confirma que el conector de Meta está disponible. Si no: "No encuentro el conector de Meta. Conéctalo desde la sección Conectores de Claude y vuelve a correr /zepol-ads:nueva-campana."
3. Revisa si el perfil trae un `diagnostico` (lo guarda /zepol-ads:diagnostico). Si no hay, ofrece: "Antes de armar la campaña conviene estudiar tu cuenta. ¿Corro el diagnóstico primero?" con las opciones "Correr diagnóstico" y "Seguir sin diagnóstico".
4. Carga la skill `armado-seguro`. Todo lo que se cree o edite en Meta sigue sus reglas, en su orden.

## Paso 1. Datos del producto

Pide los datos que guarda `guardar_producto`. Si el producto ya está en el perfil (`obtener_perfil` trae los productos), muéstralo y pregunta solo lo que cambió.

| Campo | Pregunta |
|---|---|
| `nombre` | "¿Qué producto vas a anunciar?" |
| `precio_venta` | "¿A cuánto lo vendes? Aunque no lo muestres en el anuncio, lo necesito para medir si la campaña es rentable." |
| `margen_bajo` | "¿Dirías que el margen de este producto es bajo?" |
| `creativos_disponibles` | "¿Qué creativos tienes de este producto: videos, imágenes, los dos o ninguno?" (`videos`, `imagenes`, `ambos` o `ninguno`). Luego pide cada uno: archivo, URL o ID de la publicación. |
| Fotos del producto | "¿Tienes fotos reales del producto? Pásame el enlace público de una o varias." Por ahora no se guardan en Zepol Cloud: úsalas al generar las imágenes. |
| `es_complejo` y `para_personas_mayores` | "¿El producto es difícil de explicar? ¿Es para personas mayores?" |
| `landing_url` | Solo si la campaña va a Shopify: "¿Cuál es el enlace del producto en tu tienda?" Debe empezar con https://. |

- No calcules el margen: cada negocio conoce sus números.
- Guarda el producto con `guardar_producto`, con el `perfil_id` del perfil. Si ya existía, envía su `id` para actualizarlo.

## Paso 2. Canal: WhatsApp o Shopify

- `tipo_negocio = shopify` → Shopify.
- `tipo_negocio = whatsapp` → WhatsApp.
- `tipo_negocio = hibrido` → pregunta: "¿A dónde quieres llevar a la gente con esta campaña?" con las opciones "A mi tienda Shopify" y "A mi WhatsApp".
  - Si la metodología (`obtener_metodologia` con `etapa = estructura`) recomienda un canal para este negocio y este producto, muéstralo antes de que el alumno elija. El alumno decide.

## Paso 3. Pedir la metodología

- Llama `obtener_metodologia` con el canal elegido para las etapas `estructura`, `configuracion`, `creativos` y `textos`.
- Con esas reglas, el perfil, el producto y el diagnóstico, arma la especificación completa de la campaña: objetivo, optimización, estructura, presupuestos, públicos, ubicaciones, nombres, recomendación de inicio y el plan de creativos y textos.
- Aplica solo lo que dicen las reglas. Si algo no está cubierto, pregúntale al alumno.

## Paso 4. Rama según el canal

### Rama Shopify (web contra entrega)

1. **Landing**: revisa que `landing_url` responda (HTTP 200) y que tenga el formulario contra entrega de la app del perfil (`app_formulario`).
   - Si no responde: "Tu landing no responde. Revisa que el producto esté publicado en Shopify y vuelve a intentarlo."
   - El resto de la revisión de la landing está en la metodología (`configuracion`, canal `shopify`).
2. **Píxel**: el conjunto usa el píxel del perfil (`meta_pixel_id`) con el evento de compra. Si el perfil no tiene píxel, detente: "Tu perfil no tiene píxel. Créalo en el Administrador de eventos, corre /zepol-ads:configurar para elegirlo y vuelve aquí."
3. **Enlaces**: cada anuncio lleva `landing_url` como destino y las UTM de la regla 6 de `armado-seguro`.
4. El botón sale de la metodología (`textos`).

### Rama WhatsApp

1. **Número**: la página del perfil necesita el número de WhatsApp Business (`whatsapp_numero`) vinculado y verificado. El perfil guarda el número, pero no confirma el vínculo. Si no lo está: "Tu número de WhatsApp no está vinculado a tu página. Vincúlalo en la configuración de WhatsApp de tu página y vuelve a intentarlo." Sin ese vínculo el borrador se crea, pero no se puede publicar.
2. **Destino**: el conjunto lleva `destination_type: WHATSAPP` y el botón es `WHATSAPP_MESSAGE`, sin enlace web ni UTM.
3. **Mensaje de bienvenida**: el texto sale de la metodología (`configuracion`, canal `whatsapp`). TODO(prueba): el conector no lo configura en el creativo; confirmar en la primera prueba cómo cargarlo.
4. TODO(prueba): la rama WhatsApp todavía no se probó de punta a punta con el conector. En la primera campaña, revisa cada paso con el alumno en el Administrador.

## Paso 5. Creativos

- **Ganadores que se reutilizan**: van por su publicación (`object_story_id` + `url_tags`), con la regla 5 de `armado-seguro`. Cuáles se reutilizan sale de la metodología (`creativos`) y del diagnóstico.
- **Imágenes nuevas**: genéralas con `generar_imagen` (`producto_id`, `temperatura`, `angulo`, los textos y la foto real del producto en `foto_producto_url`). Usa la clave de OpenAI que el alumno guardó en su panel. Zepol Cloud aloja cada imagen y devuelve una URL pública: súbela a la cuenta con `ads_creative_upload_media`.
  - Si `generar_imagen` dice que falta la clave de OpenAI, pide guardarla en "Mi cuenta" del panel de ZEPOL ADS. No la pidas en el chat.
- **Videos o imágenes del alumno**: necesitan una URL pública para subirse. Por ahora pídele al alumno un enlace público propio (por ejemplo, de su tienda). No uses alojamientos temporales públicos.
- **Textos**: escríbelos con la metodología (`textos`), que trae las estructuras y los bancos de frases. Muéstralos al alumno; si pide cambios, ajústalos respetando esas reglas.

## Paso 6. Propuesta y aprobación

1. Muestra todo en una sola vista:
   - tabla de la estructura (campaña, conjuntos, presupuesto por día, inicio en la hora de la cuenta);
   - cada anuncio con su texto, su título, su creativo y su destino;
   - presupuesto total por día.
2. Aclara: "Esto se crea como borrador en tu cuenta. No se publica ni gasta nada hasta que me digas que sí."
3. Pregunta: "¿Creo esta campaña como borrador en Meta?" con las opciones "Crear borrador" y "Cambiar algo".

## Paso 7. Crear el borrador

Sigue el orden de armado de `armado-seguro`, sin saltarte nada:

1. Candado del editor.
2. Campaña y, justo después, quitar el catálogo con `{"promoted_object":{}}`.
3. Conjuntos con inicio programado en la zona horaria de la cuenta.
4. Imágenes subidas por URL pública.
5. Anuncios: imagen con `link_data` en línea o ganadores por publicación, con las mejoras en `OPT_OUT` y las UTM.

Usa `hide_ui: true` cuando crees muchos objetos seguidos. Si una llamada falla, detente, explica el error con qué pasó + cómo arreglarlo y no recrees nada sin confirmar antes qué quedó creado.

## Paso 8. Guardar la receta

- Apenas se cree, guarda la receta en Zepol Cloud con `guardar_receta`: `perfil_id`, `producto_id`, `canal`, `nombre_campana`, `estado = borrador`, `meta_campaign_id` y en `receta` la especificación completa con los IDs de cada objeto.
- Al publicar, actualízala con su `id`, `estado = publicada`.
- La receta sirve para verificar lo publicado, reconstruir si algo se daña y relanzar la misma estructura con otro producto.
- Mientras Zepol Cloud no pueda leer recetas (llega en la próxima versión), guarda también una copia en `zepol/recetas/{slug-de-la-campana}.json` si puedes escribir archivos. La usan /zepol-ads:verificar y /zepol-ads:limpiar.

## Paso 9. Validar el borrador

Regla 8 de `armado-seguro`: leer con `object_state=draft`, todo `VALIDATED`, vistas previas y revisión de un anuncio de cada tipo en el Administrador (y después cerrar el editor).

## Paso 10. Revisión antes de publicar

Muestra una tabla con cada punto y su estado ("Listo" o qué falta):

| Punto | Qué se revisa |
|---|---|
| Validación | Todos los objetos en `VALIDATED`, sin errores |
| Destino | Shopify: píxel y evento de compra en cada conjunto. WhatsApp: número vinculado a la página |
| Presupuesto | Total por día de la campaña |
| Inicio | Fecha y hora en la zona horaria de la cuenta |
| Catálogo y mejoras | Catálogo vacío en la campaña y mejoras en `OPT_OUT` en cada anuncio |
| Enlaces | Shopify: cada URL responde y lleva sus UTM |
| Creativos | Cada video e imagen aparece en su vista previa |
| Vistas previas | Un enlace `preview_url` por anuncio |

Termina con: "Publica solo esta campaña, no otros borradores."

## Paso 11. Publicar

Regla 10 de `armado-seguro`. Pregunta: "¿Publico la campaña {nombre}? Empezará a gastar S/{total} por día desde el {fecha y hora}." con las opciones "Publicar campaña" y "Todavía no". Sin un "sí" explícito no llames `ads_activate_entity`.

## Paso 12. Verificar y cerrar

1. Haz la verificación después de publicar (regla 11 de `armado-seguro`, o `/zepol-ads:verificar`).
2. Confirma en una línea qué quedó publicado y desde cuándo.
3. En WhatsApp, recuerda: "Cada día dime cuántos pedidos confirmados, entregados y devueltos tuviste, y los registro para calcular tu costo por pedido entregado." Regístralos con `registrar_pedidos_whatsapp`.
4. Termina con: "Desde mañana, corre /zepol-ads:optimizar cada mañana. Para ver tus números, usa /zepol-ads:reporte."
