# sdd-skills

Kit **Spec-Driven Development (SDD)** para agentes de código: define el contrato antes de implementar.

[![skills.sh](https://skills.sh/b/PintoDaniel10/sdd-skills)](https://skills.sh/PintoDaniel10/sdd-skills)

## Skills

| Skill | Uso |
| ----- | --- |
| `spec` | Diseñar specs (preguntas, alcance, plan, criterios). **No escribe código de producto.** |
| `spec-qa` | Revisar una spec como QA (ambigüedades, contradicciones, casos límite, constitución). **Solo detecta.** |
| `spec-impl` | Implementar una spec en estado `Aprobado`, paso a paso, con pausas para revisar diffs. |
| `spec-verify` | Verificar y marcar criterios de aceptación tras implementar. Agnóstica de stack. |

Flujo:

1. `/spec` → guarda la spec en `Borrador`
2. `/spec-qa` → informe QA (opcional pero recomendado) antes de aprobar
3. Humano revisa y cambia el estado a `Aprobado`
4. `/spec-impl` → rama `spec-NN-slug` e implementación fiel al plan
5. `/spec-verify` → marca checks de criterios; humano confirma `Implementado`

Layouts de specs admitidos:

- Plano: `specs/01-auth-session.md`
- Anidado: `specs/01-auth-session/spec.md`

Agnóstico de stack: el runtime/framework se fija en un bootstrap o en `AGENTS.md`, nunca se inventa.

## Instalación (skills.sh)

```bash
npx skills add PintoDaniel10/sdd-skills
```

Listar skills del repo:

```bash
npx skills add PintoDaniel10/sdd-skills --list
```

Instalar solo una skill:

```bash
npx skills add PintoDaniel10/sdd-skills --skill spec
npx skills add PintoDaniel10/sdd-skills --skill spec-qa
npx skills add PintoDaniel10/sdd-skills --skill spec-impl
npx skills add PintoDaniel10/sdd-skills --skill spec-verify
```

Tras instalar, la página del directorio es: [skills.sh/PintoDaniel10/sdd-skills](https://skills.sh/PintoDaniel10/sdd-skills). El leaderboard se alimenta de telemetría anónima de la CLI al instalar.

## Estructura

```text
skills/
  spec/
    SKILL.md
    template.md
  spec-qa/
    SKILL.md
  spec-impl/
    SKILL.md
  spec-verify/
    SKILL.md
```

Compatible con la CLI de [skills.sh](https://skills.sh) / [vercel-labs/skills](https://github.com/vercel-labs/skills).

## Uso en Cursor sin la CLI

Copia `skills/spec`, `skills/spec-qa`, `skills/spec-impl` y `skills/spec-verify` a `.cursor/skills/` de tu proyecto (manteniendo `template.md` junto a `spec`).

## Licencia

MIT
