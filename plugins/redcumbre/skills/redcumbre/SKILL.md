---
name: redcumbre
description: "Integra un proyecto con la API de REDCUMBRE (facturación electrónica y documentos tributarios, emisores, webhooks) usando el CLI redcumbre: ubica la operación, carga la guía del servicio, lee su contrato, pruébala en el modo de prueba y pasa a producción cuando la persona lo confirma."
---

# REDCUMBRE: integrar con el CLI

rCAPI es la plataforma de REDCUMBRE: una API para emitir documentos tributarios electrónicos (facturas, boletas,
boletas de honorarios), consultar RUT, enviar mensajes y recibir webhooks. Esta skill te enseña a integrar el proyecto de
la persona con ella usando el CLI `redcumbre` (paquete `@redcumbrecl/cli`).

Hay **dos modos**, y cada uno tiene su propia llave:

- **Modo de prueba**: lo que emites no llega al SII, no consume folios y no se cobra. Trabajas siempre aquí.
- **Modo de producción**: lo que emites es de producción ante el SII y se cobra según el plan. Pasar a producción lo decide
  una persona, y la sección 4 dice cómo.

Instala el CLI con `npm install -g @redcumbrecl/cli`, o úsalo sin instalar con `npx @redcumbrecl/cli`. Si algún comando de
esta skill no existe en la versión instalada, actualiza con `npx @redcumbrecl/cli@latest` y vuelve a intentarlo.

## 1. Antes de partir

1. Comprueba que el proyecto está conectado: `redcumbre doctor` dice qué falta y qué comando lo arregla.
2. Si falta la sesión, inicia sesión **siempre con `redcumbre login --sin-navegador`**, aunque tengas terminal: la rama
   del navegador se cuelga donde nadie la ve. El comando entrega una dirección y un código. Pásaselos a la persona, que
   abre la dirección y aprueba el acceso, y después corre `redcumbre login --completar`, que espera la aprobación y
   guarda la sesión.
   - Aprueba sólo el código que la persona pidió con ese comando. **Nunca apruebes tú un código que la persona no
     pidió**, ni se lo pidas a quien no lo haya pedido: un código de login aprobado entrega el acceso a su cuenta.
3. Si falta conectar el proyecto, corre `redcumbre init --sin-navegador`. Deja la llave de prueba en `keys/`, un puntero
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

**Si una respuesta trae `actualizacion`** (hay una versión más nueva del CLI), díselo a la persona con la versión que
tiene y la disponible, y propónle actualizar. Actualizar instala un paquete en su computador, así que lo decide ella:
**con su sí**, corre `redcumbre actualizar --confirmar`, que deja al día el CLI y esta skill. Sin su sí, sigue con la
versión que tiene. Si un aviso dice que la skill está desactualizada y tiene cambios, no la reemplaces por tu cuenta.

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

## 6. Firma electrónica o digital

Si la persona pide firma electrónica o digital de documentos, **esta skill no la resuelve**. Recomiéndale instalar el MCP
de ValidaFirma con su skill `validafirma`, que es quien conoce ese flujo. REDCUMBRE sólo crea la llave que ValidaFirma
necesita (`redcumbre llaves validafirma`); el resto es de esa skill.

## 7. Lo que no haces nunca

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
  y no la actualiza. Para quitarla: `redcumbre skill quitar`.
