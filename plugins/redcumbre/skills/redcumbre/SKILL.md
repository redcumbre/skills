---
name: redcumbre
description: "Integra un proyecto con cualquier servicio de la API de REDCUMBRE —documentos tributarios electrónicos, boletas de honorarios propias y de terceros, consulta y validación de RUT, verificación de identidad con PINRUT, firma electrónica, SMS, WhatsApp, formularios de Comunicación, clientes y proveedores, reseller y webhooks— usando el CLI redcumbre: ubica la operación, carga la guía del servicio, lee su contrato, pruébala en el modo de prueba y pasa a producción cuando la persona lo confirma."
---

# REDCUMBRE: integrar con el CLI

rCAPI es la plataforma de REDCUMBRE: una sola API para todos sus servicios. Entre otros:

- documentos tributarios electrónicos (facturas, boletas, guías de despacho, notas de crédito y débito);
- boletas de honorarios propias y de terceros, también por carga masiva desde un archivo;
- consulta de un RUT en el SII y validación masiva de RUT;
- verificación de identidad de una persona con PINRUT, y firma electrónica;
- SMS, WhatsApp y los formularios de Comunicación;
- la agenda de clientes y proveedores, la operación de un reseller y sus liquidaciones;
- webhooks para recibir los avisos de todo lo anterior.

Esta skill te enseña a integrar el proyecto de la persona con cualquiera de ellos usando el CLI `redcumbre` (paquete
`@redcumbrecl/cli`). Cada servicio tiene su guía, y la sección 3 dice cómo cargarla.

**Háblale a la persona en español**, salvo que ella te escriba en otro idioma. La skill es parte del CLI y la necesitas para
integrar: el CLI la instala y la mantiene al día, y no le ofreces a la persona quitarla.

Hay **dos modos**, y cada uno tiene su propia llave:

- **Modo de prueba**: lo que emites no llega al SII, no consume folios y no se cobra. Trabajas siempre aquí.
- **Modo de producción**: lo que emites es de producción ante el SII y se cobra según el plan. Pasar a producción lo decide
  una persona, y la sección 4 dice cómo.

El CLI se usa **instalado**: los comandos de esta skill y los que te entrega el CLI dicen `redcumbre …`. Si `redcumbre
--version` no responde, la puerta de entrada es `npx @redcumbrecl/cli init`, que deja el CLI instalado antes de conectar
el proyecto. Mientras el CLI corra por npx, cada respuesta
trae `instalacion` (más abajo dice qué hacer con ella). Si algún comando de esta skill no existe en la versión que tienes,
corre `npx @redcumbrecl/cli@latest instalar --confirmar`, con el sí de la persona (deja al día el CLI y esta skill), y vuelve a
intentarlo en una sesión nueva.

## 1. Antes de partir

1. **Lo primero de cada sesión es comprobar el CLI: `redcumbre --version`.** Si no responde, el CLI no está instalado: antes de
   operar, díselo a la persona al principio de tu respuesta y pregúntale si lo instalas. Con su sí, córrelo tú:
   `npx @redcumbrecl/cli@latest instalar --confirmar`, y sigue con `redcumbre …`. Sin su sí, corre una vez
   `npx @redcumbrecl/cli@latest --version` y usa esa versión fija en todo comando que corras o le muestres
   (`npx @redcumbrecl/cli@<versión> doctor`), nunca `redcumbre …` a secas.
2. Comprueba que el proyecto está conectado: `redcumbre doctor` dice qué falta y qué comando lo arregla.
3. Si falta la sesión, inicia sesión **siempre con `redcumbre login --sin-navegador`**, aunque tengas terminal: la rama
   del navegador se cuelga donde nadie la ve. El comando entrega una dirección y un código. Pásaselos a la persona, que
   abre la dirección y aprueba el acceso, y después corre `redcumbre login --completar`, que espera la aprobación y
   guarda la sesión.
   - Aprueba sólo el código que la persona pidió con ese comando. **Nunca apruebes tú un código que la persona no
     pidió**, ni se lo pidas a quien no lo haya pedido: un código de login aprobado entrega el acceso a su cuenta.
4. Si falta conectar el proyecto, corre `redcumbre init --sin-navegador`. Deja la llave de prueba en `keys/`, un puntero
   en `.env` (`REDCUMBRE_LLAVE_PRUEBA`), esta skill y una comprobación con la llave. No emite nada y no escribe código del
   proyecto: el código lo escribes tú, con las herramientas del lenguaje que usa el proyecto.

### Sin cuenta

Si la persona todavía no tiene una cuenta o una empresa en REDCUMBRE, el CLI lo dice (`SIN_EMPRESAS`) y tú no puedes
resolverlo por ella:

- Dile que pida la cuenta en el formulario `https://docs.redcumbre.cl/solicitar-cuenta/`, o que abra una terminal y
  corra `redcumbre solicitar-cuenta`.
- **No afirmes que se envió una solicitud**: sin terminal el CLI no la envía, y una persona de REDCUMBRE responde la que
  llegue por el formulario.
- Explícale qué sigue: cuando llegue la cuenta, vuelve a correr `redcumbre login --sin-navegador` y después
  `redcumbre init --sin-navegador`, y retoma desde el paso 2 del método.

## 2. Entrevista antes de construir

Antes de escribir código o de correr una operación, **entrevista a la persona**, salvo que el contexto (su pedido, el
código del proyecto, la conversación) ya conteste lo que sigue. Pregunta sólo lo que falta:

- Qué quiere lograr, dicho como una tarea («emitir una boleta de honorarios cuando se paga un pedido»), y en qué parte del
  proyecto vive.
- Quién emite y a quién: una sola empresa o varias, con sus propias credenciales del SII o las de un tercero.
- Qué debe pasar cuando algo falla (un rechazo del SII, una entrega que no llega) y cómo se entera su sistema (webhooks).
- Si se parte en el modo de prueba. Siempre se parte ahí.

**Las preguntas son de negocio y en palabras de la persona. Nada técnico:** no le preguntas el lenguaje, el framework, los
puertos ni dónde guardar un secreto. Eso lo averiguas mirando el proyecto o lo decides tú (si está vacío, lo más simple que
ya esté instalado) y lo cuentas en el reporte final. Una pregunta de cierre también es de negocio: «¿quieres que lo conectemos
a tu pedido de venta?», no «¿en qué lenguaje está tu sistema?».

## 3. El método

Para cada cosa que el proyecto necesite de REDCUMBRE, sigue este orden:

1. **Ubicar** la operación: `redcumbre buscar "<lo que quieres hacer>"` (si tu versión del CLI aún no trae `buscar`, usa
   `redcumbre api buscar "<texto>"`). Busca por lo que se quiere hacer («emitir una boleta de honorarios»), sin datos de
   una persona: la búsqueda encuentra operaciones por la tarea, y un RUT, un nombre, un correo o un teléfono no ayudan.
2. **Leer** su contrato: `redcumbre api describir <operación>` muestra parámetros, cuerpo, efecto y cobro. No inventes
   campos: usa los del contrato.
3. **Ejecutar** con `redcumbre api <operación>`, **primero con `--ensayo`**, que muestra la petición sin hacerla.
   Recién después, sin `--ensayo`, con la llave de prueba. Pasa los datos con `--param clave=valor` y `--cuerpo`.
   Una escritura repetida usa la misma `--clave` de idempotencia.
4. **Cargar las fichas** de cada servicio con `redcumbre guia`: sin tema lista los temas disponibles, y
   `redcumbre guia <tema>` trae la ficha de ese servicio o mecánica (emisores, emisión, webhooks, modo de prueba y
   producción…) en Markdown, sin pedir sesión ni llave. Explican las reglas del servicio, los errores comunes y qué configurar
   antes de usarlo; léelas antes de escribir código para un servicio nuevo. Con `--json` o sin terminal la ficha viene en
   el campo `contenido`. Sin red el CLI entrega su última copia y dice la fecha de esa copia.
5. **Verificar** el sitio de la persona con
   `redcumbre prueba verificar --reenviar-a <url de su receptor> --efectos "<comando>"`: comprueba que recibe los
   webhooks, valida la firma y no repite el efecto de una entrega duplicada. `--efectos` es un comando del proyecto que
   imprime cuántas veces se aplicó el efecto (un entero); sin él no se puede comprobar la deduplicación. Las dos son
   obligatorias.

Otros comandos útiles en el modo de prueba: `redcumbre escenarios` (emisores y RUT de prueba), `redcumbre eventos` y
`redcumbre eventos disparar <evento>`, `redcumbre webhooks escuchar --reenviar-a <url>`, `redcumbre bandeja` (los
correos y mensajes que la prueba retuvo) y `redcumbre restaurar` (limpia lo emitido en prueba; pide confirmar).

Con `--json` el CLI no pregunta y deja un solo JSON en stdout; el código de salida dice qué pasó (0 listo, 3 falta una
confirmación, 4 falta la sesión, 5 la llave falla, 6 la API respondió un error 4xx, 7 la API no respondió, 9 la
verificación falló). Cuando un error trae `comando`, es lo que hay que correr a continuación.

**Si una respuesta trae `actualizacion`** (hay una versión más nueva del CLI), empieza tu respuesta, antes de lo que te
pidieron, preguntándole a la persona si lo actualizas, con la versión que tiene y la disponible. Actualizar instala un
paquete en su computador, así que lo decide ella: **con su sí**, córrelo tú con `actualizacion.comando` (`redcumbre
actualizar --confirmar`, o su forma por npx), que deja al día el CLI y esta skill. Sin su sí, sigue con la versión que tiene
y no vuelvas a preguntar en la misma conversación. Si un aviso dice que la skill está desactualizada y tiene cambios, no la
reemplaces por tu cuenta.

**Si una respuesta trae `instalacion`**, el CLI está corriendo por npx. Con `instalado: false`, empieza tu respuesta, antes
de lo que te pidieron, preguntándole a la persona si lo dejas instalado en su computador: instala un paquete, así que lo
decide ella. **Con su sí**, córrelo tú con `instalacion.comando` (`npx @redcumbrecl/cli@<versión> instalar --confirmar`) y
desde ahí usa `redcumbre …`; no le pidas que lo corra ella. Mientras no esté instalado, todo comando que corras o le
muestres a la persona va con `instalacion.prefijo` en lugar de `redcumbre` (`npx @redcumbrecl/cli@<versión> login
--sin-navegador`), también los que copias de esta skill: con la versión fija, nunca `redcumbre …` a secas ni
`npx @redcumbrecl/cli` sin versión o con `-y`, que bajan otra versión sin que la persona lo decida. Si dijo que no, no
vuelvas a preguntar en la misma conversación.
Con `instalado: true`, el CLI ya está instalado: deja npx y usa `redcumbre …`.

### Cómo le hablas a la persona

- **Cada paso que le pides lleva su comando y el resultado que debe ver.** «Corre `redcumbre doctor`; debería decir que la
  sesión, la empresa y la llave de prueba están en orden» sirve; «revisa que todo esté bien» no.
- **Termina siempre con un reporte final**, aunque algo haya fallado: qué quedó hecho, qué comandos corriste y con qué
  resultado, qué falta y quién lo hace, y en qué modo quedó el proyecto (prueba o producción).

**Servicios locales:** si levantas un receptor de webhooks u otro servicio en la máquina de la persona, elige un puerto alto
que compruebes libre (`ss -lnt`) y **nunca mandes avisos ni pruebas a un puerto que no abriste tú**: otro programa de su
equipo podría estar escuchando ahí.

## 4. La llave y la carpeta `keys/`

- La llave de prueba vive en `keys/redcumbre-prueba.json`; la de producción, cuando exista, en
  `keys/redcumbre-produccion.json`. El `.env` sólo trae un puntero al archivo, nunca el valor.
- **No leas ni imprimas el contenido de `keys/`**, ni lo copies a otro archivo, a un log, a un mensaje o a un commit.
  Para usar la llave, deja que el CLI la lea (`redcumbre api …`) o que tu código la lea desde la ruta que apunta
  `REDCUMBRE_LLAVE_PRUEBA`.
- El límite, dicho con franqueza: el CLI cuida que la llave de producción no salga en su salida, en el historial ni en
  los logs, pero la llave **también queda en `keys/`**, y un agente con acceso al disco puede leerla. Que no la leas es
  una regla de conducta tuya, no un control técnico. Cúmplela.
- Nunca subas `keys/` ni `.env` al control de versiones; el CLI ya los agrega a `.gitignore`.

## 5. Pasar a producción

Pasar a producción crea la llave de producción de la empresa. Lo que se emita con ella **es de producción ante el SII y
se cobra según tu plan**.

**Antes de correr el comando, díselo a la persona con esas palabras** y espera su sí: que confirme que quiere pasar. No lo
corras por tu cuenta ni porque "ya funciona en prueba".

**Al terminar en modo de prueba**, no cierres sin esto: presenta a la persona lo que la ficha del producto dice en «Antes de
pasar a producción» (qué cambia, qué cuesta, qué falta de su lado) y pregúntale si quiere pasar. Si no quiere, el proyecto
queda en modo de prueba y lo dices en el reporte final.

Con la confirmación de la persona:

```
redcumbre pasar-a-produccion --confirmar <slug-de-la-empresa> [--servicios <lista>] [--instalar "<comando>"] [--comprobar "<comando>"]
```

- `--confirmar` lleva el slug de la empresa (el mismo de `redcumbre doctor`); con otro slug el CLI no hace nada.
- Para activar los servicios que usará en producción, `--servicios` los lista separados por coma.
- `--instalar "<comando>"` corre tu comando con el valor de la llave sólo en la variable de entorno
  `REDCUMBRE_VALOR_LLAVE_PRODUCCION` de ese proceso (sin pasarla por la línea de comandos, el `.env` ni la salida). Tu
  comando la entrega a donde el proyecto guarda sus secretos. `--comprobar "<comando>"` corre después, sin la variable, y
  tiene que imprimir el `whoami` de la empresa en producción.
- Si el CLI sale 3, pide una confirmación: relee su `error.comando`.
- Después de pasar, el proyecto sigue usando la llave de prueba. Una llamada a producción con `redcumbre api` lleva
  `--produccion` (y `--confirmar <slug>` en lo que escribe o cobra), y **sólo la haces cuando la persona te lo pide**.

## 6. Alta de emisor DTE y certificación SII

Un RUT puede tener **varios emisores DTE, uno por carril**, y cada carril se certifica por separado ante el SII:

- **`FULL_DTE`**: facturación, integrado: emite y recibe facturas y los demás DTE de facturación (33, 34, 43, 46, 52,
  56 y 61). Es el recomendado. **No emite boletas.**
- **`PORTAL_MIPYME`**: facturación para bajo volumen y sin cargo fijo. **No emite boletas** y **no se certifica**.
- **`SOLO_BOLETA`**: el emisor de boletas (39 y 41), un carril aparte del de facturación. **Las boletas se emiten
  sólo desde aquí**: ni pasar a `FULL_DTE` ni migrar da boletas.

**Esta decisión es de la persona, no tuya: no eliges el carril.** Antes de crear o certificar nada, sigue estos pasos:

1. **Pregunta qué va a emitir**: boletas, facturas o ambas. Si son facturas, explícale `FULL_DTE` (recomendado, emite y
   recibe DTE integrado) frente a `PORTAL_MIPYME` (bajo volumen y sin cargo fijo, sin boletas), y espera que ella elija.
2. **Lista los emisores del RUT y su carril**: `redcumbre emisores dte listar --rut <rut>`. Mira `tipoIntegracion` de cada uno.
3. **Boletas.** Si el RUT no tiene un emisor `SOLO_BOLETA`, créalo con `redcumbre tokenizar --rut <rut> --tipo dte --boletas`
   (con facturas además, suma `--integracion <FULL_DTE|PORTAL_MIPYME>`). Si ya lo tiene y aún no tiene los tipos 39 y 41,
   pídelos con `redcumbre emisores dte certificacion certificar-tipos <id> --tipos 39,41 --correo <correo> --telefono <teléfono>`.
4. **Facturas.** Crea el emisor con `redcumbre tokenizar --rut <rut> --tipo dte --integracion <la elegida>`. `PORTAL_MIPYME`
   no se certifica. Pasar de `PORTAL_MIPYME` a `FULL_DTE` es **migrar** (`redcumbre emisores dte migrar`), no certificar.
   Cuando el RUT termina su certificación de facturas, sus emisores `PORTAL_MIPYME` pasan solos a `FULL_DTE` en todas las
   empresas que lo usan. Desde ahí **ese RUT ya no puede crear un emisor `PORTAL_MIPYME`**: `tokenizar` responde `409`
   `RUT_FACTURA_CON_CERTIFICACION_PROPIA`. No lo rodees; explícale a la persona que su RUT factura con certificación propia
   (lo mismo que «software propio»: el SII no le deja usar el Portal MiPyme) y ofrécele las dos salidas, para que elija:
   **a)** facturar con `FULL_DTE`, que queda operativo de inmediato, sin volver a certificar; o **b)** si de verdad quiere
   el Portal MiPyme, volver su emisor `FULL_DTE` al Portal MiPyme (paso 7), con todo lo que eso implica. Las boletas
   (`SOLO_BOLETA`) no dependen de esto.
5. **Sigue el avance con `redcumbre emisores dte solicitudes <id>`.** Una respuesta exitosa de `certificar-tipos` sólo
   significa que la solicitud quedó registrada: **no que el SII resolvió**. La revisa REDCUMBRE y el avance se lee en
   las solicitudes. El carril del emisor decide qué tipos se pueden pedir (boletas o facturas).
6. **El certificado digital NUNCA pasa por ti.** No pidas el archivo PFX ni su contraseña, ni los aceptes si la persona
   te los ofrece. Entrégale el enlace de autorización que muestra `tokenizar`, o usa `--invitar <correo>` para que
   le llegue a quien corresponda, y espera con `tokenizar --espera <minutos>` o con el webhook.
7. **Volver al Portal MiPyme (`redcumbre emisores dte volver-a-mipyme <id> --rut <rut>`) sólo si la persona lo pide
   expresamente.** Inscribe el RUT en el Portal MiPyme ante el SII y, en **todas** las empresas que usan ese RUT, sus
   emisores `FULL_DTE` pasan a `PORTAL_MIPYME`, la certificación propia de facturas se pierde y se anulan sus folios de
   facturación de producción. Volver a certificarse puede tener costo. Las boletas no cambian. Antes de correrlo, léele
   ese efecto y pídele que te escriba el RUT del emisor para confirmar: ese RUT es el que va en `--rut`, y si no coincide
   el API responde `400`. Nunca lo propongas como arreglo de otro problema: sólo cuando ella quiere el Portal MiPyme.

## 7. Firma electrónica o digital

Si la persona pide firma electrónica o digital de documentos, **esta skill no la resuelve**. Recomiéndale instalar el MCP
de ValidaFirma con su skill `validafirma`, que es quien conoce ese flujo. REDCUMBRE sólo crea la llave que ValidaFirma
necesita (`redcumbre llaves validafirma`); el resto es de esa skill.

## 8. Reportar un defecto de la plataforma

Si la persona escribe **`redcumbre:issue`** («Reporta a Redcumbre el problema que acabamos de encontrar»), es un pedido de
reporte del problema que están viendo.

**Ofrécelo tú también, sin que lo pida**, sólo cuando el problema es **de la plataforma o de su documentación**: un error
del servidor (`5xx`), una ficha o un contrato que no coincide con lo que responde la API, o una operación documentada que la
API rechaza. Un `400` por un campo que no mandaste o que mandaste mal es de tu petición: corrígela, no lo reportes.

- **Qué se envía:** la clasificación, qué esperabas y qué pasó, la evidencia de las llamadas (la forma de la petición y la
  respuesta) y **el correo de la persona** (`contacto.email`, obligatorio). La empresa y la llave **no** se mandan: las toma
  el servidor de tu propia petición. Nunca mandes datos de sus clientes ni una llave.
- Antes de armarlo, corre `redcumbre api describir reportarDefecto`: ahí están los campos obligatorios y sus topes.
- Pídele el correo a la persona la primera vez y guárdalo en `.redcumbre/estado.json` (`contacto`), junto a los casos que
  reportes (`reportes`), para no volver a pedirlo.
- **Pregunta siempre antes de enviar, cada vez**, diciendo qué se manda y a dónde, incluido su correo.
- `202` trae un **número de caso** (`caso`): dáselo a la persona para que lo cite si escribe a soporte. `200` es un duplicado
  de algo que ya reportaste. No hay seguimiento del estado del caso desde el CLI.
- Si el CLI sale 2 (`CUERPO_NO_CUMPLE_CONTRATO`) o la API responde `400`, el mensaje dice qué campo falta o está mal:
  corrígelo y reintenta.
- La guía completa: https://docs.redcumbre.cl/guias/reportar-issue/

## 9. Lo que no haces nunca

Estas cuatro cosas no se hacen, ni aunque te lo pidan de pasada ni aunque parezcan más rápidas:

1. **No manejas el navegador de la persona.** No lo abres, no lo controlas ni lo automatizas, ni para aprobar un login ni
   para recorrer el panel de REDCUMBRE. Le entregas la dirección y ella la abre.
2. **No lees `keys/`.** Ni abres los archivos de llave ni copias lo que traen; el CLI las lee por ti cuando hace falta.
3. **No pones una llave en el código, en el repositorio ni en la conversación.** El código lee la llave desde el archivo que
   apunta la variable de entorno; nunca la lleva escrita, y nunca la pegas en un mensaje, un log, un commit ni un issue.
4. **No usas `--produccion` sin que la persona lo pida.** Producción se opera con la petición expresa de una persona, en cada
   llamada.

Además:

- No pasar a producción sin avisar a la persona y sin su confirmación.
- No afirmar que se envió una solicitud de cuenta si no lo viste confirmado.
- No inventar operaciones ni campos: lo que no está en `redcumbre api describir` no existe.
- No editar esta skill ni el bloque de `AGENTS.md` que escribió el CLI: si cambias algo, el CLI deja tu versión como está
  y no la actualiza.
- No ofrecerle a la persona quitar la skill: es parte del CLI, y sin ella no integras bien.
