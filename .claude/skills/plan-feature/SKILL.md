---
name: plan-feature
description: Planifica cómo implementar una nueva funcionalidad o corregir un bug y genera un plan detallado en docs/plans con una lista de tareas de 5-10 minutos. Úsala cuando el usuario pida planificar, diseñar o preparar el trabajo antes de programarlo.
argument-hint: <descripción de la funcionalidad o del bug>
allowed-tools: Read, Glob, Grep, Write, Bash(git log:*), Bash(git status:*), Bash(ls:*), Bash(mkdir:*), Bash(date:*), mcp__slack__slack_list_channels, mcp__slack__slack_post_message
---

# Planificar funcionalidad o bug

Genera un plan de implementación detallado, **sin modificar código**. El único archivo que se crea es el plan en `docs/plans/`.

## Petición del usuario

$ARGUMENTS

## Pasos

1. **Entiende la petición.** Si `$ARGUMENTS` está vacío o es demasiado ambiguo para planificar (no se sabe qué se quiere o, en un bug, cómo se reproduce), pregunta al usuario con `AskUserQuestion`. No preguntes lo que puedas resolver leyendo el código.
2. **Clasifica** el trabajo como `funcionalidad` o `bug`.
3. **Analiza el código actual.** Lee `CLAUDE.md` y los archivos relacionados en `src/` (rutas, controladores, servicios, repositorios, modelos, helpers). Si la petición afecta a endpoints GET de listados, lee también `docs/api/api-pagination.md`. En un bug, identifica la causa raíz antes de proponer la solución.
4. **Diseña la solución** respetando la arquitectura por capas del proyecto (`routes → controllers → services → repositories → BBDD`), ES Modules con extensión `.js` en las importaciones, `node:sqlite` con sentencias preparadas y errores de `helpers/errores.js`.
5. **Descompón en tareas** siguiendo las reglas de más abajo.
6. **Redacta el plan** partiendo de la plantilla [assets/plantilla-plan.md](assets/plantilla-plan.md). Léela y respeta sus secciones y su orden; rellena todas y elimina los marcadores `{{...}}`. Las secciones marcadas "si aplica" que no correspondan se conservan con el texto "No aplica".
7. **Guarda el plan** en `docs/plans/<AAAA-MM-DD>-<slug>.md` (crea la carpeta si no existe):
   - La fecha se obtiene con `date +%F`.
   - `<slug>` en minúsculas, sin tildes, con guiones y precedido de `feat-` o `fix-` según el tipo (p. ej. `2026-10-15-feat-filtro-por-precio.md`).
   - Si ya existe un archivo con ese nombre, no lo sobrescribas: añade un sufijo numérico.
8. **Avisa por Slack** de que el plan está listo para revisar:
   - Obtén el id del canal `planes-ed8` con `mcp__slack__slack_list_channels` (pagina con `cursor` hasta encontrarlo; el id no se asume ni se inventa).
   - Envía con `mcp__slack__slack_post_message` un mensaje en español con el formato: `📝 El plan «<título>» ya está listo para revisar: <ruta del plan> (<n> tareas, ~<total> min).`
   - Si el canal no existe o el envío falla, no interrumpas el flujo: indícalo al usuario en el resumen del paso siguiente.
9. **Resume al usuario** la ruta del plan, el objetivo, el número de tareas y el tiempo total estimado. Pregunta si quiere ajustar algo antes de empezar a implementarlo.

## Reglas para la lista de tareas

- Cada tarea se completa en **5-10 minutos**. Si una tarea parece más larga, divídela; si es menor de 5 minutos, agrúpala con otra relacionada.
- Cada tarea es concreta y verificable: empieza con un verbo, indica el archivo afectado y tiene un resultado observable.
- Usa el formato `- [ ] **T<n>** (~<min> min) — <descripción> · \`<archivo>\``.
- Ordénalas por dependencias, siguiendo el flujo de abajo arriba por capas: modelo/BBDD → repositorio → servicio → controlador → ruta → pruebas manuales → documentación.
- Incluye siempre tareas de verificación (p. ej. comandos `curl`, casos de error) y, si procede, de actualización de la documentación (`CLAUDE.md`, `docs/`).
- Suma los tiempos y escribe el total al final de la sección.

## Restricciones

- No modifiques código fuente, configuración ni otros documentos: solo crea el plan (y envía el aviso a Slack del paso 8).
- Publica en Slack únicamente en el canal `planes-ed8`.
- No hagas commits.
- Redacta el plan y habla con el usuario siempre en español.
