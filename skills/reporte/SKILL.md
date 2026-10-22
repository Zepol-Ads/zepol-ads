---
name: reporte
description: Lee las métricas de Meta de las campañas del alumno por campaña, conjunto y anuncio, pide a Zepol Cloud la lectura de esos números (evaluar_conjuntos), calcula el costo por pedido entregado (con la tasa de entrega del perfil o los pedidos de WhatsApp registrados) y guarda el reporte en Zepol Cloud. No cambia nada en Meta. Úsala cuando el alumno escriba /zepol-ads:reporte o pregunte cómo van sus campañas.
---

# Reporte de campañas

Lee los números de Meta y los traduce a lo que importa en contra entrega: cuánto cuesta cada pedido que sí se entrega.

Esta skill **no cambia nada en Meta**: solo lee los números y guarda el reporte en Zepol Cloud. Para proponer cambios está `/zepol-ads:optimizar`.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tus números"), nunca "nosotros".
- Montos en soles con el formato S/10.50. Separador visual: punto medio con espacios.
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre de Zepol Cloud

- Qué es un buen o un mal resultado vive en Zepol Cloud, no en este plugin. La lectura de cada conjunto la entrega `evaluar_conjuntos`, y la explicación de las reglas, `obtener_metodologia` con `etapa = optimizacion`.
- No califiques los números con tu propio criterio.
- Si no ves las herramientas de Zepol Cloud (`obtener_perfil`, `evaluar_conjuntos`), muestra los números sin calificarlos y di: "ZEPOL ADS no está conectado, así que te muestro tus números sin evaluarlos. En la app de Claude, conéctalo en Customize → Plugins → ZEPOL ADS, pestaña Conectores; en Claude Code, escribe /mcp y autentica zepol-cloud."
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.
- Si Zepol Cloud responde que el alumno llegó al límite de consultas de hoy, muéstrale ese mensaje tal cual y detente. No es un error: no lo registres con `registrar_error`.
- Si un paso falla por un error de Meta o de Zepol Cloud, explícaselo al alumno y regístralo con `registrar_error` (`comando = reporte`, el paso, el código si lo hay y un mensaje corto). Nunca envíes claves, tokens, correos ni datos de clientes.

## Paso 0. Antes de empezar

1. Llama `estado_cuenta` con `version_plugin = "0.6.1"`. Mira `onboarding.pasos`:
   - si falta el perfil o la cuenta publicitaria (`perfil` o `activos_meta` en `false`), di: "Antes del reporte falta tu arranque: escribe /zepol-ads:empezar. En unos minutos encuentra tus activos de Meta y estudia tu cuenta." y detente;
   - si solo falta el diagnóstico, sigue y, al terminar, recuerda: "Te falta el arranque: escribe /zepol-ads:empezar para que estudie tu cuenta."
2. Llama `obtener_perfil`. Si hay varios perfiles, pregunta con qué negocio trabajas.
3. Del perfil usa `meta_cuenta_publicitaria_id` y `tasa_entrega_pct`.
   - **Marca propia** (`categoria = marca_propia`): las reglas de lectura de ZEPOL son de contra entrega y la metodología de marcas llega pronto. Muestra los números de Meta (gasto, ventas, costo por venta y ROAS) sin calificarlos: no llames `evaluar_conjuntos` (Paso 2), no pidas ni uses la tasa de entrega (Paso 3) y dilo: "La lectura con el método de ZEPOL para marcas llega pronto: por ahora te muestro tus números tal cual."

4. Si el alumno no dijo qué campaña ni qué periodo revisar, pregunta. Por defecto: las campañas activas, desde su inicio hasta ayer, en la zona horaria de la cuenta.
5. Busca la receta de cada campaña con `listar_recetas` (por su `meta_campaign_id`). Si una campaña no tiene receta, el reporte se guarda igual, sin `receta_id`.

## Paso 1. Leer las métricas

1. Verifica cada campo con `ads_get_field_context` antes de pedirlo.
2. Usa `ads_get_ad_entities` en `level = campaign`, `adset` y `ad`, con el periodo elegido (`time_range` o `date_preset`).
3. Pide solo lo necesario: `amount_spent`, `results`, `cost_per_result`, `ctr`, `cpm`, y en ventas por web `purchase_roas` y `omni_purchase_values`. En conjuntos, también `start_time`, `daily_budget` y `destination_type`.
4. Si están disponibles, suma el contexto de Meta:
   - `ads_insights_performance_trend` con `hide_ui = true`, para la tendencia del costo por resultado;
   - `ads_insights_anomaly_signal`, para cambios bruscos.
   - Úsalos solo para explicar los números, no para decidir acciones.

## Paso 2. Lectura de Zepol Cloud

1. Por cada campaña, arma los datos de sus conjuntos igual que en `/zepol-ads:optimizar`:
   - canal según `destination_type`;
   - precio del producto desde el perfil, o preguntado al alumno;
   - `dia`, `gasto`, `presupuesto_diario` (`daily_budget` ÷ 100), `resultados`, `costo_por_resultado`, `roas` y `valor_compras` cuando existan.
2. Llama `evaluar_conjuntos`. Aquí solo sirve como lectura: no ofrezcas aplicar nada.
3. De cada resultado usa la calificación, el ROAS, el costo por pedido estimado y el motivo.

## Paso 3. Costo por pedido entregado

- **Ventas por web**: costo por pedido entregado = gasto ÷ (compras × tasa de entrega).
  - La tasa de entrega es `tasa_entrega_pct` del perfil, como fracción (55 % → 0.55).
  - Ejemplo: S/300 de gasto, 20 compras y 55 % de entrega → S/300 ÷ (20 × 0.55) = S/27.27 por pedido entregado.
- **WhatsApp**: Meta mide conversaciones, no pedidos.
  - Primero busca los pedidos registrados con `obtener_pedidos_whatsapp` (el `perfil_id`, el periodo y el `receta_id` o el `producto_id` de la campaña). Si hay registros, usa sus entregados y dilo: "Uso los pedidos que registraste."
  - Si no hay registros, pregunta: "¿Cuántos pedidos entregados y pagados tuviste de esta campaña en estas fechas? Si no lo sabes, uso la estimación de ZEPOL."
  - Con los pedidos entregados, costo por pedido entregado = gasto ÷ pedidos entregados.
  - Si no, usa el `costo_por_pedido_estimado` de `evaluar_conjuntos` y divídelo entre la tasa de entrega. Dilo claro: "Es una estimación".
- Si el perfil no tiene tasa de entrega, pídela aquí, una sola vez: "De cada 100 pedidos contra entrega, ¿cuántos se entregan y se pagan? Con eso calculo cuánto te cuesta cada pedido entregado."
  - Si responde, guárdala con `guardar_perfil` (`perfil_id`, `nombre_negocio` y `tasa_entrega_pct`, de 0 a 100) y úsala. Si responde con pedidos totales (por ejemplo, "de 200 se entregan 110"), conviértelo a porcentaje y confírmalo.
  - Si no la sabe, muestra el resto del reporte sin el costo por pedido entregado y dile: "Cuando sepas tu tasa de entrega, dímela y la uso en tu siguiente reporte."

## Paso 4. Mostrar el reporte

1. Resumen en 2 o 3 líneas: gasto total, resultados, pedidos y costo por pedido entregado.
2. Tabla por conjunto: gasto, resultados, costo por resultado, ROAS si aplica, costo por pedido entregado y la lectura de ZEPOL en palabras simples.
3. Tabla por anuncio: gasto, resultados, costo por resultado y CTR. Marca los anuncios que más y menos rinden.
4. Si la tendencia o las alertas de Meta explican algo, dilo en una o dos líneas.

## Paso 5. Guardar el reporte

Guarda un reporte por campaña con `guardar_reporte`, con el formato que trae la herramienta:

- `origen = reporte`, el `perfil_id`, el `receta_id` si la campaña tiene receta y el periodo (`fecha_desde`, `fecha_hasta`);
- los totales de la campaña: gasto, resultados, costo por resultado, ROAS si aplica y costo por pedido entregado;
- en `metricas`: la campaña con su ID de Meta, el canal, la tabla por conjunto (con la lectura de ZEPOL), la tabla por anuncio, los pedidos entregados con su fuente (`registro`, `alumno` o `estimacion`), la tasa de entrega usada y lo que explicaron la tendencia y las alertas de Meta.

Si la herramienta rechaza un dato, corrígelo y vuelve a enviarlo. Confirma en una línea: "Guardé este reporte en ZEPOL."

Si hay algo para cambiar, termina con: "Si quieres que proponga cambios, corre /zepol-ads:optimizar."
