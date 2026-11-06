# Skills de REDCUMBRE

Skills para agentes de código (Claude Code, Codex, Cursor y los demás que leen `SKILL.md`).

## validafirma

[ValidaFirma](https://validafirma.cl) es una plataforma de firma electrónica simple para Chile: tu sistema envía un PDF y los datos de quienes firman, cada firmante verifica su identidad y firma desde su celular, y tu sistema recibe el documento firmado.

Con este skill, tu agente sabe integrar ValidaFirma en tu sistema. El skill es un cargador: le dice al agente qué es ValidaFirma, que instale el CLI (`npm install -g @redcumbrecl/validafirma-cli`), que lea las instrucciones vigentes con `validafirma guia instrucciones` y las reglas que no se negocian. Las instrucciones no viven en este repositorio: el agente las lee en vivo con el CLI, así que cambian cuando cambia ValidaFirma sin que tengas que actualizar el skill.

## Cómo instalarlo

Elige una de estas formas.

### Con `npx skills`

```bash
npx skills add redcumbre/skills --skill validafirma
```

Lo deja en el proyecto en que estás. Agrega `-g` para dejarlo disponible en todos tus proyectos.

### Como plugin de Claude Code

Dentro de Claude Code:

```
/plugin marketplace add redcumbre/skills
/plugin install validafirma@redcumbre
```

### A mano

Copia la carpeta `skills/validafirma` a la carpeta de skills de tu agente:

```bash
git clone https://github.com/redcumbre/skills.git
cp -r skills/skills/validafirma ~/.claude/skills/   # Claude Code
cp -r skills/skills/validafirma ~/.agents/skills/   # Codex y los agentes que leen ~/.agents/skills
```

### Con el CLI de ValidaFirma

Si ya usas el CLI, no necesitas nada de lo anterior: `validafirma login` deja este mismo skill en `~/.claude/skills` y en `~/.agents/skills`, y lo actualiza cuando sale una versión nueva.

## redcumbre

[REDCUMBRE](https://redcumbre.cl) es una API para emitir documentos tributarios electrónicos (facturas, boletas, boletas de honorarios), consultar RUT, enviar mensajes y recibir webhooks.

Con este skill, tu agente sabe integrar tu proyecto con la API de REDCUMBRE usando el CLI `redcumbre` (`@redcumbrecl/cli`): ubica la operación, carga la guía del servicio, lee su contrato, la prueba en el modo de prueba y pasa a producción cuando tú lo confirmas. La referencia del CLI está en [docs.redcumbre.cl/cli](https://docs.redcumbre.cl/cli/).

### Cómo instalarlo

- **Con el CLI (recomendado):** `npx -y @redcumbrecl/cli login` o `redcumbre init` lo instalan, y cada comando lo mantiene al día.
- **Con `npx skills`:** `npx skills add redcumbre/skills --skill redcumbre`.
- **Como plugin de Claude Code:** `/plugin marketplace add redcumbre/skills` y después `/plugin install redcumbre@redcumbre`.
- **A mano:** copia la carpeta `plugins/redcumbre/skills/redcumbre` a la carpeta de skills de tu agente.

Instalada desde este repositorio, sin el CLI, la skill no se renueva sola. Si usas el CLI, `redcumbre login` o `redcumbre init` la instalan y cada comando la mantiene al día; si la copiaste de acá, vuelve a copiarla cuando salga una versión nueva, o corre `redcumbre skill renovar --reemplazar-editada`, que la reemplaza por la de tu CLI.

## Licencia

MIT. Ver [LICENSE](LICENSE).
