# sdd-skills

Kit **Spec-Driven Development (SDD)** para agentes de código: define el contrato antes de implementar.

[![skills.sh](https://skills.sh/b/PintoDaniel10/sdd-skills)](https://skills.sh/PintoDaniel10/sdd-skills)

## Skills

| Skill | Uso |
| ----- | --- |
| `spec` | Diseñar specs (preguntas, alcance, plan, criterios). **No escribe código de producto.** |
| `spec-impl` | Implementar una spec en estado `Aprobado`, paso a paso, con pausas para revisar diffs. |

Flujo:

1. `/spec` → guarda `specs/NN-slug.md` en `Borrador`
2. Humano revisa y cambia el estado a `Aprobado`
3. `/spec-impl` → rama `spec-NN-slug` e implementación fiel al plan

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
npx skills add PintoDaniel10/sdd-skills --skill spec-impl
```

Tras instalar, la página del directorio es: [skills.sh/PintoDaniel10/sdd-skills](https://skills.sh/PintoDaniel10/sdd-skills). El leaderboard se alimenta de telemetría anónima de la CLI al instalar.

## Estructura

```text
skills/
  spec/
    SKILL.md
    template.md
  spec-impl/
    SKILL.md
```

Compatible con la CLI de [skills.sh](https://skills.sh) / [vercel-labs/skills](https://github.com/vercel-labs/skills).

## Uso en Cursor sin la CLI

Copia `skills/spec` y `skills/spec-impl` a `.cursor/skills/` de tu proyecto (manteniendo `template.md` junto a `spec`).

## Licencia

MIT
