# Procedencia de las skills de MyFactory

Skills copiadas fichero a fichero (no symlinks), clavadas al commit indicado y con
la licencia del origen dentro de cada carpeta como `LICENSE.upstream`.

MyFactory es el almacén: de aquí se copian a `.claude/skills/` del proyecto que
las necesite, anotando la fila correspondiente en el `SOURCES.md` de ese proyecto.

## Instaladas

| Skill | Repositorio | Commit | Licencia | Fecha | Para qué la queremos |
| --- | --- | --- | --- | --- | --- |
| `coherencia-docs` | Propia de MyFactory | — | Sin licencia declarada | 2026-09-22 | Coherencia entre los documentos de contexto de un proyecto |
| `brainstorming` | `obra/superpowers` | `5bf4e78011075bcfc0dc295f0724994cd123ee71` | MIT | 2026-09-23 | Saca una spec de la conversación antes de escribir código. Trae un `<HARD-GATE>`: sin aprobación humana de la spec escrita no se pasa al plan, y sin aprobación del plan no se toca el código |
| `writing-plans` | `obra/superpowers` | `5bf4e78011075bcfc0dc295f0724994cd123ee71` | MIT | 2026-09-23 | Convierte una spec aprobada en pasos con su test, dimensionados para un commit verificable |
| `test-driven-development` | `obra/superpowers` | `5bf4e78011075bcfc0dc295f0724994cd123ee71` | MIT | 2026-09-23 | Rojo → verde → refactor, con el test visto fallar antes de implementar |
| `verification-before-completion` | `obra/superpowers` | `5bf4e78011075bcfc0dc295f0724994cd123ee71` | MIT | 2026-09-23 | Prohíbe declarar algo terminado sin ejecutar la verificación y leer su salida |
| `writing-skills` | `obra/superpowers` | `5bf4e78011075bcfc0dc295f0724994cd123ee71` | MIT | 2026-09-23 | Para fabricar skills nuevas en este mismo repositorio |

## Referencias que quedan sin resolver

Las skills de `superpowers` se citan entre sí con la sintaxis `superpowers:<skill>`.
Estas citas apuntan a skills **no** instaladas aquí:

- `writing-plans` → `superpowers:executing-plans`, `superpowers:subagent-driven-development`, `superpowers:using-git-worktrees`
- `writing-skills` → `superpowers:systematic-debugging`, `superpowers:test-driven-development`

Solo la última resuelve. Las demás son menciones en prosa: la skill funciona igual,
pero si alguna vez se sigue esa pista no habrá nada al otro lado. Se instalan si
hacen falta, no por adelantado.

## Descartadas tras evaluarlas

Ver [`../../catalogo-sdd.md`](../../catalogo-sdd.md) para el porqué de cada una:
GitHub Spec Kit, OpenSpec, GSD, BMAD-METHOD, MUSUBI y Agent OS.
