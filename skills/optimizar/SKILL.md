---
name: optimizar
description: Revisión diaria de las campañas activas con ZEPOL ADS. Lee los resultados de cada conjunto en Meta, le pide a Zepol Cloud la acción recomendada para cada uno (evaluar_conjuntos) y la propone al alumno en una tabla. Nunca aplica nada sin aprobación explícita. Úsala cuando el alumno escriba /zepol-ads:optimizar o pida revisar, escalar, apagar o ajustar presupuestos de sus campañas.
---

# Optimizar campañas

Cada día, revisa los conjuntos activos y **propone** una acción para cada uno. **Nunca aplica nada sin la aprobación explícita del alumno.**

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Te propongo"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Aplicar cambios", "Aplicar solo algunos", "No cambiar nada".
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre de Zepol Cloud

- Las reglas de optimización de ZEPOL viven en Zepol Cloud, no en este plugin: cuándo apagar, cuándo mantener, cuánto subir o bajar y cuándo preguntar al alumno.
- Las acciones las calcula la herramienta `evaluar_conjuntos`. La explicación de las reglas la entrega `obtener_metodologia` con `etapa = optimizacion`.
- No calcules ni inventes acciones por tu cuenta, ni con tu propio criterio.
- Si no ves las herramientas de Zepol Cloud (`obtener_perfil`, `evaluar_conjuntos`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Customize → Plugins → ZEPOL ADS, abre la pestaña Conectores y toca Conectar. En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:optimizar." y detente.
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.
- Si Zepol Cloud responde que el alumno llegó al límite de consultas de hoy, muéstrale ese mensaje tal cual y detente. No es un error: no lo registres con `registrar_error`.
- Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = optimizar`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Paso 0. Antes de empezar

1. Llama `estado_cuenta` con `version_plugin = "0.6.1"`. Mira `onboarding.pasos`:
   - si falta el perfil o la cuenta publicitaria (`perfil` o `activos_meta` en `false`), di: "Antes de optimizar falta tu arranque: escribe /zepol-ads:empezar. En unos minutos encuentra tus activos de Meta y estudia tu cuenta." y detente;
   - si solo falta el diagnóstico, sigue y, al terminar, recuerda: "Te falta el arranque: escribe /zepol-ads:empezar para que estudie tu cuenta."
2. Llama `obtener_perfil`. Si hay varios perfiles, pregunta con qué negocio trabajas.
   - Si el negocio es de marca propia (`categoria = marca_propia`), detente: "La optimización diaria todavía usa las reglas de contra entrega y podría proponerte apagar conjuntos que para tu marca sí son rentables. Las campañas para marcas llegan pronto. Mientras tanto, revisa tus números con /zepol-ads:reporte."
3. Del perfil usa `meta_cuenta_publicitaria_id`. Si no está, pídele al alumno el ID de su cuenta publicitaria.
4. Carga la skill `armado-seguro`.
5. Trae las recetas con `listar_recetas` para saber qué campaña tiene receta (por su `meta_campaign_id`). Una campaña sin receta se optimiza igual; al final puedes ofrecer adoptarla con `/zepol-ads:verificar`.

## Paso 1. Leer los resultados en Meta

1. Verifica los campos con `ads_get_field_context` antes de usarlos.
2. Lista los conjuntos activos con `ads_get_ad_entities` en `level = adset`, filtrando `effective_status` en `ACTIVE`. Pide estos campos: `id`, `name`, `campaign_id`, `campaign_name`, `start_time`, `daily_budget`, `destination_type`, `optimization_goal`, `amount_spent`, `results`, `cost_per_result`, y en ventas por web también `purchase_roas` y `omni_purchase_values`.
3. **Periodo:** usa el que indique la metodología de optimización. Si no indica uno, usa desde el día en que arrancó cada conjunto hasta ayer, en la zona horaria de la cuenta (`time_range`). No incluyas el día de hoy, que está incompleto.
4. Si el alumno pidió una campaña concreta, revisa solo los conjuntos de esa campaña.
5. Si no hay conjuntos activos, dilo en una línea y termina.

## Paso 2. Armar los datos de cada campaña

Agrupa los conjuntos por campaña. Para cada campaña:

1. **Canal:**
   - `destination_type` de WhatsApp o mensajes → `whatsapp`;
   - sitio web → `shopify`;
   - si no está claro, pregúntale al alumno si la campaña lleva a WhatsApp o a su tienda.
2. **Producto y precio:**
   - busca en los productos del perfil (`obtener_perfil` los trae) el que corresponde a la campaña, por su nombre;
   - si no lo encuentras, pregunta: "¿Qué producto vende la campaña {nombre} y a qué precio?" No lo inventes. Si el alumno no lo sabe, envía la evaluación sin precio.
3. **Datos de cada conjunto** para `evaluar_conjuntos`:
   - `nombre`: el nombre del conjunto;
   - `dia`: días completos desde su `start_time` hasta hoy, en la zona horaria de la cuenta. Si arrancó hoy, es `0`;
   - `gasto`: `amount_spent` del periodo, en soles;
   - `presupuesto_diario`: `daily_budget` dividido entre 100, porque Meta lo da en céntimos;
   - `resultados`: `results` del periodo (conversaciones en WhatsApp, compras en la web);
   - `costo_por_resultado`: `cost_per_result`, si Meta lo da;
   - en la web, `roas` (`purchase_roas`) y `valor_compras` (`omni_purchase_values`), si Meta los da;
   - en WhatsApp, `pedidos` solo si el alumno te dice cuántos pedidos salieron de ese conjunto.
   - No envíes datos que no tienes: deja fuera el campo.

## Paso 3. Pedir la acción recomendada

1. Llama `evaluar_conjuntos` una vez por campaña, con su `canal`, su `precio_producto` y la lista de conjuntos.
2. Si necesitas explicar una regla, pide `obtener_metodologia` con `etapa = optimizacion` y el canal, y explícala con sus palabras. No agregues reglas propias.

## Paso 4. Proponer

1. Muestra una tabla por campaña con estas columnas: conjunto, resultados, costo por resultado, acción propuesta, presupuesto actual, presupuesto nuevo (si cambia) y motivo.
   - Escribe cada acción en palabras simples a partir de lo que devuelve `evaluar_conjuntos`: "Apagar", "Mantener", "Tú decides", o "Subir a S/{presupuesto_nuevo}" y "Bajar a S/{presupuesto_nuevo}" cuando cambia el presupuesto.
   - Usa el motivo que devuelve `evaluar_conjuntos`.
2. Si un conjunto trae `pregunta_al_usuario`, hazle esa pregunta al alumno tal cual.
3. Pregunta: "¿Aplico estos cambios?" con las opciones "Aplicar cambios", "Aplicar solo algunos" y "No cambiar nada".
   - "Aplicar solo algunos": pregunta cuáles, uno por uno o con una lista.
   - "Mantener" no cambia nada en Meta.
   - Sin un "sí" explícito no cambies nada.

## Paso 5. Aplicar lo aprobado

Solo lo que el alumno aprobó, con las reglas de `armado-seguro`:

1. Candado del editor antes de cambiar nada (regla 1).
2. Apagar: `ads_update_entity` con solo `{"status":"PAUSED"}`. No llames `ads_activate_entity` después (regla 10).
3. Cambiar un presupuesto: `ads_update_entity` con `daily_budget` en céntimos (S/25 = `2500`). El cambio queda en borrador (`is_draft=true`):
   - muestra el cambio pendiente;
   - publícalo con `ads_activate_entity`, con solo ese conjunto en `object_ids` y `publish_as_active=false`, únicamente con el "sí" del alumno a esa publicación.
4. Verifica cada cambio leyendo el objeto publicado con `ads_get_ad_entities`. Si no se ve el cambio, di que está pendiente; no lo des por hecho.
5. Pide refrescar el Administrador de anuncios antes de abrirlo.

## Paso 6. Guardar la optimización

Guarda una optimización por campaña con `guardar_reporte`, con el formato que trae la herramienta:

- `origen = optimizacion`, el `perfil_id`, el `receta_id` si la campaña tiene receta y el periodo leído;
- los totales de la campaña y, en `metricas`, la campaña con su ID de Meta, el canal y la tabla por conjunto;
- en `recomendaciones`, una por conjunto: la acción y el motivo que devolvió `evaluar_conjuntos`, el presupuesto actual y el nuevo, si el alumno la aprobó (`aprobada`) y si quedó aplicada y verificada en Meta (`aplicada`).

Guárdala también si el alumno no aprobó nada: sirve para comparar mañana.

## Paso 7. Cierre

- Resume en una línea qué cambió y qué quedó igual.
- Si algún conjunto quedó como "Tú decides" y el alumno no respondió, recuérdaselo.
- Termina con: "Mañana vuelve a correr /zepol-ads:optimizar antes de tu hora de más ventas. Para ver el detalle de tus números, usa /zepol-ads:reporte."
