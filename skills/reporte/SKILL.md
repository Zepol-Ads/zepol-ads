---
name: reporte
description: Lee las métricas de Meta de las campañas del alumno por campaña, conjunto y anuncio, pide a Zepol Cloud la lectura de esos números (evaluar_conjuntos) y calcula el costo por pedido entregado con la tasa de entrega del perfil. Solo lee, no cambia nada en Meta. Úsala cuando el alumno escriba /zepol-ads:reporte o pregunte cómo van sus campañas.
---

# Reporte de campañas

Lee los números de Meta y los traduce a lo que importa en contra entrega: cuánto cuesta cada pedido que sí se entrega.

Esta skill **solo lee**. No cambia nada en Meta. Para proponer cambios está `/zepol-ads:optimizar`.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tus números"), nunca "nosotros".
- Montos en soles con el formato S/10.50. Separador visual: punto medio con espacios.
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre de Zepol Cloud

- Qué es un buen o un mal resultado vive en Zepol Cloud, no en este plugin. La lectura de cada conjunto la entrega `evaluar_conjuntos`, y la explicación de las reglas, `obtener_metodologia` con `etapa = optimizacion`.
- No califiques los números con tu propio criterio.
- Si no ves las herramientas de Zepol Cloud (`obtener_perfil`, `evaluar_conjuntos`), muestra los números sin calificarlos y di: "ZEPOL ADS no está conectado, así que te muestro tus números sin evaluarlos. En la app de Claude, conéctalo en Configuración → Conectores; en Claude Code, escribe /mcp y autentica zepol-cloud."
- Si una herramienta de Zepol Cloud responde que la cuenta no tiene una licencia activa, muestra ese mensaje tal cual y detente.

## Paso 0. Antes de empezar

1. Llama `obtener_perfil`. Si no hay perfiles, pide correr `/zepol-ads:configurar`. Si hay varios, pregunta con qué negocio trabajas.
2. Del perfil usa `meta_cuenta_publicitaria_id` y `tasa_entrega_pct`.
3. Si el alumno no dijo qué campaña ni qué periodo revisar, pregunta. Por defecto: las campañas activas, desde su inicio hasta ayer, en la zona horaria de la cuenta.

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
  - Pregunta: "¿Cuántos pedidos entregados y pagados tuviste de esta campaña en estas fechas? Si no lo sabes, uso la estimación de ZEPOL."
  - Si responde, costo por pedido entregado = gasto ÷ pedidos entregados.
  - Si no, usa el `costo_por_pedido_estimado` de `evaluar_conjuntos` y divídelo entre la tasa de entrega. Dilo claro: "Es una estimación".
- Si el perfil no tiene tasa de entrega: "No tengo tu tasa de entrega, así que no puedo calcular el costo por pedido entregado. Corre /zepol-ads:configurar para agregarla." y muestra el resto del reporte.

## Paso 4. Mostrar el reporte

1. Resumen en 2 o 3 líneas: gasto total, resultados, pedidos y costo por pedido entregado.
2. Tabla por conjunto: gasto, resultados, costo por resultado, ROAS si aplica, costo por pedido entregado y la lectura de ZEPOL en palabras simples.
3. Tabla por anuncio: gasto, resultados, costo por resultado y CTR. Marca los anuncios que más y menos rinden.
4. Si la tendencia o las alertas de Meta explican algo, dilo en una o dos líneas.
5. Si hay algo para cambiar, termina con: "Si quieres que proponga cambios, corre /zepol-ads:optimizar."
