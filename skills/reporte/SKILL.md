---
name: reporte
description: Lee las métricas de Meta de las campañas del alumno por campaña, conjunto y anuncio, y calcula el costo por pedido entregado con la tasa de entrega contra entrega del perfil. La lectura de resultados la entrega el servidor de ZEPOL. Solo lee, no cambia nada en Meta. Úsala cuando el alumno escriba /zepol-ads:reporte o pregunte cómo van sus campañas.
---

# Reporte de campañas

Lee los números de Meta y los traduce a lo que importa en contra entrega: cuánto cuesta cada pedido que sí se entrega.

Esta skill **solo lee**. No cambia nada en Meta. Para proponer cambios está `/zepol-ads:optimizar`.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Reviso tus números"), nunca "nosotros".
- Montos en soles con el formato S/10.50. Separador visual: punto medio con espacios.
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre del servidor

- Qué es un buen o un mal resultado, y qué hacer con cada uno, vive en el servidor de ZEPOL, no en este plugin.
- Pídelo con la herramienta `metodologia` del servidor de ZEPOL. TODO(servidor): la herramienta aún no existe; nombre y parámetros por confirmar.
- Si el servidor no está conectado, muestra los números sin calificarlos y di: "El servidor de ZEPOL aún no está conectado, así que te muestro tus números sin evaluarlos. Cuando esté listo, vuelve a correr /zepol-ads:reporte."

## Paso 0. Antes de empezar

1. Lee el perfil `zepol/perfil-negocio.json`. Si no existe, pide correr `/zepol-ads:configurar`.
2. Pregunta qué campaña y qué periodo revisar si no lo dijo. Por defecto, las campañas con receta en `zepol/recetas/` y desde su inicio.

## Paso 1. Leer las métricas

- Usa `ads_get_ad_entities` a nivel campaña, conjunto y anuncio, con el periodo elegido. Verifica cada campo con `ads_get_field_context` antes de pedirlo.
- Pide solo los campos necesarios: gasto, resultados, costo por resultado, compras o conversaciones, valor de compras, CTR y CPM.
- TODO(servidor): la lista exacta de métricas y cortes la define el servidor.

## Paso 2. Costo por pedido entregado

- **Shopify**: costo por pedido entregado = gasto ÷ (compras × tasa de entrega).
  - La tasa de entrega es `entregas.tasa_entrega_pct` del perfil, como fracción (55 % → 0.55).
  - Ejemplo: S/300 de gasto, 20 compras y 55 % de entrega → S/300 ÷ (20 × 0.55) = S/27.27 por pedido entregado.
- **WhatsApp**: Meta mide conversaciones, no compras.
  - Si el alumno registró sus pedidos, usa sus pedidos entregados: gasto ÷ pedidos entregados.
  - Si no, TODO(servidor): la estimación de pedidos a partir de las conversaciones la entrega el servidor.
  - TODO(servidor): leer los pedidos registrados desde el panel de ZEPOL.
- Si el perfil no tiene tasa de entrega: "No tengo tu tasa de entrega, así que no puedo calcular el costo por pedido entregado. Corre /zepol-ads:configurar para agregarla." y muestra el resto del reporte.
- ROAS (ventas ÷ gasto): en Shopify sale del valor de compras del píxel. Usa `precio_venta` del producto si hace falta comparar.

## Paso 3. Mostrar el reporte

1. Resumen en 2 o 3 líneas: gasto total, pedidos y costo por pedido entregado.
2. Tabla por conjunto y otra por anuncio, con las métricas del paso 1 y el costo por pedido entregado.
3. TODO(servidor): pedir a `metodologia` (etapa `reporte`) la lectura de estos números y mostrarla debajo de las tablas.
4. Si hay algo para cambiar, termina con: "Si quieres que proponga cambios, corre /zepol-ads:optimizar."

## Paso 4. Guardar

- Guarda el reporte en `zepol/reportes/{slug-de-la-campana}-{fecha}.json` con tu herramienta de escritura de archivos.
- TODO: definir el formato del reporte.
- TODO(servidor): enviarlo a Zepol Cloud con `guardar_reporte`, para el panel, y actualizar el perfil de la cuenta del diagnóstico.
