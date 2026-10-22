# {{Título del plan}}

- **Tipo**: {{funcionalidad | bug}}
- **Fecha**: {{AAAA-MM-DD}}
- **Estado**: Pendiente

## 1. Contexto

{{Qué se pide y por qué. En un bug: cómo se reproduce, comportamiento actual y comportamiento esperado.}}

## 2. Objetivo

{{Resultado concreto que se espera al terminar. Una o dos frases.}}

## 3. Alcance

### Incluye

- {{Elemento incluido}}

### No incluye

- {{Elemento fuera de alcance}}

## 4. Análisis del código actual

{{Archivos, capas y funciones relacionados, con rutas (p. ej. `src/services/auth.js`). En un bug, causa raíz identificada.}}

## 5. Diseño de la solución

{{Enfoque elegido y decisiones relevantes, respetando la arquitectura por capas: routes → controllers → services → repositories → BBDD.}}

### Cambios por capa

| Capa | Archivo | Cambio |
| --- | --- | --- |
| {{routes}} | {{src/routes/...}} | {{Descripción}} |

### API (si aplica)

- **Endpoint**: {{MÉTODO /ruta}}
- **Entrada**: {{params, query, body}}
- **Respuesta**: {{código de estado y forma del JSON}}
- **Errores**: {{casos y códigos}}

### Base de datos (si aplica)

{{Cambios en tablas o consultas SQL (sentencias preparadas).}}

## 6. Lista de tareas

Cada tarea debe poder completarse en 5-10 minutos.

- [ ] **T1** (~5 min) — {{Tarea concreta}} · `{{archivo}}`
- [ ] **T2** (~10 min) — {{Tarea concreta}} · `{{archivo}}`

**Tiempo total estimado**: {{N}} min

## 7. Pruebas y verificación

- {{Cómo comprobar que funciona (p. ej. comando `curl` con el resultado esperado)}}
- {{Casos límite y de error}}

## 8. Riesgos y dudas abiertas

- {{Riesgo o pregunta pendiente}}

## 9. Documentación a actualizar

- {{Archivo de `docs/` o sección de `CLAUDE.md` afectada, o "Ninguna"}}
