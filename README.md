# Skills de REDCUMBRE

Skills para agentes de código (Claude Code, Codex, Cursor y los demás que leen `SKILL.md`).

## validafirma

[ValidaFirma](https://validafirma.cl) es una plataforma de firma electrónica simple para Chile: tu sistema envía un PDF y los datos de quienes firman, cada firmante verifica su identidad y firma desde su celular, y tu sistema recibe el documento firmado.

Con este skill, tu agente sabe integrar ValidaFirma en tu sistema. El skill es un cargador: le dice al agente qué es ValidaFirma, que instale el CLI (`npm install -g @redcumbrecl/validafirma-cli`), que lea las instrucciones vigentes con `validafirma guia instrucciones` y las reglas que no se negocian. Las instrucciones no viven en este repositorio: el agente las lee en vivo con el CLI, así que cambian cuando cambia ValidaFirma sin que tengas que actualizar el skill.

## Cómo instalarlo

Elige una de estas formas.

### Con `npx skills`

```bash
npx skills add redcumbre/skills
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

## Licencia

MIT. Ver [LICENSE](LICENSE).
