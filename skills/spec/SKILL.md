---
name: spec
description: Spec-Driven Development (SDD). Diseña specs agnósticas de stack — aclara requisitos, fija bootstrap/stack si hace falta, y escribe el contrato sección a sección antes de cualquier código. Úsalo al empezar una feature grande o un proyecto nuevo.
disable-model-invocation: true
argument-hint: 'descripción corta de la feature, bootstrap o requisito'
---

# /spec — Diseñador SDD (Spec-Driven Development)

## Contexto de sesión

Antes de la Fase 1, obtén fecha y listado de specs con el shell disponible. **No inventes la fecha.**

**Fecha de hoy** (campo `**Fecha:**` del encabezado):

- Unix / Git Bash: `date +%F`
- PowerShell (Windows): `Get-Date -Format yyyy-MM-dd`
- Si el shell falla: usa la fecha del contexto del sistema / prompt del usuario (`Today's date` o equivalente). Nunca inventes otra.

**Specs que ya existen:**

- Preferido: listar con Glob `specs/*.md` (o `specs/**/*.md`).
- Alternativa shell: Unix `ls specs/ 2>/dev/null`; PowerShell `Get-ChildItem specs -ErrorAction SilentlyContinue`.
- Si no hay carpeta o está vacía: anota que `specs/` aún no existe o no tiene specs.

Si tu entorno inyecta bloques `!`comando`` y funcionan, puedes usarlos; si fallan, aplica lo de arriba.

---

Esta skill aplica **Spec-Driven Development (SDD)** en el repo actual: el contrato vive en `specs/` y gobierna la implementación posterior (`/spec-impl`). **Aquí no escribes código de producto.** Aclaras qué construir, preguntas cuando falte definición, y guardas la spec lista en `specs/`.

El kit es **agnóstico de stack**. Web, API, CLI, móvil, monorepo u otro: el stack se elige en un **bootstrap** (o se lee de `AGENTS.md`), nunca se inventa. Las etiquetas canónicas de specs en este flujo están en **español** (ver `AGENTS.md` del proyecto).

## Filosofía

Una spec no es documentación decorativa. Es el contrato SDD que impulsa la ejecución. Si la spec es vaga, el código improvisa. Por eso este flujo es **deliberadamente lento en la fase de definición** y **rápido en la fase de escritura**.

Lee `template.md` (en el mismo directorio que esta skill) para ver la estructura completa que seguirá la spec. Apóyate en él en cada paso.

## Flujo del comando

- Sigue las cuatro fases en orden. **Nunca saltes la Fase 2** — las preguntas son el punto central. Si el usuario quiere ir más rápido, recuérdale que el coste de una mala spec se paga después en código. (La Fase 3 sí tiene un camino rápido una vez que la Fase 2 está genuinamente completa; ver más abajo.)
- Tus respuestas deben estar en el mismo idioma que el prompt inicial. Ej.: si el prompt inicial está en español, tus respuestas deben estar en español; si está en inglés, en inglés.

### Fase 1 — Entender el contexto

Antes de preguntar sobre la feature, asegúrate de tener contexto del proyecto:

1. Lee el archivo de memoria del proyecto, si existe. Prueba en este orden y detente en el primero que encuentres: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`.
2. Mira el listado de `specs/` (paso de contexto de sesión) para ver qué specs ya existen y cómo están numeradas.
3. Si ya hay specs previas, lee al menos las dos más recientes para captar las convenciones del proyecto — incluido el **idioma** en que están escritas y la redacción exacta que usan para estados y encabezados de sección. Una spec nueva debe coincidir con las existentes.
4. **Greenfield / sin stack (obligatorio si aplica).** Detecta si aún no hay base de producto: stack pendiente o vacío en `AGENTS.md` **y** no hay código de aplicación reconocible (p. ej. no hay `src/`, `app/`, `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, ni equivalentes). En ese caso **no** trates la petición como una feature normal:
   - Pregunta si lo que quieren es un **bootstrap** (`00-…`: elegir **cualquier** stack, estructura mínima, scripts de arranque/test, actualizar `AGENTS.md`) o una feature de producto.
   - Si es feature de producto y aún no hay bootstrap ni stack fijado, **detente** y propone: (a) actualizar `AGENTS.md` con stack mínimo **acordado con el usuario**, y/o (b) escribir primero la spec de bootstrap. **No inventes ni recomiendes un stack por defecto** (ni React, ni Node, ni otro) a menos que el usuario pida una recomendación explícita; entonces ofrece 2–4 opciones con razones y espera decisión.
   - Al numerar, **ignora** `specs/00-ejemplo-formato.md` (es referencia `Obsoleto`, no una spec de trabajo).
   - Solo continúa con la feature cuando el usuario confirme el camino (bootstrap primero, o stack ya fijado en memoria).

Si el argumento `$ARGUMENTS` llega vacío, pide al usuario una descripción inicial de **una sola frase** de lo que quiere construir. Si la descripción no cabe en una frase, esa es la primera señal de que la feature es demasiado grande — sugiere dividirla antes de continuar.

### Fase 2 — Aclarar con preguntas

Esta es la fase más importante del comando. Tu trabajo aquí es **detectar ambigüedades y preguntar**, no asumir.

Haz preguntas en bloques de 3 a 5 a la vez (no una pregunta suelta seguida de otra — eso agota). Tras cada bloque, espera la respuesta antes de continuar.

**Categorías de preguntas que siempre debes considerar:**

- **Alcance:** ¿Qué entra y qué NO? ¿Qué partes se aplazan a otra spec?
- **Stack / tooling:** Obligatoria en bootstrap o si la spec toca tooling. Runtime, framework(s), layout de carpetas, cómo arrancar, cómo testear, dónde queda documentado en `AGENTS.md`. En features normales: **lee** el stack de `AGENTS.md` y **no lo reabras** salvo que el usuario quiera cambiarlo.
- **Datos:** ¿Qué estructuras nuevas se introducen? ¿Cómo se nombran? ¿Dónde viven?
- **Integración:** ¿Depende de specs anteriores? ¿Modifica algo existente o solo añade?
- **Persistencia:** ¿Se guarda algo entre sesiones/despliegues? ¿Dónde? ¿Con qué versionado? (solo si aplica al dominio)
- **UX / API / estados:** Según el tipo de producto: pantallas, endpoints, códigos de salida CLI, estados de error e intermedios.
- **Riesgos:** ¿Qué puede romper esto? ¿Qué pasa en el caso degradado?
- **Decisiones cerradas:** ¿Hay alguna decisión que el usuario ya tomó y no quiere reabrir?

**Cómo formular las preguntas:**

- Usa preguntas concretas, no abiertas. ❌ "¿Cómo imaginas la persistencia?" → ✅ "¿Persistencia en archivo local, base SQL, o sin persistir (solo memoria)?"
- Cuando ofrezcas opciones, da 2–4, marca cuál es tu recomendación **solo si el usuario pidió consejo o el dominio ya acota las opciones**, y por qué. En elección de stack greenfield, no empujes un framework favorito.
- **Camino principal (Cursor y la mayoría de agentes):** bloques en markdown numerado con opciones claras; el usuario responde eligiendo. Si el agente expone una herramienta nativa de opción múltiple (p. ej. `AskUserQuestion`), úsala; si no existe o falla, **siempre** el markdown numerado. Pon tu recomendación primero y etiquétala cuando haya una.
- Si detectas una respuesta que abriría la caja de Pandora (p. ej. "y también queremos auth + sync multi-región"), señala que merece su propia spec y pregunta si lo dejamos fuera del alcance de esta.

**Cuándo dejar de preguntar:**

Para cuando puedas responder estas tres preguntas sin asumir nada:

1. ¿Qué archivos aparecerán o cambiarán?
2. ¿Cuál es el primer paso ejecutable y cuál el último?
3. ¿Cómo verifico que la feature está terminada?

Si aún no puedes responder alguna, sigue preguntando.

### Fase 3 — Escribir la spec

Una vez cerrada la Fase 2, decide cómo escribirla:

**Si ya tienes toda la información que necesitas** — es decir, puedes responder las tres preguntas de la Fase 2 **sin asumir nada** — entonces **no vayas sección por sección**. Escribe la spec completa y salta directo a la Fase 4 para guardar el archivo. No pidas confirmación sección por sección, y no muestres un borrador para aprobación primero: el usuario ya respondió todo en la Fase 2, y volver a preguntar es fricción. El usuario revisa el archivo guardado y pide cambios si hace falta.

**Solo si aún falta información** (el usuario cortó la Fase 2, una respuesta fue vaga, o alguna sección no se puede escribir sin inventar algo), desarrolla las secciones **una por una**, mostrando cada una y esperando confirmación antes de pasar a la siguiente.

En ambos casos el contenido sigue el mismo orden:

1. **Encabezado** (estado, dependencias, fecha, objetivo en una frase). El objetivo en una frase es crítico — si no cabe en una frase, vuelve a la Fase 2.
2. **Alcance** (qué entra y qué NO). El "no entra" debe ser explícito.
3. **Modelo de datos** (estructuras concretas con nombres reales). Si la feature no introduce datos nuevos, omite esta sección y dilo explícitamente.
4. **Plan de implementación** (pasos numerados, cada uno dejando el sistema funcional).
5. **Criterios de aceptación** (checklist booleana, no aspiracional). Incluye, si aporta, cómo verificar (comandos o pruebas) alineado a esos criterios — ver `template.md`.
6. **Decisiones tomadas y descartadas** (con justificación breve). En bootstrap: incluir la decisión de stack y que queda escrita en `AGENTS.md`.
7. **Riesgos identificados** (solo si aplica — si no hay riesgos relevantes, omítelo).

**Tras cada sección (solo en el modo sección por sección):**

- Muéstrala formateada en markdown.
- Pregunta: "¿Esta sección se queda así o quieres ajustarla?"
- Si el usuario pide cambios, aplícalos y vuelve a mostrar.
- Solo pasa a la siguiente sección cuando el usuario confirme.

**Errores comunes a evitar:**

- Generar criterios de aceptación no verificables ("que funcione bien").
- Meter en el plan de implementación cosas que no están en el alcance.
- Asumir nombres de archivos, estructuras o **stack** que el usuario no confirmó.
- Saltar la sección de decisiones — esa sección es la de mayor valor a largo plazo.

### Fase 4 — Guardar la spec

Cuando el contenido esté listo (porque ya tenías todo, o porque todas las secciones fueron confirmadas):

1. Determina el siguiente número secuencial a partir del listado de `specs/` (excluye `00-ejemplo-formato.md`). Toma el número más alto de specs de trabajo y súmale uno, con cero a la izquierda a dos dígitos. Si la última de trabajo es `01-auth-session.md`, esta será `02-`. Si no hay specs de trabajo, empieza en `01-` (o `00-` si es explícitamente bootstrap y el usuario lo acordó; no reutilices el nombre del ejemplo de formato).
2. Genera un slug corto en kebab-case a partir del objetivo (p. ej. `auth-session`, `bootstrap-stack`). Ver **Argumentos** abajo.
3. Usa la fecha obtenida en el contexto de sesión para el campo `**Fecha:**`. **Nunca escribas una fecha que no hayas obtenido de ahí.**
4. Escribe el archivo directamente en `specs/NN-slug.md` con todas las secciones. **No pidas permiso para escribirlo y no preguntes si el nombre del archivo vale** — anuncia la ruta en la confirmación final. Solo pregunta si el archivo destino ya existe.
5. Marca el estado como `Borrador` por defecto (o la palabra equivalente que usen las specs existentes en este repo). **No lo marques como `Aprobado` automáticamente** — eso lo hace el usuario cuando lo haya releído.
6. Si el encabezado lista dependencias (`**Depende de:** SPEC 01`), comprueba que cada spec referenciada exista realmente en `specs/`. Si alguna no existe, dilo en lugar de escribir una referencia colgante.
7. **Crea el archivo de configuración si no existe.** Busca `specs/.spec-config.yml`. Si **falta**, créalo con el contenido por defecto de abajo. Si **ya existe, déjalo intacto** — nunca sobrescribas la configuración del usuario.

   ```yaml
   # configuración del flujo SDD
   #
   # AutoCreateBranch — controla si /spec-impl crea la rama git automáticamente.
   #   true  (por defecto) → /spec-impl crea y cambia a spec-NN-slug sin preguntar
   #   false               → /spec-impl pide confirmación [y/N] antes de crear la rama
   AutoCreateBranch: true
   ```

8. Si la spec es de **bootstrap** y el usuario acordó el stack, en la confirmación final recuerda que, al implementar (o ya si el usuario lo pidió en esta sesión sin código de producto), la sección Stack de `AGENTS.md` debe quedar actualizada. **No escribas código de producto aquí.**
9. Confirma al usuario:
   - Ruta del archivo creado.
   - Recordatorio: la spec está en estado `Borrador`. Cámbiala a `Aprobado` cuando la hayas releído.
   - Si acabas de crear `specs/.spec-config.yml`, menciona que existe y que `AutoCreateBranch` por defecto es `true`.
   - Siguiente paso: una vez revisada y aprobada, ejecuta `/spec-impl NN-slug` para implementarla.
   - **Detente aquí.** No propongas implementar la spec, escribir código ni ninguna acción posterior más allá de esta confirmación.

## Reglas duras

- **Nunca escribas código de producto durante este comando.** Solo el archivo `.md` de la spec al final (y `specs/.spec-config.yml` / actualización de la sección Stack en `AGENTS.md` si el usuario lo acordó como parte del greenfield/bootstrap).
- **Nunca inventes el stack.** Se acuerda con el usuario y se documenta.
- **Nunca propongas implementar la spec después de guardarla.** Tu trabajo termina cuando el archivo está escrito. El usuario ejecuta `/spec-impl` cuando esté listo.
- **Nunca asumas decisiones que el usuario no confirmó.** Si te falta información, pregunta — en la Fase 2.
- **No vuelvas a preguntar en la Fase 3 lo que ya se respondió en la Fase 2.** Si la información está completa, escribe la spec entera y guárdala.
- **Si el usuario quiere acelerar y saltarse la Fase 2**, recuérdale: "Preguntar ahora ahorra horas después. ¿Seguro que quieres saltártelas?". Si insiste, respeta su decisión pero regístrala en la sección de decisiones ("Definición rápida sin aclaración detallada").
- **Si la feature es demasiado grande** (no cabe en una frase, toca más de tres áreas, requiere decisiones en cuatro o más dominios), propone dividirla en dos o más specs antes de continuar.

## Tono al hacer preguntas

Sé directo y concreto. No te disculpes por preguntar. El usuario invocó esta skill precisamente porque quiere que preguntes. Usa preguntas concretas, una por línea cuando hay varias, y numéralas.

Ejemplo de un bloque bien formado (bootstrap / stack — genérico):

> Antes de cerrar el bootstrap necesito fijar el stack (sin asumir ninguno):
>
> 1. **Tipo de producto.** ¿App web, API/backend, CLI, móvil, librería, monorepo u otro?
> 2. **Runtime y tooling.** ¿Qué lenguaje/runtime y gestor de dependencias quieres? (elige tú; si pides recomendación, doy 2–4 opciones).
> 3. **Layout mínimo.** ¿Carpetas raíz y punto de entrada? ¿Dónde documentamos arranque y test en `AGENTS.md`?

Ejemplo de un bloque bien formado (feature — dominio neutro):

> Antes de escribir el modelo de datos necesito aclarar tres cosas:
>
> 1. **Límite de la feature.** ¿Solo lectura, o también crear/actualizar/borrar?
> 2. **Contrato externo.** ¿UI, endpoints HTTP, comandos CLI, o eventos? Lista los nombres concretos.
> 3. **Persistencia.** ¿Reutiliza lo del stack en `AGENTS.md` o introduce un almacén nuevo?

## Argumentos

`$ARGUMENTS` es **la descripción**, no el nombre del archivo. Trátalo como el punto de partida de la Fase 1 y deriva el slug del objetivo en la Fase 4.

La única excepción: si `$ARGUMENTS` ya es un único token en kebab-case sin espacios (p. ej. `/spec auth-session` o `/spec bootstrap-stack`), úsalo como slug **y** como semilla de la descripción, sin pedir confirmación.

Si invocaron `/spec` sin argumentos, empieza pidiendo la descripción en una frase.
