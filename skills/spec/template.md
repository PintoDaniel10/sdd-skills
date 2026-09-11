# Plantilla de una spec útil — SDD

Referencia que consulta la skill `/spec` al generar specs con **Spec-Driven Development (SDD)**. Cada sección incluye propósito y un ejemplo mínimo. **No es texto para copiar literalmente** — es la forma que la skill debe respetar.

El kit es **agnóstico de stack**: web, API, CLI, móvil, monorepo, etc. El stack se fija en un bootstrap y se documenta en `AGENTS.md`.

En este flujo las etiquetas canónicas están en **español** (`Estado`, `Alcance`, `Objetivo`, etc.).

---

## Feature vs bootstrap

Antes de numerar y escribir, distingue el tipo de spec:

| Tipo | Cuándo | Numeración típica | Qué cubre |
| ---- | ------ | ----------------- | --------- |
| **Bootstrap** | Repo sin stack/estructura mínima de producto | `00-…` (acordado con el usuario) | Elegir **cualquier** stack (runtime, framework, layout), scripts de arranque/test, actualizar `AGENTS.md`, dejar listo `specs/` |
| **Feature** | Ya hay base de producto (código o stack fijado en `AGENTS.md`) | `01-`, `02-`, … | Un cambio acotado sobre el sistema existente |

No mezcles bootstrap y la primera feature de producto en la misma spec. Si el usuario pide una feature y el repo sigue greenfield, `/spec` debe proponer bootstrap (o fijar stack en `AGENTS.md`) primero — ver Fase 1 de la skill.

Ignora `specs/00-ejemplo-formato.md` al numerar specs reales: es solo referencia de formato (`Obsoleto`).

---

## Encabezado

Toda spec empieza con metadatos en un blockquote (sin tablas, sin bloques, simple como se muestra abajo):

```markdown
# SPEC NN — Título corto y descriptivo

> **Estado:** Borrador
> **Depende de:** SPEC 00, SPEC 01
> **Fecha:** YYYY-MM-DD
> **Objetivo:** Una sola frase. Si necesitas dos frases, la feature es demasiado grande.
```

**Estados válidos (canónicos en este flujo):** `Borrador`, `En revisión`, `Aprobado`, `Implementado`, `Obsoleto`.

> Las skills de implementación también reconocen equivalentes en otros idiomas por compatibilidad. En specs nuevas usa siempre el conjunto en español acordado en `AGENTS.md`.

**Regla del objetivo:** una frase que un humano lee en 5 segundos y entiende qué se va a construir. Si no cabe en una frase, divide la feature.

---

## Sección 1 — Por qué existe esta spec (opcional)

Para specs que toman decisiones no obvias o rompen patrones del proyecto, una sección breve que explique el **porqué** del trabajo. No el qué — el qué viene después.

En specs simples, omítela.

---

## Sección 2 — Alcance

Dos subbloques explícitos. **Ambos son obligatorios.**

```markdown
## Alcance

**Entra:**

- Comportamiento concreto uno (ruta, endpoint, comando o pantalla).
- Comportamiento concreto dos.

**Fuera de alcance (para specs futuras):**

- Capacidad relacionada que se aplaza.
- Integración que surgió en la conversación pero no entra ahora.
```

**Por qué importa el "fuera":** captura lo que el usuario mencionó en la fase de preguntas pero se decidió aplazar. Sin ese registro, durante la implementación habrá tentación de colarlo "ya que estamos".

---

## Sección 3 — Modelo de datos

Las estructuras concretas que aparecen o cambian. Usa forma real del dominio (tipos, campos, nombres), no pseudocódigo vago. Adapta la sintaxis al stack elegido (JS, Python, Go, SQL, etc.).

```markdown
## Modelo de datos

\`\`\`text
Entidad:
  id: string
  nombre: string
  createdAt: datetime (ISO)
\`\`\`

Convenciones:

- IDs: estrategia acordada en la spec (uuid, slug, serial, …).
- Tiempos: zona y formato acordados.
```

**Ilustrativo — API:** cuerpo JSON de request/response con campos reales.  
**Ilustrativo — UI:** estado de pantalla (`idle | loading | empty | error`) sin asumir framework.

Si la feature no introduce datos nuevos, escríbelo explícitamente: _"Esta feature no introduce nuevas estructuras de datos. Reutiliza el modelo de la SPEC 01."_

---

## Sección 4 — Plan de implementación

Pasos numerados. Cada paso debe dejar el sistema en un estado **funcional y ejecutable** según el stack del proyecto. Nada de "implementar a medias y seguir mañana".

```markdown
## Plan de implementación

1. Crear el módulo / paquete X con un esqueleto vacío exportable.
2. Implementar la operación A. Prueba: comando o URL documentado en AGENTS.md / aquí.
3. Conectar X al punto de entrada existente W.
4. ...
```

**Reglas:**

- Cada paso debe poder commitearse por sí solo.
- Si un paso requiere más de 30–50 líneas de código, divídelo.
- El último paso del plan **no** es "probarlo todo" — eso son los criterios de aceptación.
- Usa rutas y comandos del stack fijado; no inventes otro framework a mitad de plan.

---

## Sección 5 — Criterios de aceptación

Checklist booleana. Cada ítem se puede verificar con sí o no.

```markdown
## Criterios de aceptación

- [ ] El sistema arranca / responde con el comando de smoke documentado.
- [ ] El caso vacío o de error acordado se comporta como se describe (no cuelga en silencio).
- [ ] Tras la operación principal, el resultado observable coincide con la spec.
```

**Anti-patrones a evitar:**

- ❌ "Que funcione bien." → no verificable.
- ❌ "Buena UX." → subjetivo.
- ❌ "Sin bugs." → no operativo.
- ✅ "Si falla la dependencia externa, se muestra/registra error recuperable y no se corrompe el estado." → verificable, booleano.

### Cómo verificar (opcional, breve)

Si ayuda a quien implementa, añade bajo los criterios **cómo** comprobarlos: comando, URL, o prueba manual concreta. No sustituye la checklist; la apoya. Los comandos deben coincidir con `AGENTS.md` cuando el stack ya esté fijado.

```markdown
## Cómo verificar

- Ejecutar el comando de arranque / test documentado en AGENTS.md.
- Comprobar el caso feliz y el caso de error listados arriba.
```

En specs triviales, omite esta subsección si los propios criterios ya dicen el procedimiento.

---

## Sección 6 — Decisiones tomadas y descartadas

La sección con más valor dentro de 3 meses. Captura **qué consideraste**, no solo qué elegiste.

```markdown
## Decisiones

- **Sí:** documentar el stack elegido en AGENTS.md durante el bootstrap. Fuente de verdad para /spec-impl.
- **No:** dejar el runtime implícito en el código sin memoria de proyecto.
- **Sí:** (ejemplo de dominio) versionar el esquema de datos / contrato público.
- **No:** (ejemplo) ampliar el alcance a auth en esta misma spec.
```

Cada decisión idealmente tiene una razón breve. Las decisiones sin razón son las primeras en cuestionarse después.

---

## Sección 7 — Riesgos identificados (opcional)

Solo cuando hay riesgos no obvios. Tabla simple:

```markdown
## Riesgos

| Riesgo                         | Mitigación                                              |
| ------------------------------ | ------------------------------------------------------- |
| Stack aún ambiguo a mitad de feature | Parar; no improvisar; volver a /spec o actualizar AGENTS.md |
| Contrato externo rompe clientes     | Versionar API / flag / migración documentada            |
```

En specs pequeñas o features muy contenidas, omítela.

---

## Sección final — Qué NO entra (refuerzo)

Repite explícitamente al final lo que **no** se hará en esta spec. Esta repetición es deliberada.

```markdown
## Qué **no** entra en esta spec

- Capacidad A (otra spec si algún día llega).
- Capacidad B.
- Capacidad C.

Cada una de esas, si llega, va en su propia spec.
```

---

## Reglas globales del documento completo

- **Una frase por idea.** Si una frase tiene dos comas y un punto y coma, divídela.
- **Nombres concretos.** Si dices "el módulo X", da la ruta real cuando exista. Si dices "una clave o ruta", da el string exacto.
- **Sin TODOs.** Un TODO en una spec significa que la decisión no se tomó. Tómala o anótala como decisión pendiente con una razón.
- **Sin código ejecutable largo.** La spec describe; el código se escribe después. Snippets cortos para ilustrar estructuras están bien; funciones completas no.
- **Sin stack inventado.** Si aún no hay stack, esta spec es bootstrap o la feature espera a que lo haya.
- **Markdown estándar.** Sin extensiones raras. Debe renderizar en GitHub sin sorpresas.
