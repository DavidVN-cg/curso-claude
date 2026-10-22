---
name: implement-plan
description: Implementa un plan existente de docs/plans siguiendo su lista de tareas, lo verifica y avisa por Slack al canal planes-ed8 al terminar. Úsala cuando el usuario pida implementar, ejecutar o desarrollar un plan generado con plan-feature.
argument-hint: <ruta o nombre del plan en docs/plans (opcional)>
allowed-tools: Read, Glob, Grep, Write, Edit, Bash(git log:*), Bash(git status:*), Bash(git diff:*), Bash(ls:*), Bash(date:*), Bash(npm:*), Bash(node:*), Bash(curl:*), mcp__slack__slack_list_channels, mcp__slack__slack_post_message
---

# Implementar un plan

Ejecuta, tarea a tarea, un plan de `docs/plans/` generado con `plan-feature`, lo verifica y avisa por Slack cuando termina.

## Plan a implementar

$ARGUMENTS

## Pasos

1. **Localiza el plan.**
   - Si `$ARGUMENTS` es una ruta o un nombre, búscalo en `docs/plans/`.
   - Si está vacío, lista `docs/plans/` y elige el plan con estado `Pendiente` o `En curso`; si hay varios, pregunta al usuario con `AskUserQuestion`.
   - Si no existe ningún plan, avisa al usuario y sugiérele usar `plan-feature`. No inventes un plan.
2. **Lee el plan completo** y `CLAUDE.md`. Revisa `git status` para conocer el estado de partida del repositorio.
3. **Marca el plan como `En curso`** en su cabecera (`- **Estado**: En curso`).
4. **Ejecuta las tareas en orden** (`T1`, `T2`, …):
   - Respeta la arquitectura por capas (`routes → controllers → services → repositories → BBDD`), ES Modules con extensión `.js` en las importaciones, `node:sqlite` con sentencias preparadas y errores de `helpers/errores.js`.
   - Al terminar cada tarea, márcala en el plan cambiando `- [ ]` por `- [x]`.
   - No te salgas del alcance del plan. Si surge algo no previsto que cambia el diseño, detente y consúltalo al usuario.
5. **Verifica** siguiendo la sección "Pruebas y verificación" del plan: arranca la API (`npm start` o `npm run dev` en segundo plano), ejecuta los `curl` indicados, comprueba los casos de error y detén el proceso al acabar. Si algo falla, corrígelo antes de continuar.
6. **Actualiza la documentación** indicada en la sección "Documentación a actualizar" del plan (`CLAUDE.md`, `docs/`), en español.
7. **Cierra el plan**: cambia su estado a `Implementado` (o déjalo `En curso` si quedan tareas sin hacer y explica por qué).
8. **Avisa por Slack** de que la implementación está lista para revisar:
   - Obtén el id del canal `planes-ed8` con `mcp__slack__slack_list_channels` (pagina con `cursor` hasta encontrarlo; el id no se asume ni se inventa).
   - Envía con `mcp__slack__slack_post_message` un mensaje en español con el formato: `✅ La implementación del plan «<título>» ya está lista para revisar (<n>/<total> tareas completadas). Plan: <ruta del plan>.`
   - Si quedaron tareas sin completar o la verificación no pasó, dilo en el mensaje en lugar de afirmar que está todo listo.
   - Si el canal no existe o el envío falla, no interrumpas el flujo: indícalo al usuario en el resumen.
9. **Resume al usuario** qué se ha implementado, los archivos modificados, el resultado de la verificación y si se envió el aviso a Slack. Recuérdale que los cambios no están commiteados y ofrécele usar `/commit`.

## Restricciones

- No hagas commits, `git push` ni cambios en la configuración de git.
- No modifiques archivos ajenos al alcance del plan, salvo el propio plan (checkboxes y estado).
- Publica en Slack únicamente en el canal `planes-ed8`.
- Redacta el código comentado, la documentación y habla con el usuario siempre en español.
