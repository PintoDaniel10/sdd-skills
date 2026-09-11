---
name: spec-impl
description: Spec-Driven Development (SDD). Implementa una spec aprobada sin improvisar stack — valida estado Aprobado, crea rama, y ejecuta el plan paso a paso con pausas para revisar diffs.
disable-model-invocation: true
argument-hint: <NN-nombre-spec>
---

# /spec-impl — Implementador SDD

## Contexto de sesión

Antes de la Fase 1, obtén el estado del repo y de las specs. Usa el shell disponible (Bash o PowerShell). Si un comando falla, usa Glob/Read como alternativa.

**Estado del repositorio:** `git status --short`

**Rama actual:** `git branch --show-current`

**Specs disponibles:**

- Preferido: Glob `specs/*.md`
- Alternativa: Unix `ls specs/`; PowerShell `Get-ChildItem specs -ErrorAction SilentlyContinue`
- Si no hay carpeta: anota que `specs/` no existe

**Configuración de creación de ramas:**

- Lee `specs/.spec-config.yml` con Read (o `Get-Content` / `cat`)
- Si no existe: trata como `AutoCreateBranch: true` (por defecto)

Si tu entorno inyecta bloques `!`comando`` y funcionan, puedes usarlos; si fallan, aplica lo de arriba.

---

## Instrucciones

Esta skill es la mitad de ejecución del **Spec-Driven Development (SDD)**: implementas el contrato aprobado **tal cual**, usando el stack documentado en `AGENTS.md` y/o en la propia spec. **No asumas framework, runtime ni layout** si no están fijados.

Sigue estas cuatro fases en orden estricto. **No avances a la siguiente fase si la anterior no se completó correctamente.**

---

### Fase 1 — Identificar la spec

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` está vacío:

- Lista los archivos disponibles en `specs/` (ya los tienes arriba).
- Pide al usuario que indique el nombre exacto de la spec.
- Detente y espera una respuesta. No continúes.

Si `$ARGUMENTS` tiene valor:

- Busca el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`01-auth-session`), solo el número (`01`), o solo el slug (`auth-session`). Intenta encontrar el archivo correcto en cualquiera de esos casos.
- No implementes `00-ejemplo-formato` (está `Obsoleto` y es solo referencia de formato).
- Si no encuentras el archivo, muestra las specs disponibles y pide al usuario que corrija el nombre.
- Si lo encuentras, continúa a la Fase 2.

---

### Fase 2 — Validar el estado de la spec

Lee el archivo de la spec localizado en la Fase 1 usando la herramienta Read (o el shell si hace falta).

En el contenido del archivo, busca la línea que contiene el estado de la spec. En este repo la etiqueta canónica es `**Estado:**` (ver `AGENTS.md`). Por compatibilidad también puedes encontrar `**Status:**` u otras. Empareja por posición (línea de estado cerca del inicio) y por la máquina de estados, no solo por la etiqueta exacta.

**Regla absoluta:** Solo puedes continuar si el estado **significa "Aprobado"** — independientemente del idioma usado. El valor canónico en este flujo es `Aprobado`.

Trata cualquiera de los siguientes (y sus equivalentes en otros idiomas) como el estado **Aprobado** y continúa:

- Español (canónico aquí): `Aprobado`
- Inglés: `Approved`
- Portugués: `Aprovado`
- Francés: `Approuvé`
- Alemán: `Genehmigt`
- Italiano: `Approvato`
- …o la palabra de cualquier otro idioma que signifique claramente "aprobado"

Cualquier otra cosa (`Borrador`, `En revisión`, `Implementado`, `Obsoleto`, o equivalentes / valor no reconocido) significa **detenerse** y mostrar el mensaje de error de abajo.

| Categoría de estado                              | Ejemplos (cualquier idioma)                       | Acción                                                                     |
| ------------------------------------------------ | ------------------------------------------------- | -------------------------------------------------------------------------- |
| Aprobado                                         | `Approved`, `Aprobado`, `Aprovado`, `Approuvé`, … | Continuar a la Fase 3.                                                     |
| Borrador                                         | `Draft`, `Borrador`, …                            | Detenerse. Mostrar el mensaje de error de abajo.                           |
| En revisión                                      | `In review`, `En revisión`, …                     | Detenerse. Mostrar el mensaje de error de abajo.                           |
| Implementado                                     | `Implemented`, `Implementado`, …                  | Detenerse. Mostrar el mensaje de error de abajo.                           |
| Obsoleto                                         | `Obsolete`, `Obsoleto`, …                         | Detenerse. Mostrar el mensaje de error de abajo.                           |
| Línea de estado no encontrada / valor no reconocido | —                                              | Detenerse. El archivo no sigue el formato esperado. Díselo al usuario.     |

Si no estás seguro de si un valor significa "aprobado", **no asumas**. Detente y pide al usuario que aclare o que actualice la spec a la redacción canónica.

**Mensaje de error estándar cuando el estado no significa Aprobado:**

```
❌ No puedo implementar esta spec.

Estado actual: [ESTADO ENCONTRADO]
Solo trabajo con specs cuyo estado signifique "Aprobado" (canónico en este repo: `Aprobado`;
también se aceptan equivalentes como `Approved`).

Para continuar tienes dos opciones:
  1. Si la spec está lista para implementarse, ábrela y cambia el estado
     a "Aprobado" manualmente.
     Ese cambio lo hace el humano, no el agente.
  2. Si la spec aún necesita trabajo, usa /spec [nombre] para retomarla.
```

No ofrezcas alternativas, no sugieras "puedo empezar igual si quieres". El bloqueo es intencional.

---

### Fase 3 — Crear la rama git y cambiar a ella

Una vez confirmado que el estado significa `Aprobado`:

0. **Revisa primero el working tree.** Mira la salida de `git status --short`. Si **no está vacía**, detente, muestra los cambios pendientes y pregunta:

   ```
   ⚠️ Hay cambios sin commitear en el working tree.
   Cambiar de rama los arrastraría. ¿Qué quieres hacer?
     1. Commitéalos o haz stash tú mismo, y vuelve a ejecutar este comando  (recomendado)
     2. Continuar de todos modos — los cambios viajan a la nueva rama
   ```

   Espera la respuesta. **No hagas stash ni commit en nombre del usuario** salvo que lo pida explícitamente. Si el working tree está limpio, salta directo al paso 1 sin mencionarlo.

1. Deriva el nombre de la rama del nombre completo del archivo de la spec, sin la extensión. Formato: `spec-NN-slug`. Ejemplos:

   - `00-bootstrap-stack.md` → rama `spec-00-bootstrap-stack`
   - `01-auth-session.md` → rama `spec-01-auth-session`

2. Lee el flag `AutoCreateBranch` de la configuración (`specs/.spec-config.yml` o el valor por defecto del contexto de sesión).

   - Si el archivo de configuración no existe, el valor falta, o el valor no se reconoce → trátalo como `true` (el valor por defecto).
   - Solo un `false` explícito (en cualquier capitalización) desactiva la creación automática de ramas.

   **Si `AutoCreateBranch` es `true` (por defecto):** procede sin preguntar.

   - Si la rama **no existe**: créala con `git checkout -b spec-NN-slug`.
   - Si **ya existe**: significa que se está retomando trabajo previo. Cambia a ella, lee `git log --oneline` en la rama, y dile al usuario qué pasos del plan parecen ya hechos y desde qué paso propones retomar. Espera confirmación del punto de reanudación antes de implementar nada.
   - En ambos casos: cambia a la rama con `git checkout spec-NN-slug` y confirma que el cambio fue exitoso antes de continuar.

   **Si `AutoCreateBranch` es `false`:** pregunta antes de tocar git. Muestra:

   ```
   AutoCreateBranch está en false.
   ¿Crear y cambiar a la rama spec-NN-slug? [y/N]
   ```

   - Si el usuario responde **sí**: crea/cambia a la rama exactamente como en el caso `true` de arriba.
   - Si el usuario responde **no** o lo deja vacío: **no crees ninguna rama.** Dile al usuario que implementarás en la rama actual y pide confirmación explícita para continuar ahí. No improvises — espera la respuesta.

3. Confirma visualmente al usuario que la spec está lista y qué rama está activa:

   ```
   ✅ Listo para implementar (SDD).

   Spec:   specs/NN-slug.md
   Rama:   spec-NN-slug  (activa)   (← o la rama actual, si no se creó una nueva)
   Estado: Aprobado   (← repite el valor real encontrado en la spec)
   ```

4. **Aún no empieces a implementar.** Primero muestra el resumen de la spec al usuario. Las secciones canónicas están en español (ver `AGENTS.md`); extrae y muestra:
   - El **objetivo** (línea tras `**Objetivo:**`; compat: `**Objective:**`).
   - El **alcance** (`## Alcance`; compat: `## Scope`).
   - El **plan de implementación** (`## Plan de implementación`; compat: `## Implementation plan`).
   - Los **criterios de aceptación** (`## Criterios de aceptación`; compat: `## Acceptance criteria`).

Empareja los encabezados de sección por significado, no solo por redacción exacta.

5. **Stack antes de codificar.** Lee `AGENTS.md` (sección Stack) y lo que diga la spec sobre tooling. Si la implementación requiere un stack o comandos que **no** están definidos ni en la spec ni en `AGENTS.md`, **detente**, descríbelo y pregunta. **No improvises framework ni runtime.**

---

### Fase 4 — Implementar paso a paso

Tras mostrar el resumen de la spec, dile al usuario:

```
Voy a implementar la spec siguiendo el plan de implementación exactamente (SDD).
Usaré el stack documentado en AGENTS.md / en la spec.
Haré una pausa tras cada paso para que puedas revisar el diff.

¿Empezamos con el Paso 1?
```

Espera confirmación explícita ("sí", "adelante", "vamos", o equivalente). No empieces sin ella.

Una vez confirmado, sigue estas reglas durante toda la implementación:

**Nunca hagas commit automáticamente.** Ni por paso, ni al final. Tú escribes el código y muestras el diff; el commit es decisión y comando del usuario. Solo haz commit si te lo pide explícitamente.

**Una regla por encima de todo:** implementa lo que la spec dice. Si algo te parece subóptimo, menciónalo como observación pero implementa lo acordado. Los cambios al contrato van a la spec, no al código por sorpresa.

**Stack:** sigue `AGENTS.md` y la spec. Si durante un paso descubres que falta una decisión de tooling, para y pregunta — no cambies de framework por conveniencia.

**Ritmo de trabajo:**

- Implementa un paso del plan.
- Muestra un resumen de qué archivos tocaste y qué hiciste.
- Di: `Paso N completado. ¿Puedes revisar el diff y decirme si continúo con el Paso N+1?`
- Espera confirmación antes de continuar.

**Si durante la implementación encuentras una ambigüedad** que la spec no resuelve:

- Detente.
- Describe la ambigüedad exactamente.
- Presenta dos o tres opciones concretas.
- Espera la decisión del usuario.
- No improvises.

**Si el usuario pide algo que está fuera del alcance de la spec:**

- Recuérdale que está fuera del alcance de esta spec.
- Sugiere anotarlo para la siguiente spec.
- No lo implementes en esta rama.

**Al terminar el último paso:**

```
✅ Todos los pasos del plan están implementados.

Siguiente paso: verifica los criterios de aceptación de la spec uno a uno.
Si todos pasan, actualiza el estado de la spec a "Implementado"
y haz el commit final antes de fusionar esta rama.
```

---

## Resumen del comportamiento esperado

```
/spec-impl 01-auth-session

  Fase 1  →  Encuentra specs/01-auth-session.md
  Fase 2  →  Lee el estado → "Aprobado" → ✅ continúa
  Fase 3  →  git checkout -b spec-01-auth-session → …
              Muestra objetivo, alcance, plan y criterios
              Verifica stack en AGENTS.md / spec
  Fase 4  →  Implementa paso a paso con pausas
              Termina recordando verificar los criterios de aceptación

/spec-impl 00-bootstrap-stack  (estado: Borrador)

  Fase 1  →  Encuentra specs/00-bootstrap-stack.md
  Fase 2  →  Lee el estado → "Borrador" → ❌ se detiene
              Muestra el mensaje de error estándar
              No crea rama, no toca código
```

**La creación de ramas se controla con el flag `AutoCreateBranch`** en `specs/.spec-config.yml`. Por defecto es `true`. Ponlo en `false` para que la Fase 3 pregunte `[y/N]` antes de crear la rama.
