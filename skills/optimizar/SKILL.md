---
name: optimizar
description: Revisión diaria de las campañas activas con ZEPOL ADS. Lee los resultados de cada conjunto, pide al servidor de ZEPOL la acción recomendada para cada uno y la propone al alumno en una tabla. Nunca aplica nada sin aprobación explícita. Úsala cuando el alumno escriba /zepol-ads:optimizar o pida revisar, escalar, apagar o ajustar presupuestos de sus campañas.
---

# Optimizar campañas

Cada día, revisa los conjuntos activos y **propone** una acción para cada uno. **Nunca aplica nada sin la aprobación explícita del alumno.**

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Te propongo"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Aplicar cambios", "Aplicar solo algunos", "No cambiar nada".
- Errores con la forma qué pasó + cómo arreglarlo.

## Metodología: siempre del servidor

- Las reglas de optimización de ZEPOL viven en el servidor, no en este plugin: cuándo pausar, cuándo mantener, cuánto subir o bajar y cuándo preguntar al alumno.
- Pídelas con la herramienta `metodologia` (o `diagnostico`) del servidor de ZEPOL. TODO(servidor): la herramienta aún no existe; nombre y parámetros por confirmar.
- Si el servidor no está conectado, no propongas acciones con tu propio criterio. Di: "El servidor de ZEPOL aún no está conectado, así que todavía no puedo proponerte cambios. Puedes ver tus números con /zepol-ads:reporte." y detente.

## Paso 0. Antes de empezar

1. Lee el perfil `zepol/perfil-negocio.json`. Si no existe, pide correr `/zepol-ads:configurar`.
2. Carga la skill `armado-seguro`.
3. TODO(servidor): cuándo conviene correr la revisión cada día lo define el servidor.

## Paso 1. Leer los resultados

- Lista las campañas y los conjuntos activos con `ads_get_ad_entities` y lee sus métricas del periodo que pida el servidor (verifica los campos con `ads_get_field_context`).
- Para cada conjunto, ten a mano el producto (`zepol/productos/`) y el canal de su receta (`zepol/recetas/`).

## Paso 2. Pedir la acción recomendada

- TODO(servidor): enviar a `metodologia` (etapa `optimizacion`) los resultados de cada conjunto, el precio de venta del producto, el canal y el perfil. Devuelve una acción por conjunto con su motivo, o una pregunta para el alumno cuando la decisión es suya.
- No calcules ni inventes acciones por tu cuenta.

## Paso 3. Proponer

1. Muestra una tabla: conjunto, resultado clave, acción propuesta, presupuesto actual, presupuesto nuevo (si cambia) y motivo.
2. Si el servidor marcó un conjunto como decisión del alumno, pregúntale lo que pida el servidor.
3. Pregunta: "¿Aplico estos cambios?" con las opciones "Aplicar cambios", "Aplicar solo algunos" y "No cambiar nada".
   - "Aplicar solo algunos": pregunta cuáles, uno por uno o con una lista.
   - Sin un "sí" explícito no cambies nada.

## Paso 4. Aplicar lo aprobado

Solo lo que el alumno aprobó, con las reglas de `armado-seguro`:

1. Candado del editor antes de cambiar nada (regla 1).
2. Pausar: `ads_update_entity` con solo `{"status":"PAUSED"}`. No llames `ads_activate_entity` después (regla 10).
3. Cambiar un presupuesto: `ads_update_entity` con `daily_budget` en céntimos (S/25 = `2500`). El cambio queda en borrador (`is_draft=true`):
   - muestra el cambio pendiente;
   - publícalo con `ads_activate_entity`, con solo ese conjunto en `object_ids` y `publish_as_active=false`, únicamente con el "sí" del alumno a esa publicación.
4. Verifica cada cambio leyendo el objeto publicado con `ads_get_ad_entities`. Si no se ve el cambio, di que está pendiente; no lo des por hecho.
5. Pide refrescar el Administrador antes de abrirlo.

## Paso 5. Cierre

- Resume en una línea qué cambió y qué quedó igual.
- Actualiza la receta con los presupuestos y estados nuevos.
- TODO(servidor): registrar las acciones en Zepol Cloud para el historial del panel.
- TODO(servidor): mostrar los avisos que entregue el servidor junto con las acciones.
