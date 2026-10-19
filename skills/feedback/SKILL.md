---
name: feedback
description: Envía a Alexis un comentario, una idea o un problema sobre ZEPOL ADS, con una calificación opcional de 0 a 10, y lo guarda en Zepol Cloud. Úsala cuando el alumno escriba /zepol-ads:feedback, cuando quiera reportar algo que salió mal o cuando diga qué le gustaría que ZEPOL haga.
---

# Enviar feedback

Le hace llegar a Alexis lo que el alumno piensa de ZEPOL: lo que funcionó, lo que falló y lo que le falta.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Le envío tu comentario"), nunca "nosotros".
- Llamados a la acción con verbo + objeto: "Enviar comentario", "Corregir texto".
- Errores con la forma qué pasó + cómo arreglarlo.

## Zepol Cloud

- Si no ves las herramientas de Zepol Cloud (por ejemplo `registrar_feedback`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Customize → Plugins → ZEPOL ADS, abre la pestaña Conectores y toca Conectar. En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:feedback." y detente.
- El feedback se puede enviar aunque la cuenta no tenga una licencia activa.

## Paso 1. El comentario

1. Si el alumno ya escribió su comentario junto al comando, úsalo. Si no, pregunta: "¿Qué quieres contarle a Alexis? Puede ser algo que falló, algo que te gustó o algo que te gustaría que ZEPOL haga."
2. Si habla de un comando de ZEPOL (por ejemplo, nueva-campana u optimizar), anota cuál.
3. Pregunta: "Del 0 al 10, ¿qué tan útil te está siendo ZEPOL?" Es opcional: si no quiere responder, sigue sin calificación.

## Paso 2. Revisar antes de enviar

- No envíes claves, contraseñas, tokens ni datos de clientes (nombres, teléfonos o direcciones). Si el comentario los trae, quítalos y díselo al alumno.
- Muestra el texto que vas a enviar y pregunta: "¿Le envío esto a Alexis?" con las opciones "Enviar comentario" y "Corregir texto".
  - Si el alumno ya pidió en su mensaje que lo envíes sin preguntar, envíalo directo.

## Paso 3. Enviar

1. Llama `registrar_feedback` con `mensaje`, y `comando` y `satisfaccion` si los tienes.
2. Si responde con un error, muéstralo con qué pasó + cómo arreglarlo. No reintentes en bucle.
3. Confirma en una línea: "Listo, Alexis ya tiene tu comentario. Gracias por ayudar a mejorar ZEPOL."
