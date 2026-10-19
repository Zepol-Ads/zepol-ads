# ZEPOL ADS

Asistente para crear, publicar, medir y optimizar campañas de Meta (Facebook e Instagram) desde Claude. Está hecho para ecommerce con pago contra entrega en Perú, con campañas a WhatsApp o a Shopify.

- Todo se crea como borrador en tu cuenta.
- Nada se publica ni gasta sin tu "sí".
- Funciona con el conector oficial de Meta (`mcp.facebook.com/ads`) y con tu propia cuenta.

Versión 0.5.0. Empieza con un solo comando: `/zepol-ads:empezar` lee lo que llenaste en la web de ZEPOL, encuentra tus activos de Meta, estudia tu cuenta y te propone tu primera campaña. Todos los comandos usan Zepol Cloud: el perfil, los productos, la metodología, las imágenes, las recetas, los diagnósticos y los reportes viven en el servidor. Falta la prueba en vivo de punta a punta.

## Comandos

| Comando | Qué hace |
|---|---|
| `/zepol-ads:empezar` | **Empieza aquí.** Arranque completo: conexiones, tu negocio, tus activos de Meta, el estudio de tu cuenta, tu plan de arranque y el borrador de tu primera campaña |
| `/zepol-ads:configurar` | Cambia los datos de tu negocio con una entrevista corta |
| `/zepol-ads:diagnostico` | Estudia tu cuenta (accesos, historial por producto y ganadores) y guarda el diagnóstico |
| `/zepol-ads:nueva-campana` | Arma una campaña a WhatsApp o a Shopify como borrador, guarda su receta y la publica con tu aprobación |
| `/zepol-ads:verificar` | Compara lo publicado contra la receta de la campaña; adopta como receta una campaña creada por tu cuenta |
| `/zepol-ads:reporte` | Métricas y costo por pedido entregado; guarda el reporte |
| `/zepol-ads:optimizar` | Propone acciones diarias, no aplica nada sin tu aprobación y guarda lo que se decidió |
| `/zepol-ads:limpiar` | Marca borradores obsoletos y reconstruye desde la receta |
| `/zepol-ads:feedback` | Le envía a Alexis tu comentario, una idea o un problema |
| `/zepol-ads:novedades` | Novedades de ZEPOL y el estado de tu cuenta (licencia, imágenes y versión) |

`armado-seguro` no es un comando: son las reglas técnicas que las demás skills siguen cada vez que crean, editan o publican en Meta.

## Qué no está en este repositorio

Este repositorio es público y no contiene la metodología de campañas: estructuras, presupuestos, públicos, textos, umbrales ni reglas de optimización. Todo eso vive en el servidor de ZEPOL y solo llega a quien tiene licencia. Sin servidor, las skills se detienen en esos pasos y lo dicen.

Sí están aquí las reglas técnicas de seguridad del conector de Meta (`skills/armado-seguro`), porque protegen tu cuenta y no dependen de la metodología.

## Requisitos

- Claude Pro o Max, en Claude Code o en la app de Claude.
- El conector oficial de Meta conectado con tu cuenta de Facebook.
- Una cuenta publicitaria de Perú con método de pago.

## Instalación (canal beta)

**En la app de Claude:**

1. Entra a Customize → Plugins → Add → Add marketplace, pega `Zepol-Ads/zepol-ads` y toca Add.
2. Instala **zepol-ads** y enciende "Sync automatically" para recibir las actualizaciones.
3. En el plugin, abre la pestaña Conectores, toca **Conectar** en ZEPOL ADS y luego **Permitir acceso** con la cuenta de tu licencia.
4. En Customize → Connectors, conecta **Meta Ads** con tu cuenta de Facebook.

**En Claude Code:**

```bash
claude plugin marketplace add Zepol-Ads/zepol-ads
```

```bash
claude plugin install zepol-ads@zepol-ads-beta
```

Después, reinicia Claude y escribe `/zepol-ads:empezar`. La primera vez, Claude te pide iniciar sesión en ZEPOL ADS con la cuenta de tu licencia.

Para probar sin instalar, desde una copia local: `claude --plugin-dir ./plugin`.

## Dónde se guardan tus datos

En Zepol Cloud, el servidor de ZEPOL, con la cuenta de tu licencia: perfil del negocio (con su diagnóstico), productos (con sus fotos), recetas de campaña, reportes, creativos y pedidos de WhatsApp. Cada alumno solo ve sus propios datos. El plugin no guarda archivos locales.

Ningún dato guarda contraseñas, claves ni tokens. La clave de OpenAI se guarda cifrada desde el panel.

## Servidor de ZEPOL

El plugin se conecta a Zepol Cloud, el servidor remoto de ZEPOL, con la configuración de `mcp/zepol-cloud.json`, activada en `.claude-plugin/plugin.json` desde la versión 0.2.0.

- La primera vez, Claude pide iniciar sesión: abre el panel de ZEPOL ADS, el alumno entra con su cuenta y toca **Permitir acceso**.
- Sin una licencia activa, el servidor no entrega la metodología.

Herramientas de Zepol Cloud que usan las skills:

- perfil y productos: `obtener_perfil`, `guardar_perfil`, `guardar_producto` y `guardar_diagnostico`;
- metodología: `obtener_metodologia` y `evaluar_conjuntos`;
- campañas: `generar_imagen`, `guardar_receta`, `listar_recetas` y `obtener_receta`;
- resultados: `guardar_reporte`, `listar_reportes`, `registrar_pedidos_whatsapp` y `obtener_pedidos_whatsapp`;
- cuenta: `estado_cuenta`, `obtener_novedades`, `registrar_feedback` y `registrar_error`.

Los formatos de las recetas, los reportes y los diagnósticos los define el servidor y llegan con cada herramienta.

## Pendientes

Las skills marcan cada pendiente en el texto:

- `TODO(prueba)`: hay que confirmarlo con una campaña real (rama WhatsApp, lista de mejoras para videos nuevos, mensaje de bienvenida).
- `TODO`: guía de permisos.

Fuera de las skills:

- URL pública del repositorio, licencia y correo de contacto en `plugin.json`.
- Segundo marketplace estable (`zepol-ads`) para la venta.
- Al subir la versión, cámbiala en `plugin.json`, en `marketplace.json` y en el `version_plugin` que las skills envían a `estado_cuenta` (`empezar`, `novedades`, `nueva-campana`, `optimizar`, `reporte`, `verificar` y `limpiar`).
- `empezar` enlaza a la bienvenida del panel en `zepol-ad-beacon.lovable.app`: cámbialo cuando el panel pase a `zepolads.com`.
- Probar si el conector de Meta (`https://mcp.facebook.com/ads`) puede venir dentro del plugin: pide el ID de la app de Meta, así que no está comprobado.
- Guía de instalación para Windows y para la app de Claude.
- Permisos: un plugin no puede traer reglas de permiso, así que la guía de instalación debe explicar qué herramientas de lectura aprobar de una vez.

## Estructura

```
plugin/
  .claude-plugin/
    plugin.json
    marketplace.json      marketplace beta zepol-ads-beta
  mcp/
    zepol-cloud.json      servidor remoto de ZEPOL (Zepol Cloud)
  skills/
    empezar/              arranque completo (empieza aquí)
    configurar/           entrevista del perfil del negocio
    armado-seguro/        reglas técnicas del conector de Meta
    diagnostico/
    nueva-campana/
    verificar/
    reporte/
    optimizar/
    limpiar/
    feedback/
    novedades/
```
