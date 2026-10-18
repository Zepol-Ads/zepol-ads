---
name: nueva-campana
description: Arma una campaña nueva de Meta para un producto con ZEPOL ADS, a WhatsApp o a Shopify (web contra entrega). Pide los datos del producto, trae la metodología del servidor de ZEPOL, crea todo como borrador con las reglas de armado seguro, guarda la receta, hace la revisión antes de publicar y publica solo con un sí explícito del alumno. Úsala cuando el alumno escriba /zepol-ads:nueva-campana o pida lanzar, crear o testear un producto en Meta.
---

# Nueva campaña

Arma la campaña de un producto. Todo se crea como borrador y nada se publica ni gasta sin el "sí" explícito del alumno.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Armo tu campaña"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Crear borrador", "Publicar campaña", "Cambiar un texto".
- Errores con la forma qué pasó + cómo arreglarlo.
- Excepción: los textos de los anuncios para Meta sí pueden llevar emojis. Los define el servidor de ZEPOL.

## Metodología: siempre del servidor

- La metodología de ZEPOL vive en el servidor de ZEPOL, no en este plugin: estructura, objetivo, presupuestos, públicos, ubicaciones, nombres, horarios, creativos y textos.
- Pídela con la herramienta `metodologia` del servidor de ZEPOL. TODO(servidor): la herramienta aún no existe; nombre y parámetros por confirmar.
- Si el servidor no está conectado (no ves sus herramientas), no improvises: no inventes estructuras, presupuestos, públicos, textos ni horarios con tu propio criterio. Di: "El servidor de ZEPOL aún no está conectado, así que todavía no puedo armar la campaña. Ya guardé los datos de tu producto. Cuando esté listo, vuelve a correr /zepol-ads:nueva-campana." y detente.

## Paso 0. Antes de empezar

1. Lee el perfil `zepol/perfil-negocio.json`. Si no existe: "No encuentro tu perfil de negocio. Corre /zepol-ads:configurar primero; toma unos 5 minutos."
2. Confirma que el conector de Meta está disponible. Si no: "No encuentro el conector de Meta. Conéctalo desde la sección Conectores de Claude y vuelve a correr /zepol-ads:nueva-campana."
3. Busca un diagnóstico de la cuenta (`zepol/diagnostico-{cuenta_publicitaria_id}.json`). Si no hay, ofrece: "Antes de armar la campaña conviene estudiar tu cuenta. ¿Corro el diagnóstico primero?" con las opciones "Correr diagnóstico" y "Seguir sin diagnóstico".
4. Carga la skill `armado-seguro`. Todo lo que se cree o edite en Meta sigue sus reglas, en su orden.

## Paso 1. Datos del producto

Pide los datos con el formato de `producto.schema.json` de la skill `configurar` (`../configurar/esquemas/producto.schema.json` desde la carpeta de esta skill). Si ya existe `zepol/productos/{slug}.json`, muéstralo y pregunta solo lo que cambió.

| Campo | Pregunta |
|---|---|
| `nombre` | "¿Qué producto vas a anunciar?" |
| `precio_venta` | "¿A cuánto lo vendes? Aunque no lo muestres en el anuncio, lo necesito para medir si la campaña es rentable." |
| `margen_bajo` | "¿Dirías que el margen de este producto es bajo?" |
| `creativos_disponibles` | "¿Qué creativos tienes de este producto: videos, imágenes, los dos o ninguno?" Luego pide cada uno: archivo, URL o ID de la publicación. |
| `fotos_producto` | "¿Tienes fotos reales del producto? Pásame una o varias." |
| `complejo_o_personas_mayores` | "¿El producto es difícil de explicar o es para personas mayores?" |
| `landing_url` | Solo si la campaña va a Shopify: "¿Cuál es el enlace del producto en tu tienda?" Debe empezar con https://. |

- No calcules el margen: cada negocio conoce sus números.
- Guarda el producto en `zepol/productos/{slug}.json` con tu herramienta de escritura de archivos. TODO(servidor): guardarlo en Zepol Cloud.

## Paso 2. Canal: WhatsApp o Shopify

- `negocio.tipo = solo_shopify` → Shopify.
- `negocio.tipo = solo_whatsapp` → WhatsApp.
- `negocio.tipo = hibrido` → pregunta: "¿A dónde quieres llevar a la gente con esta campaña?" con las opciones "A mi tienda Shopify" y "A mi WhatsApp".
  - TODO(servidor): pedir a `metodologia` la recomendación de canal para este negocio y este producto, y mostrarla antes de que el alumno elija. El alumno decide.

## Paso 3. Pedir la metodología

- TODO(servidor): llamar a `metodologia` con la etapa `nueva_campana`, el canal, el perfil, el producto y el diagnóstico.
- Debe devolver la especificación completa de la campaña: objetivo, optimización, estructura, presupuestos, públicos, ubicaciones, nombres, recomendación de inicio y el plan de creativos y textos.
- Si no hay servidor, detente con el mensaje de la sección "Metodología: siempre del servidor".

## Paso 4. Rama según el canal

### Rama Shopify (web contra entrega)

1. **Landing**: revisa que `landing_url` responda (HTTP 200) y que tenga el formulario contra entrega de la app del perfil (`entregas.app_formulario`).
   - Si no responde: "Tu landing no responde. Revisa que el producto esté publicado en Shopify y vuelve a intentarlo."
   - TODO(servidor): el resto de la revisión de la landing lo define el servidor.
2. **Píxel**: el conjunto usa el píxel del perfil (`meta.pixel_id`) con el evento de compra. Si el perfil no tiene píxel, detente: "Tu perfil no tiene píxel. Créalo en el Administrador de eventos, corre /zepol-ads:configurar para elegirlo y vuelve aquí."
3. **Enlaces**: cada anuncio lleva `landing_url` como destino y las UTM de la regla 6 de `armado-seguro`.
4. El botón lo define el servidor.

### Rama WhatsApp

1. **Número**: la página del perfil necesita el número de WhatsApp Business (`meta.whatsapp_numero`) vinculado y verificado. `meta.whatsapp_vinculado` dice lo que respondió el alumno, pero no lo confirma. Si no lo está: "Tu número de WhatsApp no está vinculado a tu página. Vincúlalo en la configuración de WhatsApp de tu página y vuelve a intentarlo." Sin ese vínculo el borrador se crea, pero no se puede publicar.
2. **Destino**: el conjunto lleva `destination_type: WHATSAPP` y el botón es `WHATSAPP_MESSAGE`, sin enlace web ni UTM.
3. **Mensaje de bienvenida**: TODO(servidor): el texto lo entrega el servidor. TODO(prueba): el conector no lo configura en el creativo; confirmar en la primera prueba cómo cargarlo.
4. TODO(prueba): la rama WhatsApp todavía no se probó de punta a punta con el conector. En la primera campaña, revisa cada paso con el alumno en el Administrador.

## Paso 5. Creativos

- **Ganadores que se reutilizan**: van por su publicación (`object_story_id` + `url_tags`), con la regla 5 de `armado-seguro`. TODO(servidor): cuáles se reutilizan lo decide el servidor con el diagnóstico.
- **Imágenes nuevas**: TODO(servidor): generarlas con la herramienta `generar_imagen` de Zepol Cloud, a partir de la foto real del producto. El servidor las aloja y devuelve una URL pública. Súbelas a la cuenta con `ads_creative_upload_media`.
  - Sin servidor: "Todavía no puedo generar imágenes: el servidor de ZEPOL aún no está conectado."
- **Videos o imágenes del alumno**: necesitan una URL pública para subirse. TODO(servidor): alojarlos en Zepol Cloud. No uses alojamientos temporales públicos.
- **Textos**: TODO(servidor): los entrega el servidor. Muéstralos tal cual al alumno; si pide cambios, pide al servidor la versión corregida.

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

- Guarda la especificación completa de la campaña y los IDs de cada objeto en `zepol/recetas/{slug-de-la-campana}.json`, apenas se cree.
- La receta sirve para verificar lo publicado, reconstruir si algo se daña y relanzar la misma estructura con otro producto.
- TODO: definir el formato de la receta.
- TODO(servidor): guardarla en Zepol Cloud con la herramienta `guardar_campana`.

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
3. En WhatsApp, recuerda: "Anota tus pedidos de WhatsApp de cada día para que el reporte calcule tu costo por pedido entregado." TODO(servidor): dónde se anotan (panel de ZEPOL).
4. TODO(servidor): cuándo correr el primer `/zepol-ads:reporte`.
