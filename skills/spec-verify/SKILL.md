---
name: spec-verify
description: Spec-Driven Development (SDD). Verifica y marca los criterios de aceptación de una spec implementada. Agnóstica de stack — Context7 según el framework detectado; browser y visión solo para criterios de UI. Úsala tras /spec-impl.
disable-model-invocation: true
argument-hint: <NN-nombre-spec>
---

# /spec-verify — Verificador de criterios (SDD)

## Rol

Eres un agente verificador de los criterios de aceptación de un archivo de especificación (spec).

Tu labor es revisar, corregir y marcar los checks del "Acceptance criteria" / `Criterios de aceptación` de una spec.

- Usa Context7 para contrastar con la documentación **actual** del framework/runtime **ya fijado** en el proyecto (no asumas Next.js ni ningún otro).
- Usa el MCP de browser de Cursor cuando el criterio implique pantallas o UI web.
- Si el modelo de la sesión soporta visión, úsalo para comparar screenshots con lo descrito en la spec; si no, basate en el snapshot accesible y dilo explícitamente.

El kit es **agnóstico de stack**. Tus respuestas siguen el idioma del prompt del usuario; las etiquetas canónicas de specs en este flujo están en español.

## Contexto de sesión

Antes de la Fase 1, obtén:

**Estado del repositorio:** `git status --short`

**Rama actual:** `git branch --show-current`

**Specs disponibles:**

- Preferido: Glob `specs/**/*.md`
- Alternativa: Unix `find specs -name '*.md' 2>/dev/null`; PowerShell `Get-ChildItem specs -Recurse -Filter *.md -ErrorAction SilentlyContinue`

---

## Resolución de archivo

El argumento recibido es: `$ARGUMENTS`

Igual que `/spec-qa`:

- Ruta directa (`specs/01-auth-session.md`, `specs/001-heat-map/spec.md`)
- Token (`01`, `auth-session`, `01-auth-session`) → plano `specs/NN-slug.md` o carpeta `specs/NN-slug/spec.md`
- Sin argumentos: listar y pedir. Detente.

**Rechazar:** `00-ejemplo-formato` y estado `Obsoleto` (o equivalente). No verificar.

---

## Fases

### Fase 1 — Identificar la spec

Localiza el archivo. Si no existe, lista disponibles y detente. Si es ejemplo de formato u Obsoleto, rechazo estándar.

### Fase 2 — Validar el estado

Lee la línea de estado (`**Estado:**` / `**Status:**`).

| Categoría | Ejemplos | Acción |
| --------- | -------- | ------ |
| Aprobado / Implementado | `Aprobado`, `Approved`, `Implementado`, `Implemented`, … | Continuar |
| Borrador / En revisión / Obsoleto / no reconocido | `Borrador`, `En revisión`, `Obsoleto`, … | Detenerse |

**Mensaje cuando el estado no permite verificar:**

```
❌ No puedo verificar esta spec.

Estado actual: [ESTADO]
Solo verifico specs en estado que signifique "Aprobado" o "Implementado"
(canónico: `Aprobado` / `Implementado`).

Si aún no está implementada, usa /spec-impl tras aprobarla.
Si la spec necesita trabajo de definición, usa /spec o /spec-qa.
```

No ofrezcas “seguir igual”.

### Fase 3 — Avisos de repo (no bloquean) + contexto

1. Mira `git status --short` y `git branch --show-current` (contexto de sesión).
2. Si el working tree **no está vacío**: avisa que hay cambios sin commit. Continúa salvo que el usuario pida parar. **No** hagas stash ni commit.
3. Deriva el nombre esperado de rama: `spec-NN-slug` (del archivo de la spec, sin extensión; si es carpeta `NN-slug/spec.md`, usa `spec-NN-slug`). Si la rama actual **no** coincide: avisa la discrepancia y continúa. **No** hagas `checkout` automático.
4. Lee `AGENTS.md` (sección Stack) y lo que diga la spec sobre tooling / “Cómo verificar”. **No inventes framework.** Si hace falta un stack y no está documentado, detente y pregunta.
5. Extrae los ítems de `## Criterios de aceptación` (compat. `## Acceptance criteria`). Empareja por significado.
6. Clasifica cada criterio: `ui` | `api` | `cli` | `code_docs` | `comando`.

Muestra un resumen breve (ruta, rama, estado, lista de criterios clasificados) y pregunta:

```
¿Empiezo a verificar el criterio 1?
```

Espera confirmación antes de la Fase 4.

### Fase 4 — Verificar uno a uno

Por cada criterio, en orden:

**Según tipo:**

- **`code_docs`:** Context7 (`resolve-library-id` → `query-docs`) con la librería/framework **detectado** en `AGENTS.md` / spec / señales del repo ya acordadas. Contrasta el código real con las recomendaciones actuales. No asumas un framework por defecto.
- **`ui`:** Browser MCP — navegar a la URL/ruta de la spec o de `AGENTS.md`, snapshot y/o screenshot. Con visión, compara la captura con lo descrito; sin visión, usa el snapshot y decláralo. Si la app no está levantada o no hay URL, marca inconcluso y explica qué falta.
- **`api` / `cli` / `comando`:** smoke con los comandos de la spec o `AGENTS.md` (curl, tests, códigos de salida). Sin inventar comandos no documentados.

**Marcado:**

- Cumple con evidencia → `- [x]` en el archivo de la spec (edita solo el checkbox y, si hace falta, una nota breve bajo el ítem o al final en un bloque de verificación).
- Falla o inconcluso → deja `- [ ]` y anota la razón breve.

Tras cada criterio: resume evidencia (pass/fail/inconcluso) y pregunta si continúas con el siguiente.

**Al terminar:**

- Si **todos** pasaron: informa el resultado y pregunta si el humano quiere cambiar `**Estado:**` a `Implementado`. **No** lo cambies sin confirmación explícita.
- Si **alguno** falló o quedó inconcluso: **no** sugieras `Implementado`. Lista los pendientes.

**Nunca** hagas commit automático.

---

## Reglas duras

- **Agnóstico de stack.** Sin default a Next u otro framework. Context7 solo contra la lib detectada; browser solo para UI.
- **Solo checks y notas de verificación.** No reescribas alcance, plan ni decisiones.
- **No commits** ni checkout/stash automáticos.
- **No improvises** stack, URLs ni comandos ausentes de la spec / `AGENTS.md`.
- Rechaza ejemplo de formato y `Obsoleto`.

## Resumen del comportamiento esperado

```
/spec-verify 01-auth-session

  Fase 1  →  Encuentra specs/01-auth-session.md
  Fase 2  →  Estado Aprobado o Implementado → ✅
  Fase 3  →  Avisos WT/rama si aplica; clasifica criterios; pide confirmación
  Fase 4  →  Verifica uno a uno; marca checks; pide confirmación para Implementado

/spec-verify 01-auth-session  (estado: Borrador)

  Fase 2  →  ❌ se detiene
```
