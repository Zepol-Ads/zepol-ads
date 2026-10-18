# ZEPOL ADS

Asistente para crear, publicar, medir y optimizar campañas de Meta (Facebook e Instagram) desde Claude. Está hecho para ecommerce con pago contra entrega en Perú, con campañas a WhatsApp o a Shopify.

- Todo se crea como borrador en tu cuenta.
- Nada se publica ni gasta sin tu "sí".
- Funciona con el conector oficial de Meta (`mcp.facebook.com/ads`) y con tu propia cuenta.

Versión 0.1.0 · esqueleto. La mayoría de los pasos todavía esperan al servidor de ZEPOL (ver "Pendientes").

## Comandos

| Comando | Qué hace | Estado |
|---|---|---|
| `/zepol-ads:configurar` | Entrevista corta para crear el perfil de tu negocio | Lista |
| `/zepol-ads:diagnostico` | Estudia tu cuenta: accesos, historial por producto y ganadores | Esqueleto |
| `/zepol-ads:nueva-campana` | Arma una campaña a WhatsApp o a Shopify como borrador y la publica con tu aprobación | Esqueleto |
| `/zepol-ads:verificar` | Compara lo publicado contra la receta de la campaña | Esqueleto |
| `/zepol-ads:reporte` | Métricas y costo por pedido entregado | Esqueleto |
| `/zepol-ads:optimizar` | Propone acciones diarias; no aplica nada sin tu aprobación | Esqueleto |
| `/zepol-ads:limpiar` | Marca borradores obsoletos y reconstruye desde la receta | Esqueleto |

`armado-seguro` no es un comando: son las reglas técnicas que las demás skills siguen cada vez que crean, editan o publican en Meta.

## Qué no está en este repositorio

Este repositorio es público y no contiene la metodología de campañas: estructuras, presupuestos, públicos, textos, umbrales ni reglas de optimización. Todo eso vive en el servidor de ZEPOL y solo llega a quien tiene licencia. Sin servidor, las skills se detienen en esos pasos y lo dicen.

Sí están aquí las reglas técnicas de seguridad del conector de Meta (`skills/armado-seguro`), porque protegen tu cuenta y no dependen de la metodología.

## Requisitos

- Claude Pro o Max, en Claude Code o en la app de Claude.
- El conector oficial de Meta conectado con tu cuenta de Facebook.
- Una cuenta publicitaria de Perú con método de pago.

## Instalación de prueba local (Claude Code)

Agrega esta carpeta como marketplace e instala el plugin:

```bash
claude plugin marketplace add ./plugin
```

```bash
claude plugin install zepol-ads@zepol-ads-beta
```

El primer comando se corre desde la carpeta que contiene `plugin/`. En Windows funciona igual, desde PowerShell o desde la terminal de Claude Code. Después, reinicia Claude Code y escribe `/zepol-ads:configurar`.

Para probar sin instalar, también puedes abrir Claude Code con `claude --plugin-dir ./plugin`.

## Dónde se guardan tus datos

Por ahora, en la carpeta de trabajo, dentro de `zepol/`:

| Archivo | Qué guarda |
|---|---|
| `zepol/perfil-negocio.json` | Perfil del negocio (formato: `skills/configurar/esquemas/perfil-negocio.schema.json`) |
| `zepol/productos/{producto}.json` | Datos de cada producto (formato: `skills/configurar/esquemas/producto.schema.json`) |
| `zepol/diagnostico-{cuenta}.json` | Diagnóstico de la cuenta |
| `zepol/recetas/{campana}.json` | Receta de cada campaña, para verificar y reconstruir |
| `zepol/reportes/{campana}-{fecha}.json` | Reportes |

Ningún archivo guarda contraseñas, claves ni tokens.

## Servidor de ZEPOL

El plugin se conecta a Zepol Cloud, el servidor remoto de ZEPOL, con la configuración de `mcp/zepol-cloud.json`, activada en `.claude-plugin/plugin.json` desde la versión 0.2.0.

- La primera vez, Claude pide iniciar sesión: abre el panel de ZEPOL ADS, el alumno entra con su cuenta y toca **Permitir acceso**.
- Sin una licencia activa, el servidor no entrega la metodología.

Herramientas previstas en el servidor: `metodologia`, `generar_imagen`, `guardar_campana`, `guardar_reporte` y `diagnostico`, más una para guardar el perfil. Los nombres y parámetros finales están por confirmar.

## Pendientes

Las skills marcan cada pendiente en el texto:

- `TODO(servidor)`: depende de Zepol Cloud (metodología, imágenes, alojamiento, guardado en Supabase, licencias).
- `TODO(prueba)`: hay que confirmarlo con una campaña real (rama WhatsApp, lista de mejoras para videos nuevos, mensaje de bienvenida).
- `TODO`: falta definir algo en el plugin (formatos de receta, diagnóstico y reporte; guía de permisos).

Fuera de las skills:

- URL pública del repositorio, licencia y correo de contacto en `plugin.json`.
- Segundo marketplace estable (`zepol-ads`) cuando exista el repositorio en GitHub.
- Guía de instalación para Windows y para la app de Claude.
- Permisos: un plugin no puede traer reglas de permiso, así que la guía de instalación debe explicar qué herramientas de lectura aprobar de una vez.

## Estructura

```
plugin/
  .claude-plugin/
    plugin.json
    marketplace.json      marketplace de prueba local zepol-ads-beta
  mcp/
    zepol-cloud.json      servidor remoto de ZEPOL (Zepol Cloud)
  skills/
    configurar/           entrevista y esquemas de datos
    armado-seguro/        reglas técnicas del conector de Meta
    diagnostico/
    nueva-campana/
    verificar/
    reporte/
    optimizar/
    limpiar/
```
