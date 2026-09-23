# MyFactory

Almacén de skills para Claude Code. De aquí se copian a `.claude/skills/` del
proyecto que las necesite; no se instalan globalmente ni como marketplace.

## Qué hay

| Skill | Origen | Para qué |
| --- | --- | --- |
| `coherencia-docs` | propia | Contrasta los documentos de contexto de un proyecto entre sí y resuelve sus incoherencias |
| `brainstorming` | superpowers | Saca una spec de la conversación y no deja implementar sin aprobación humana |
| `writing-plans` | superpowers | Convierte una spec aprobada en pasos con su test |
| `test-driven-development` | superpowers | Rojo → verde → refactor |
| `verification-before-completion` | superpowers | Evidencia antes de declarar algo terminado |
| `writing-skills` | superpowers | Para fabricar skills nuevas aquí |

Procedencia, commit y licencia de cada una: [`.claude/Skills/SOURCES.md`](.claude/Skills/SOURCES.md).

Por qué estas y no otras — evaluación de Spec Kit, OpenSpec, GSD, BMAD, MUSUBI,
Agent OS y seis productos comerciales: [`catalogo-sdd.md`](catalogo-sdd.md).

## Instalar una skill en un proyecto

```bash
cp -r MyFactory/.claude/Skills/<skill> <proyecto>/.claude/skills/<skill>
```

Y se añade su fila al `SOURCES.md` del proyecto: repositorio, commit, licencia y
para qué se quiere. Una skill sin procedencia anotada no entra.

## Nota sobre el nombre de la carpeta

Aquí las skills viven en `.claude/Skills/` (con mayúscula), que es como se creó el
repositorio. Windows no distingue mayúsculas, pero Linux y macOS sí, y Claude Code
busca `.claude/skills/`. En el destino se copia siempre en minúscula.
