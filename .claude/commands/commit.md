---
description: Crea un commit siguiendo la especificación Conventional Commits
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*)
argument-hint: [pista opcional sobre el tipo o alcance del commit]
---

# Crear commit (Conventional Commits)

Crea un commit de los cambios actuales siguiendo la especificación [Conventional Commits 1.0.0](https://www.conventionalcommits.org/es/v1.0.0/).

## Contexto

- Estado del repositorio: !`git status --short`
- Cambios en el área de preparación (staged): !`git diff --cached`
- Cambios sin preparar: !`git diff`
- Últimos commits (para seguir el estilo): !`git log --oneline -10 2>/dev/null || echo "(aún no hay commits)"`

## Pista del usuario

$ARGUMENTS

## Formato del mensaje

```
<tipo>[ámbito opcional][!]: <descripción>

[cuerpo opcional]

[pie(s) opcional(es)]
```

### Tipos permitidos

- `feat`: nueva funcionalidad.
- `fix`: corrección de un error.
- `docs`: cambios solo en documentación.
- `style`: formato que no afecta al código (espacios, comas, etc.).
- `refactor`: cambio de código que no corrige un error ni añade funcionalidad.
- `perf`: mejora de rendimiento.
- `test`: añadir o corregir tests.
- `build`: cambios en el sistema de build o dependencias (`package.json`, etc.).
- `ci`: cambios en la configuración de integración continua.
- `chore`: tareas de mantenimiento que no tocan `src` ni tests.
- `revert`: revierte un commit anterior.

### Reglas

1. El **ámbito** es opcional y va entre paréntesis; usa el recurso o capa afectada (p. ej. `auth`, `vendehumos`, `db`, `routes`).
2. La **descripción** va en español, en imperativo o infinitivo según el estilo del historial, en minúscula, sin punto final y con un máximo aproximado de 72 caracteres en la primera línea.
3. El **cuerpo** (opcional) va separado por una línea en blanco; explica el *qué* y el *porqué*, no el *cómo*.
4. Los **cambios incompatibles** se marcan con `!` tras el tipo/ámbito (`feat(auth)!: ...`) y/o con un pie `BREAKING CHANGE: <explicación>`.
5. Los **pies** usan el formato `Clave: valor` (p. ej. `Refs: #12`, `Closes: #34`).
6. Si el sistema indica una línea de atribución (`Co-Authored-By`) para los commits, añádela como último pie.

## Pasos

1. Analiza el estado y los diffs anteriores. Si no hay ningún cambio, avisa al usuario y no hagas nada más.
2. Si hay cambios sin preparar y nada en staging, decide qué archivos forman un cambio coherente y prepáralos con `git add <archivo>` (nombrando cada archivo; **nunca** `git add -A` ni `git add .`). No añadas archivos sensibles (`.env`, credenciales, bases de datos en `data/`).
3. Si los cambios mezclan varios propósitos independientes, propón dividirlos en varios commits y crea uno por cada propósito.
4. Elige el `tipo` y el `ámbito` que mejor describan el cambio, teniendo en cuenta la pista del usuario si la hay.
5. Redacta el mensaje y muéstraselo al usuario utilizando la tool `AskUserQuestion` para preguntarle si le parece bien el mensaje. Si no le parece bien, genera otro mensaje y le vuelves a preguntar. Cuando lo dé por válido, créalo con `git commit` usando un heredoc para respetar los saltos de línea:

   ```bash
   git commit -m "$(cat <<'EOF'
   feat(auth): añade endpoint de perfil

   Requiere token Bearer y devuelve los datos del usuario autenticado.
   EOF
   )"
   ```

6. Ejecuta `git status` para confirmar que el commit se ha creado y muestra al usuario el mensaje final.

## Restricciones

- No hagas `git push`, ni `--amend`, ni `--no-verify`, ni modifiques la configuración de git.
- Si un hook de pre-commit falla, corrige el problema y crea un commit **nuevo**; no uses `--amend`.
- Responde siempre en español.
