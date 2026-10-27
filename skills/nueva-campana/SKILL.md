---
name: nueva-campana
description: Arma una campaña nueva de Meta para un producto contra entrega con ZEPOL ADS, a WhatsApp o a su web (Shopify, WooCommerce u otra, con formulario contra entrega). Pide los datos del producto, trae la metodología de Zepol Cloud, crea todo como borrador con las reglas de armado seguro, guarda la receta, hace la revisión antes de publicar y publica solo con un sí explícito del alumno. Úsala cuando el alumno escriba /zepol-ads:nueva-campana o pida lanzar, crear o testear un producto en Meta.
---

# Nueva campaña

Arma la campaña de un producto. Todo se crea como borrador y nada se publica ni gasta sin el "sí" explícito del alumno.

El recorrido completo: metodología de Zepol Cloud por etapa → `guardar_producto` → `generar_imagen` → borrador en Meta → `guardar_receta` (primero como borrador y, al publicar, como publicada, con los IDs de Meta).

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
- Si no ves las herramientas de Zepol Cloud (por ejemplo `obtener_perfil`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Customize (Personalizar) → Plugins → ZEPOL ADS, abre la pestaña Connectors (Conectores) y toca Connect (Conectar). En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:nueva-campana." y detente.
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.
- Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = nueva-campana`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Paso 0. Antes de empezar

1. Llama `estado_cuenta` con `version_plugin = "0.6.4"`. Si `onboarding.arranque_completo` es `false`, di: "Antes de armar una campaña falta tu arranque: escribe /zepol-ads:empezar. En unos minutos encuentra tus activos de Meta, estudia tu cuenta y te deja listo." y detente. Si `/zepol-ads:empezar` te trajo hasta aquí, el arranque ya está hecho.
2. Llama `obtener_perfil`. Si hay varios perfiles, pregunta con qué negocio trabajas.
   - Si `categoria` es `marca_propia`, di: "Las campañas para marcas propias llegan pronto, con su propia metodología. Mientras tanto, puedo darte el reporte de tus campañas actuales con /zepol-ads:reporte: cuánto gastaste, cuántas ventas tuviste y cuánto te costó cada una." y detente. No le ofrezcas /zepol-ads:optimizar: todavía usa las reglas de contra entrega.
3. Confirma que el conector de Meta está disponible. Si no: "No encuentro el conector de Meta. En la app de Claude, entra a Customize (Personalizar) → Connectors (Conectores), busca Meta Ads, toca Connect (Conectar) y entra con tu cuenta de Facebook. En Claude Code, escribe /mcp y autentica el conector de Meta. Después vuelve a correr /zepol-ads:nueva-campana."
4. Usa el `diagnostico` del perfil (ganadores y alertas) en los pasos que lo piden.
5. Carga la skill `armado-seguro`. Todo lo que se cree o edite en Meta sigue sus reglas, en su orden.

## Paso 1. Datos del producto

Pide los datos que guarda `guardar_producto`. Si el producto ya está en el perfil (`obtener_perfil` trae los productos), muéstralo y pregunta solo lo que cambió y lo que falta.

**Sin dato no es "no".** Si `margen_bajo`, `es_complejo` o `para_personas_mayores` vienen en `null` (o no vienen), el alumno nunca los respondió. Antes de armar la campaña, pregúntale los que falten y guárdalos con `guardar_producto` (`true` o `false`). Un `false` ya es una respuesta: no lo vuelvas a preguntar. Si el alumno no sabe, no lo envíes.

| Campo | Pregunta |
|---|---|
| `nombre` | "¿Qué producto vas a anunciar?" |
| `precio_venta` | "¿A cuánto lo vendes? No va en el anuncio: lo necesito para medir si la campaña es rentable." |
| `margen_bajo` | "¿Dirías que el margen de este producto es bajo?" |
| `creativos_disponibles` | "¿Qué creativos tienes de este producto: videos, imágenes, los dos o ninguno?" (`videos`, `imagenes`, `ambos` o `ninguno`). Luego pide cada uno: archivo, URL o ID de la publicación. |
| `fotos_urls` | "¿Tienes fotos reales del producto? Pásame el enlace público (https) de una o varias, por ejemplo el de la foto en tu tienda." La primera es la que usa `generar_imagen`. |
| `es_complejo` y `para_personas_mayores` | "¿El producto es difícil de explicar? ¿Es para personas mayores?" |
| `landing_url` | Solo si la campaña va a su web: "¿Cuál es el enlace del producto en tu tienda?" Debe empezar con https://. |
| `extras` | "Cuéntame del producto: qué problema resuelve, para quién es, sus beneficios, las dudas que más te preguntan, la garantía y sus variantes." Guarda solo lo que el alumno diga, en `extras` (`problema`, `publico`, `beneficios`, `objeciones`, `garantia`, `variantes`). |

- No calcules el margen: cada negocio conoce sus números.
- No inventes datos del producto. Si el alumno no sabe algo, no lo envíes.
- Si el producto tiene `estilo` (lo propone `/zepol-ads:empezar`), úsalo en sus imágenes. Si no lo tiene y hay fotos, propónlo como en `empezar`, Paso 2b, y guárdalo con el producto.
- Guarda el producto con `guardar_producto`, con el `perfil_id` del perfil. Si ya existía, envía su `id` para actualizarlo: lo que no envíes se conserva y los `extras` se combinan con los guardados. `fotos_urls` reemplaza la lista completa.

## Paso 2. Canal: WhatsApp o tu web

El canal `shopify` de la metodología es la web con formulario contra entrega, sea Shopify, WooCommerce u otra.

- `tipo_negocio = shopify` (vende con una web) → canal `shopify`.
- `tipo_negocio = whatsapp` → canal `whatsapp`.
- `tipo_negocio = hibrido` → pregunta: "¿A dónde quieres llevar a la gente con esta campaña?" con las opciones "A mi tienda" y "A mi WhatsApp".
  - Si la metodología (`obtener_metodologia` con `etapa = estructura`) recomienda un canal para este negocio y este producto, muéstralo antes de que el alumno elija. El alumno decide.

## Paso 3. Pedir la metodología

- Llama `obtener_metodologia` con el canal elegido para las etapas `estructura`, `configuracion`, `creativos` y `textos`.
- Si Zepol Cloud responde que el alumno llegó al límite de consultas de hoy, muéstrale ese mensaje tal cual y detente. No armes ninguna parte sin su metodología.
- Con esas reglas, el perfil, el producto y el diagnóstico, arma la especificación completa de la campaña: objetivo, optimización, estructura, presupuestos, públicos, ubicaciones, nombres, recomendación de inicio y el plan de creativos y textos.
- Aplica solo lo que dicen las reglas. Si algo no está cubierto, pregúntale al alumno.

## Paso 4. Rama según el canal

### Rama web (Shopify, WooCommerce u otra, con formulario contra entrega)

1. **Landing**: revisa que `landing_url` responda (HTTP 200).
   - Si no responde: "Tu landing no responde. Revisa que el producto esté publicado en tu tienda y vuelve a intentarlo."
   - **Formulario contra entrega:** no le preguntes al alumno qué app usa. Míralo tú en la landing (Releasit u otra app de formulario contra entrega). Si el perfil no lo tiene o tiene otra, guárdalo con `guardar_perfil` (`perfil_id`, `nombre_negocio` y `app_formulario`, el nombre de la app). Si no encuentras ningún formulario, dilo: "No veo un formulario contra entrega en tu landing. Sin él, tus clientes no pueden pedir." y agrégalo a pendientes.
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
- **Imágenes nuevas**: genéralas con `generar_imagen` (`producto_id`, `temperatura`, `angulo` y los textos). En `angulo` va solo el ángulo: ZEPOL agrega solo el estilo del producto y los colores de la marca. Usa la primera foto guardada del producto; pasa `foto_producto_url` solo si el alumno quiere otra foto. Zepol Cloud aloja cada imagen y devuelve su link y la imagen misma.
  - **Créditos:** cada imagen usa 1 crédito de ZEPOL del alumno. La primera corrección de cada imagen es gratis; las siguientes usan 1 crédito. Antes de generar, dile cuántas imágenes vas a crear y cuántos créditos le quedan (`estado_cuenta` → `creditos`), y pregunta si sigues. Si algo falla, el crédito vuelve solo.
  - **Tiempo:** crear una imagen tarda hasta 2 minutos; avísale al alumno antes de pedirla. Si la llamada se corta o no llega la respuesta, no la pidas otra vez: usa `ver_creativos` con el `producto_id` y toma la más reciente. Así no paga dos veces.
  - **Revisión antes de subirla:** muéstrale la imagen al alumno y revísala con él:
    - los textos, escritos tal cual y con sus tildes, sin letras de más ni palabras inventadas;
    - el producto, parecido a la foto real (forma, colores, etiquetas y logo);
    - nada importante (textos, producto o logo) fuera de la zona segura que indica la respuesta (`revisar`).

    Si algo no está bien, ofrécele una corrección antes de subirla a Meta y recuérdale que la primera corrección de cada imagen es gratis (`correccion` en la respuesta). Si no tienes la imagen a la vista (por ejemplo, porque la recuperaste), pídela con `ver_creativos` y `ver_imagen: true`.
  - **Corregir:** llama `generar_imagen` con el mismo `producto_id`, `temperatura` y `angulo`, más `editar_creativo_id` (el `id` del creativo que hay que arreglar) y `cambios` (qué cambiar, en una frase concreta: "corrige la tilde de Pídela", "sube el texto principal"). Envía `textos` solo si cambian. La corrección es un creativo nuevo: desde ahí usa su `id`.
  - **Subirla a Meta:** solo cuando el alumno la apruebe, con `ads_creative_upload_media`: `upload_source: "URL"`, `media_type: "IMAGE"` y `media_url` = el link de la imagen (`creativo.url`; la respuesta lo trae listo en `subir_a_meta`). Toma el `image_hash` que devuelve para el anuncio (regla 4 de `armado-seguro`).
  - Si `generar_imagen` dice que no le quedan créditos, que llegó al límite de imágenes de hoy, que la creación de imágenes quedó en pausa hasta mañana o que todavía no está activa, díselo y sigue con los creativos que el alumno tiene.
  - Si `generar_imagen` dice que falta la foto, vuelve al paso 1 y pide el enlace de una foto real del producto.
  - Guarda el `id` de cada creativo aprobado: va en la receta (`creativo.creativo_id`).
- **Videos o imágenes del alumno**: necesitan una URL pública para subirse. Por ahora pídele al alumno un enlace público propio (por ejemplo, de su tienda). No uses alojamientos temporales públicos.
- **Textos**: escríbelos con la metodología (`textos`), que trae las estructuras y los bancos de frases. Ningún texto ni imagen lleva el precio. Muéstralos al alumno; si pide cambios, ajústalos respetando esas reglas.

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

- Apenas se cree el borrador, guarda la receta en Zepol Cloud con `guardar_receta`: `perfil_id`, `producto_id`, `canal`, `nombre_campana`, `estado = borrador` y en `receta` la especificación completa, con el formato que trae la herramienta:
  - `origen = nueva_campana` y la zona horaria de la cuenta;
  - la campaña, cada conjunto y cada anuncio con su ID de Meta (`meta_id`, solo números);
  - por conjunto: presupuesto por día, inicio, optimización, píxel y evento (Shopify), destino, público y ubicaciones tal como se enviaron a Meta;
  - por anuncio: tipo, creativo (`creativo_id` de `generar_imagen`, `image_hash`, `video_id` u `object_story_id`), textos, botón, enlace y UTM.
- Guarda el `id` de la receta que devuelve.
- Al publicar, actualízala con su `id` y `estado = publicada`.
- La receta sirve para verificar lo publicado (/zepol-ads:verificar), limpiar y reconstruir si algo se daña (/zepol-ads:limpiar) y relanzar la misma estructura con otro producto.
- Si la herramienta rechaza un dato, corrígelo y vuelve a enviarla. No sigas al paso 9 sin la receta guardada.

## Paso 9. Validar el borrador

Regla 8 de `armado-seguro`: leer con `object_state=draft`, todo `VALIDATED`, vistas previas y revisión de un anuncio de cada tipo en el Administrador (y después cerrar el editor).

## Paso 10. Revisión antes de publicar

Muestra una tabla con cada punto y su estado ("Listo" o qué falta):

| Punto | Qué se revisa |
|---|---|
| Validación | Todos los objetos en `VALIDATED`, sin errores |
| Destino | Web: píxel y evento de compra en cada conjunto. WhatsApp: número vinculado a la página |
| Presupuesto | Total por día de la campaña |
| Inicio | Fecha y hora en la zona horaria de la cuenta |
| Catálogo y mejoras | Catálogo vacío en la campaña y mejoras en `OPT_OUT` en cada anuncio |
| Enlaces | Web: cada URL responde y lleva sus UTM |
| Creativos | Cada video e imagen aparece en su vista previa |
| Vistas previas | Un enlace `preview_url` por anuncio |

Termina con: "Publica solo esta campaña, no otros borradores."

## Paso 11. Publicar

Regla 10 de `armado-seguro`. Pregunta: "¿Publico la campaña {nombre}? Empezará a gastar S/{total} por día desde el {fecha y hora}." con las opciones "Publicar campaña" y "Todavía no". Sin un "sí" explícito no llames `ads_activate_entity`.

## Paso 12. Verificar y cerrar

1. Haz la verificación después de publicar (regla 11 de `armado-seguro`, o `/zepol-ads:verificar`).
2. Confirma en una línea qué quedó publicado y desde cuándo.
3. En WhatsApp, recuerda: "Cada día dime cuántos pedidos confirmados, entregados y devueltos tuviste, y los registro para calcular tu costo por pedido entregado." Regístralos con `registrar_pedidos_whatsapp`, con el `receta_id` de esta campaña.
4. Termina con: "Desde mañana, corre /zepol-ads:optimizar cada mañana. Para ver tus números, usa /zepol-ads:reporte."
