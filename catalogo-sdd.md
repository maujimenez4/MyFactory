# Catálogo de sistemas de desarrollo dirigido por specs

Evaluación hecha el **2026-09-23** clonando cada repositorio y leyendo sus artefactos.
La pregunta que se responde aquí es una sola: **¿esto nos ayuda a escribir mejores
specs en `ciberpunk-storymaker`?** Ese proyecto ya tiene su proceso en `CLAUDE.md` §3
(spec → plan → código → cierre, cuatro puertas, aprobación humana en commit propio) y
su vocabulario cerrado en `docs/definitions.md` §2. Lo que se busca es **técnica**,
no un proceso de repuesto.

Criterio de instalación, en este orden:

1. **Auto-contenida.** Si necesita su CLI, sus scripts o `~/.claude/<framework>/`, no es una skill: es un framework.
2. **No trae proceso propio.** El nuestro ya existe y tiene puertas.
3. **No introduce vocabulario.** §2 prohíbe términos que no estén en `docs/definitions.md`.

## Instaladas

| Skill | Origen | Por qué entra |
| --- | --- | --- |
| `brainstorming` | superpowers | Es la puerta **Spec** de §3 escrita como skill: clasifica cuánto proceso pide el trabajo, escribe lo entendido para que el humano lo corrija, y su `<HARD-GATE>` impide implementar sin aprobación de la spec y del plan **por separado** — justo lo que §3 exige y lo que §14 prohíbe saltarse |
| `writing-plans` | superpowers | Es §3.3: pasos del tamaño de un commit verificable, cada uno con su test, decisiones de ficheros antes de las tareas |
| `test-driven-development` | superpowers | Es §3.4: rojo → verde → refactor, con el test visto fallar |
| `verification-before-completion` | superpowers | Es el checklist de §15: evidencia antes de afirmar que algo pasa |
| `writing-skills` | superpowers | Para fabricar aquí las skills que falten |

Las cinco son markdown puro: **cero referencias a ficheros fuera de su carpeta**
(comprobado). Se copian y funcionan.

## Descartadas, y qué le robamos a cada una

### GitHub Spec Kit — `github/spec-kit` · MIT · `8850a18`

El más completo en plantillas: `spec-template.md` (historias priorizadas P1/P2/P3, cada
una *testeable por separado*, `Given/When/Then`, requisitos `FR-###`), `plan-template`,
`tasks-template`, `checklist-template` y `constitution-template`.

**No se instala** porque sus comandos no son autónomos: llevan literales
`__SPECKIT_COMMAND_*__` que solo resuelve `specify init` al escribirlos, y llaman a
`.specify/scripts/{bash,powershell,python}/…`. Copiarlos a `.claude/skills/` deja
prompts rotos. Instalarlo de verdad crearía un segundo proceso, en inglés, en paralelo
al de §3.

**Lo aprovechable, y es mucho:**

- **`/clarify`** — barrido de ambigüedad por taxonomía (alcance funcional, modelo de
  dominio, interacción, no-funcionales, bordes…), marca cada categoría como
  *Clear / Partial / Missing*, hace **como mucho 5 preguntas** dirigidas y escribe las
  respuestas **de vuelta en la spec**. Es exactamente el apartado *Preguntas abiertas*
  de §3.2 y el mandato «antes de escribir una spec, pregunta», pero con método.
- **`/analyze`** — pasada **de solo lectura** que cruza spec ↔ plan ↔ tareas contra la
  constitución del proyecto y reporta duplicación, adjetivos vagos sin criterio medible,
  requisitos sin objeto medible y huecos de cobertura. No edita: informa y espera. Es el
  control que hoy le falta a la puerta *Plan*.
- **`checklist-template`** — el checklist como artefacto **de revisor**: `[x]` significa
  «revisado y satisfecho», no «implementado».

### OpenSpec — `Fission-AI/OpenSpec` · MIT · `ed5d386`

Ligero y orientado a cambios (`propose → apply → archive`), pensado también para
brownfield. Su aportación real es **el formato del requisito**:

```markdown
### Requirement: Update Workflow Command
El sistema SHALL … y SHALL NOT …

#### Scenario: Revisar sin avanzar la frontera
- **WHEN** el usuario pide revisar un artefacto existente
- **THEN** la skill lo actualiza y reconcilia los demás
- **AND** no crea ninguno que aún no exista
```

Un requisito **no existe sin al menos un escenario**. Eso es, literalmente, el
criterio 3 de §3.2: *si no se puede comprobar, no es un criterio de aceptación*.

**No se instala** porque su unidad es `openspec/changes/<id>/` con `proposal.md`,
`design.md`, `tasks.md` y deltas de spec — una carpeta paralela a `specs/NNN-slug/`
que haría convivir dos procesos. Nota operativa: **clonarlo en Windows exige
`git -c core.longpaths=true`**; sin eso el checkout queda a medias y en silencio.

### GSD (Get Shit Done) — archivado

`gsd-build/get-shit-done` **ya no se mantiene**: su README solo dice que el proyecto
continúa en `open-gsd/gsd-core` (MIT, `af822a8`). El repo activo trae skills en formato
`SKILL.md`, pero cargan su lógica con `@~/.claude/gsd-core/workflows/…`: **sin instalar
el framework no hacen nada**. Además impone su propio ciclo (`PROJECT.md`, `ROADMAP.md`,
`STATE.md`, fases numeradas).

**Lo aprovechable:** `gsd-spec-phase` no pregunta hasta quedarse a gusto, sino que
**puntúa la ambigüedad** en cuatro dimensiones tras cada ronda de preguntas (máximo 6) y
**no deja escribir la SPEC hasta bajar de 0,20**. Convierte «no quedan preguntas
abiertas» en algo medible en vez de una impresión.

### BMAD-METHOD — `bmad-code-org/BMAD-METHOD` · MIT · `1b59caa`

Ciclo completo con personas (analista, PM, arquitecto, dev, QA) y ~32 skills. Su
`bmad-prd` es un facilitador serio, con dos caminos (rápido / acompañado) y marcas
`[ASSUMPTION]` sobre lo que infirió.

**No se instala:** cada skill arranca ejecutando `uv run {project-root}/_bmad/scripts/…`,
resuelve `customize.toml` y escribe un `.memlog.md`; sin ese andamiaje, no arranca. Y su
artefacto es un **PRD con épicas y sprints**, que no es lo que produce §3.2.

**Lo aprovechable:** marcar explícitamente en la spec lo que es suposición del agente y
no dato del usuario. Encaja con §11 (*un agente solo afirma lo que procede del canon, el
contexto o el usuario; lo demás es propuesta*).

### MUSUBI — `nahisaho/MUSUBI` · MIT · `d9c21f4`

El más riguroso sobre el papel: 9 artículos constitucionales, requisitos en formato
**EARS**, ADRs, C4, y un `traceability-auditor` que persigue requisito → diseño →
tarea → código → test.

**No se instala**, por tres razones que se acumulan: (1) sus 25 skills dependen de un
`steering/{structure,tech,product}.md` propio —18 referencias solo en
`requirements-analyst`— que duplicaría `CLAUDE.md` y `docs/`; (2) su constitución de 9
artículos **choca de frente** con la nuestra (manda *library-first* y *CLI interface* en
todo); (3) `requirements-analyst` declara que conduce el diálogo **en japonés**.

**Lo aprovechable:** EARS como gramática de requisito
(*Cuando `<disparador>`, el sistema deberá `<respuesta>`*) y la idea de auditar
trazabilidad de punta a punta: en este proyecto sería «toda regla de dominio de §8 tiene
un test que la comprueba», verificado en vez de prometido.

### Agent OS — `buildermethods/agent-os` · MIT · `475b0ca`

Cinco comandos markdown, casi sin dependencias: `discover-standards`, `index-standards`,
`inject-standards`, `plan-product`, `shape-spec`. Su tesis es buena para brownfield: lee
el código, **escribe las convenciones que de verdad usa** y luego le inyecta al agente
solo las pertinentes.

**No se instala** porque ese trabajo aquí ya está hecho y a mano: `CLAUDE.md` §5–§8 y
`docs/` *son* las convenciones, y mantenerlas coherentes ya es el oficio de
`coherencia-docs`. `shape-spec`, además, crea su propia estructura de specs.

**Lo aprovechable:** `shape-spec` exige **plan mode** y `AskUserQuestion` para todo, y se
declara «shaping, no documentación exhaustiva». Buena vacuna contra la spec que se
redacta sola.

## Opciones comerciales

No son descargables ni instalables como skills: son SaaS o un IDE. Se ha comprobado el
**2026-09-23** que los seis sitios responden (HTTP 200); nada más se ha verificado, y sus
capacidades aquí son las que anuncian, no las que hayamos medido.

| Producto | Qué promete | Encaje con este proyecto |
| --- | --- | --- |
| **EasySpecs** (`easyspecs.ai`) | Documenta primero el código real y sobre esa base genera specs con *Trust Specs* (validadores, casos borde, pruebas de rollback), verificadas en cascada; agnóstica del agente | Es lo más cercano a lo que aquí se llama *puerta de calidad*. Evaluable si alguna vez se quiere externalizar la verificación de specs; no sustituye a §3 |
| **Kiro** (AWS) | IDE dirigido por specs con *steering files* | Ata al IDE. Descartado: aquí se trabaja en Claude Code |
| **Tessl** | La spec como artefacto principal | Cambiaría el centro de gravedad del repositorio |
| **BrainGrid** | Planifica y descompone specs, y entrega al agente que se elija | Solapa con `writing-plans` |
| **CodeMySpec** | Comprueba que el código cumple la spec | Solapa con lo que harían `/analyze` y un auditor de trazabilidad |
| **Augment Code (Cosmos)** | Contexto persistente sobre bases de código grandes | El problema de contexto de este proyecto es el presupuesto de 100.000 tokens de §4.1, que es de diseño propio |

## Lo que falta, y es lo importante

Ninguna de las cinco skills instaladas conoce `specs/NNN-slug/spec.md`, ni los estados
`borrador → en-revision → aprobada → implementada`, ni `docs/definitions.md`. Aportan
**rigor de proceso**; no aportan **la forma de nuestra spec**.

Las tres técnicas que de verdad subirían la calidad de las specs de este proyecto están
en frameworks que no se pueden instalar, y son adaptables a una skill propia:

1. El **barrido de ambigüedad por taxonomía con ≤5 preguntas** de Spec Kit, escribiendo
   las respuestas en *Preguntas abiertas* → *Decisiones*.
2. La **puntuación de ambigüedad como puerta** de GSD: un umbral, no una sensación.
3. El **requisito sin escenario no es requisito** de OpenSpec, en el vocabulario de
   `docs/definitions.md`.

Eso sería una skill de este repositorio, no una descarga.
