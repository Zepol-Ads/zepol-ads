---
name: novedades
description: Muestra las novedades de ZEPOL ADS (funciones nuevas, cambios de la metodología y arreglos) y el estado de la cuenta del alumno en Zepol Cloud (licencia, hasta cuándo recibe novedades, sus créditos para imágenes, cuántas imágenes le quedan hoy y si hay una versión nueva del plugin). Úsala cuando el alumno escriba /zepol-ads:novedades o pregunte qué hay de nuevo o cómo está su cuenta.
---

# Novedades y estado de la cuenta

Cuenta qué hay de nuevo en ZEPOL y cómo está la cuenta del alumno.

## Cómo le hablas al alumno

- Español neutro de LATAM, tuteando. Sin voseo, sin "usted" y sin emojis.
- Directo y concreto, en primera persona del singular ("Te cuento qué hay de nuevo"), nunca "nosotros".
- Fechas en palabras ("31 de enero de 2027"), sin formatos técnicos.
- Errores con la forma qué pasó + cómo arreglarlo.

## Zepol Cloud

- Si no ves las herramientas de Zepol Cloud (por ejemplo `estado_cuenta`), ZEPOL no está conectado. Di: "ZEPOL ADS no está conectado. En la app de Claude, entra a Customize (Personalizar) → Plugins → ZEPOL ADS, abre la pestaña Connectors (Conectores) y toca Connect (Conectar). En Claude Code, escribe /mcp y autentica zepol-cloud. Después vuelve a correr /zepol-ads:novedades." y detente.
- Esta skill funciona aunque la cuenta no tenga una licencia activa.

## Paso 1. Estado de la cuenta

1. Llama `estado_cuenta` con `version_plugin = "0.6.3"` (la versión de este plugin).
2. Muestra una tabla corta con dos columnas, "Dato" y "Estado":
   - **Licencia**: activa, inactiva o sin licencia, con su tipo (beta o compra).
   - **Novedades**: hasta qué fecha las recibe, o "de por vida".
   - **Imágenes de hoy**: cuántas le quedan.
   - **Créditos**: cuántos le quedan para crear imágenes con IA y cuándo se renuevan (`creditos`). Si `por_mes` es 0: "Los créditos del mes vienen con tus novedades: renuévalas para seguir recibiéndolos." Si `imagenes_disponibles` es `false`: "La creación de imágenes todavía no está activa."
   - **Versión del plugin**: la instalada y, si `actualizar_plugin` es `true`, "Hay una versión nueva".
   - **Arranque**: "Hecho" si `onboarding.arranque_completo` es `true`; si no, "Falta: escribe /zepol-ads:empezar".
3. Si la licencia no está activa, muestra el aviso de la herramienta tal cual.
4. Si las novedades vencieron (`novedades_vigentes = false`), explica: "ZEPOL sigue funcionando, pero las funciones nuevas que salgan desde ahora no entran en tu licencia. Escríbele a Alexis por WhatsApp al +51 954 105 380 para renovarlas."

## Paso 2. Novedades

1. Llama `obtener_novedades`. Si el alumno pide algo concreto ("lo de este mes"), usa `desde`.
2. Muestra cada novedad con su título, su fecha, su tipo en palabras ("Función nueva", "Metodología", "Arreglo") y un resumen de su texto en 2 o 3 líneas. Si el alumno pide el detalle de una, muéstrala completa.
3. Si una novedad trae `incluida_en_tu_licencia = false`, márcala: "No entra en tu licencia: necesitas renovar las novedades."
4. Si no hay novedades, dilo en una línea.

## Paso 3. Actualizar el plugin

Si hay una versión nueva del plugin, explica cómo actualizarlo:

- **En la app de Claude:** en Customize (Personalizar) → Plugins, busca zepol-ads y actualízalo. Si tienes encendido "Sync automatically" (sincronizar automáticamente), se actualiza solo.
- **En Claude Code:**

```bash
claude plugin marketplace update zepol-ads-beta
```

```bash
claude plugin update zepol-ads@zepol-ads-beta
```

- Después, reinicia Claude.

Termina con: "Si algo no funciona como esperas, cuéntaselo a Alexis con /zepol-ads:feedback."
