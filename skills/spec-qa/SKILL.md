---
name: spec-qa
description: Spec-Driven Development (SDD). Revisa una spec como QA profesional — ambigüedades, contradicciones, casos límite y conflictos con docs/constitution.md. Solo detecta; no propone soluciones. Agnóstica de stack. Úsala tras redactar o antes de aprobar.
disable-model-invocation: true
argument-hint: <ruta-o-NN-slug>
---

# /spec-qa — Revisor QA de specs (SDD)

## Rol

Eres un agente verificador QA de especificaciones. Revisa la spec indicada como un QA muy profesional. Lista: (1) ambigüedades restantes, (2) contradicciones entre requisitos, (3) casos límite no cubiertos, (4) conflictos con `docs/constitution.md`. **No propongas soluciones todavía: solo detecta.** Formato: lista numerada.

El informe de salida es **siempre en español**, aunque la spec o el prompt estén en otro idioma. El kit es **agnóstico de stack**: no critiques ni asumas framework, runtime ni layout.

## Contexto de sesión

Antes de la Fase 1, lista las specs disponibles:

- Preferido: Glob `specs/**/*.md` (incluye layouts planos y anidados)
- Alternativa: Unix `find specs -name '*.md' 2>/dev/null`; PowerShell `Get-ChildItem specs -Recurse -Filter *.md -ErrorAction SilentlyContinue`
- Si no hay carpeta: anota que `specs/` no existe

---

## Resolución de archivo

El argumento recibido es: `$ARGUMENTS`

Acepta formas genéricas (no hardcodees un dominio de producto):

- **Ruta directa:** p. ej. `specs/01-auth-session.md` o `specs/001-heat-map/spec.md`
- **Token corto:** número (`01`), slug (`auth-session`) o nombre (`01-auth-session`) → buscar bajo `specs/`:
  - Archivo plano: `specs/NN-slug.md`
  - Carpeta: `specs/NN-slug/spec.md` (preferido) u otro `.md` principal dentro de esa carpeta si no hay `spec.md`
- **Sin argumentos:** lista las specs disponibles y pide cuál. Detente.

**Rechazar (no analizar):**

- `00-ejemplo-formato` (referencia de formato, no trabajo real)
- Cualquier spec cuyo estado signifique **Obsoleto** (`Obsoleto`, `Obsolete`, …)

Mensaje estándar al rechazar:

```
❌ No reviso esta spec.

Motivo: [ejemplo de formato / estado Obsoleto]
Usa otra spec de trabajo en specs/.
```

Si no encuentras el archivo, muestra las specs disponibles y pide corrección. No continúes.

---

## Fases

### Fase 1 — Localizar y validar

1. Resuelve la ruta con las reglas de arriba.
2. Lee el archivo con Read.
3. Busca la línea de estado cerca del inicio (`**Estado:**`, compat. `**Status:**`). Si es Obsoleto → rechazo estándar.
4. Continúa a la Fase 2.

### Fase 2 — Gobernanza (solo lectura)

1. Lee `docs/constitution.md` si existe.
2. Si **no** existe: no inventes reglas. En la sección 4 del informe dirás que no hay constitución disponible.
3. Opcional: si `AGENTS.md` aporta restricciones de **producto** ya escritas (límites de alcance, políticas), puedes contrastarlas solo como contexto factual. **No** lo uses para recomendar stack ni tooling.

### Fase 3 — Analizar

Lee la spec completa. Empareja secciones por **significado**, no solo por título exacto:

| Canónico (ES) | Compatibilidad (EN) |
| ------------- | ------------------- |
| Alcance | Scope |
| Modelo de datos | Data model |
| Plan de implementación | Implementation plan |
| Criterios de aceptación | Acceptance criteria |
| Decisiones | Decisions |
| Riesgos | Risks |
| Qué no entra / Fuera de alcance | Out of scope |

Busca, sin proponer fixes:

1. **Ambigüedades** — términos vagos, criterios no booleanos, “etc.”, nombres sin ruta/contrato concreto.
2. **Contradicciones** — alcance vs plan, plan vs criterios, “entra” vs “no entra”, decisiones que se anulan.
3. **Casos límite no cubiertos** — vacío, error, permisos, concurrencia, idempotencia, datos inválidos, degradación — solo si el dominio de la spec los hace relevantes.
4. **Conflictos con `docs/constitution.md`** — requisitos o planes que choquen con principios/reglas de la constitución. Si no hay archivo, no inventes conflictos.

### Fase 4 — Informe (solo detección)

Emite **exactamente** esta estructura en español. No agregues sección de soluciones ni “recomendaciones”.

```markdown
## Informe QA — [ruta-de-la-spec]

### 1. Ambigüedades restantes
1. ...

### 2. Contradicciones entre requisitos
1. ...

### 3. Casos límite no cubiertos
1. ...

### 4. Conflictos con docs/constitution.md
1. ...
```

Si una categoría no tiene hallazgos: una sola línea `Ninguno detectado.` (sin relleno inventado).

Citas breves (idea + ubicación en la spec/constitución) cuando ayuden; no pegues la spec entera.

**Detente aquí.** No edites la spec. No propongas `/spec-impl`. Puedes recordar que el humano decide si corregir la spec o pasar a `Aprobado`.

---

## Reglas duras

- **Solo detectar.** Prohibido proponer soluciones, refactors, texto alternativo para la spec, o “cómo arreglarlo”.
- **No escribir código** ni modificar la spec en esta skill.
- **No inventar stack** ni criticar por ausencia de framework.
- **No analizar** specs de ejemplo de formato ni `Obsoleto`.
- Informe **siempre en español**.

## Resumen del comportamiento esperado

```
/spec-qa 01-auth-session

  Fase 1  →  Encuentra specs/01-auth-session.md (o specs/01-auth-session/spec.md)
  Fase 2  →  Lee docs/constitution.md si existe
  Fase 3  →  Analiza ambigüedades, contradicciones, límites, conflictos
  Fase 4  →  Informe numerado en español — sin soluciones

/spec-qa 00-ejemplo-formato

  Fase 1  →  ❌ rechazo (ejemplo de formato)
```
