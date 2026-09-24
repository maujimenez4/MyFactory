# Fase 3 · Skill de prácticas para repositorios grandes (diseño, no creada)

Consultado el 2026-09-24. Entrada: las doce prácticas de «incorporar ya» de
[`fase2-clasificacion.md`](fase2-clasificacion.md). Confianza: **oficial** (docs de Claude Code),
**divulgación** (terceros). **Aquí no se crea ni se instala nada**: esto es el diseño.

## Qué es y cómo funciona

**Nombre propuesto: `repo-grande`.** Una skill propia de MyFactory que reúne las decisiones de
sesión en un repositorio que no cabe en contexto. No es documentación: es un procedimiento corto
que se dispara cuando toca decidir por dónde entrar.

Frontmatter propuesto (los campos son **oficiales**):

```yaml
---
name: repo-grande
description: Cómo trabajar en un repositorio grande o desconocido (monorepo, cientos de miles o
  millones de líneas) sin gastar el contexto: dónde arrancar la sesión, qué delegar a un subagente,
  en qué orden explorar y cómo acotar la tarea a un módulo.
when_to_use: Usar al empezar en un repositorio que no conoces o que no cabe en contexto; al
  planificar una exploración ("¿dónde está X?", "cómo funciona Y"); antes de un cambio que cruza
  varios paquetes; cuando la sesión se compacta más de una vez; o cuando se pregunta cómo bajar el
  gasto de tokens en este repo. No usar para una tarea de un solo fichero ya localizado.
---
```

`description` + `when_to_use` se concatenan y se truncan a **1.536 caracteres** (oficial); la
propuesta cabe holgada. La última frase es tan importante como las demás: una skill sin frontera de
«cuándo no» se carga siempre.

**Qué contiene el cuerpo** (SKILL.md, objetivo < 120 líneas; el tope oficial es 500): seis
decisiones con su regla, nada de teoría. Arrancar dentro del paquete y no en la raíz (P-8);
explorar en tres capas, arquitectura → módulo → fichero, sin abrir un fichero hasta tener el mapa
(P-9); delegar el barrido a un subagente y quedarse con la conclusión (P-1); acotar la tarea a un
módulo y decirlo en el prompt (P-10); lo estable arriba y lo variable abajo, para no invalidar la
caché (P-5); y elegir modelo por tarea (P-7). Más dos disparadores hacia ficheros de apoyo: si el
paquete no tiene `CLAUDE.md`, ir a la plantilla; si un barrido devuelve `dist/` o `vendor/`, ir al
bloque de exclusiones.

**Revelación progresiva** (oficial): SKILL.md navega, el detalle vive en ficheros vecinos que solo
se leen si hacen falta — `plantilla-claude-md.md`, `exclusiones.md`, `medicion.md`.

**Qué NO debe hacer.** No llevar `context: fork`: una skill que corre en un subagente aislado no
cambia el comportamiento de la sesión principal, que es justo lo que se busca. No repetir lo que ya
dice el `CLAUDE.md` del proyecto: si dos instrucciones se contradicen, el modelo elige una al azar
(oficial). No explicar qué es el prompt caching (P-4): es automático y no hay nada que decidir. No
enseñar a escribir skills (P-6): eso es cómo se fabrica esta, no lo que dice.

## Cómo se instala y configura (no realizado)

1. Copiar la carpeta `repo-grande/` de `.claude/Skills/` de MyFactory a `.claude/skills/` del
   proyecto destino, fichero a fichero, y anotar su fila en el `SOURCES.md` de ese proyecto.
2. Ámbito: `.claude/skills/` la carga en ese repo; `~/.claude/skills/`, en todos los proyectos de la
   máquina (oficial). Para consultoría, **proyecto**: se revisa en el PR y se retira al cerrar. En un
   monorepo puede colgar de un subdirectorio y cargarse solo al trabajar ahí dentro (oficial).
3. Sin `allowed-tools` ni `hooks`: no hace falta pre-aprobar nada, la skill solo dice cómo trabajar.
4. Verificación mínima: que aparece en el listado de skills, y que se dispara con una pregunta de
   exploración real y **no** con «arréglame este test».

## Ahorro de tokens esperado y cómo medirlo

**Por sí sola no ahorra nada, y cuesta.** Su `description` está en contexto en cada sesión, y su
cuerpo, una vez invocado, permanece **a lo largo de los turnos siguientes** (oficial). Solo ahorra
si cambia el comportamiento: si por leerla el agente delega el barrido en vez de abrir treinta
ficheros. El ahorro es de la conducta, no del texto. **No hay cifra publicada de esto y no la
inventamos.**

Cómo se mediría: banco de 15-20 tareas reales del repo (localizar, explicar, cambiar, encontrar
usos); A/B con el mismo modelo, con y sin la skill, repetido varias veces por la varianza. Por
tarea: tokens de entrada, salida, `cacheRead` y `cacheCreation`; coste; llamadas a herramienta;
compactaciones; y **si la tarea salió bien**, juzgado por una persona. Un ahorro con peor tasa de
acierto no es un ahorro.

De dónde salen los números (oficial): telemetría OTEL, `CLAUDE_CODE_ENABLE_TELEMETRY=1` más
`OTEL_METRICS_EXPORTER`. `claude_code.token.usage` y `claude_code.cost.usage` traen `type`
(input/output/cacheRead/cacheCreation), `model`, `query_source` (`main`/`subagent`/`auxiliary`),
`agent.name` y **`skill.name`**. Con `query_source` se ve si el trabajo se movió al subagente, que
es el efecto buscado. Aviso de lectura: `skill.name` atribuye las peticiones hechas **mientras la
skill está activa**, no el ahorro que causó. Sirve para su coste, no para su beneficio; el
beneficio solo sale del A/B.

## Riesgos y limitaciones, incluida la seguridad del código del cliente

- **El riesgo principal es el que se pide mirar: contexto pagado en cada sesión.** La descripción se
  paga siempre; el cuerpo, en cada sesión que la invoque y hasta que compacte. Y hay un segundo
  coste escondido: al compactar, Claude Code vuelve a adjuntar los **primeros 5.000 tokens por
  skill**, con un presupuesto combinado de 25.000 (oficial). Una skill hinchada se come ese
  presupuesto y desplaza a otras. Por eso el tope propio de 120 líneas es más estricto que el
  oficial de 500.
- **Cuándo estorbaría:** en repos pequeños, en sesiones de un solo fichero, y en un equipo que ya
  trabaja así — ahí es texto que repite lo obvio. Si al medir se ve que se dispara donde no hace
  falta, hay dos salidas oficiales: acotar con `paths` (globs que limitan cuándo se activa) o
  ponerla en `disable-model-invocation: true`, que la saca del contexto y la deja solo como comando
  manual. Esa segunda es la palanca real contra el coste fijo.
- **Contradicciones y ruido.** Si el proyecto ya tiene reglas en `CLAUDE.md`, dos instrucciones
  distintas sobre lo mismo degradan el cumplimiento (oficial): la skill cede ante el `CLAUDE.md`
  del cliente y lo dice en su primera línea. Y la mitad de las doce prácticas son juicio, no
  receta: como máximas («explora por capas») no cambian nada, así que van como reglas comprobables.
- **Seguridad.** La skill no contiene código del cliente ni lo mueve: es texto nuestro. El riesgo
  está en lo que **induce a escribir**. (a) Un `CLAUDE.md` de paquete o un documento de arquitectura
  describen la estructura interna del cliente y **se commitean en su repo**: la plantilla prohíbe
  pegar código, rutas de infraestructura, credenciales o datos personales. (b) Las reglas de
  exclusión (`permissions.deny` con `Read(...)`) sí bloquean la lectura, pero son higiene de
  contexto, **no una frontera de seguridad**: un secreto se saca del repo, no se tapa con una regla.
  (c) Todo lo que el agente lee viaja al proveedor del modelo; la skill reduce ese volumen, no lo
  elimina.
- **Límite último:** no da capacidades nuevas. No indexa, no entiende símbolos y no sustituye a
  Serena ni a ast-grep. Ordena el trabajo; no ve más.

## Cómo se integraría en MyFactory

Carpeta `.claude/Skills/repo-grande/` en MyFactory, de donde se copia a `.claude/skills/` del
proyecto destino. Fila en `SOURCES.md`: `repo-grande` · **Propia de MyFactory** · commit — · sin
licencia declarada · 2026-09-24 · «Prácticas de sesión para repositorios que no caben en contexto».

Ficheros que la acompañan, todos cargados bajo demanda desde SKILL.md:

| Fichero | Qué es |
| --- | --- |
| `SKILL.md` | Las seis decisiones. < 120 líneas |
| `plantilla-claude-md.md` | Dos plantillas: raíz (corto, objetivo < 100 líneas; el tope oficial recomendado es 200) y de paquete, que **se carga solo cuando se leen ficheros de esa carpeta** (oficial). Cubre P-2, P-3 y P-12 |
| `exclusiones.md` | Bloque `permissions.deny` de `.claude/settings.json` con `Read(dist/**)`, `Read(vendor/**)`, generados y datos, y el aviso de que una regla `deny` con ruta relativa casa a cualquier profundidad (oficial). Cubre P-11 |
| `medicion.md` | El A/B y las variables OTEL de arriba, para poder afirmar el ahorro en vez de prometerlo |

Alternativa oficial que conviene conocer y **no** usar aquí: `.claude/rules/` con campo `paths`
carga instrucciones solo al tocar ficheros que casan con un glob. Mejor que una skill para reglas
por tipo de fichero; peor para un procedimiento de arranque, que es lo que esto es.

No se entrega a un cliente sin pasar antes la medición de `medicion.md` en un repositorio propio
grande. Hoy no tenemos uno: `ciberpunk-storymaker` es pequeño y no sirve de banco.

## Fuentes

Todas consultadas el 2026-09-24. Todas oficiales.

- https://code.claude.com/docs/en/skills — frontmatter, 1.536 caracteres, < 500 líneas,
  ubicaciones, `paths`, `disable-model-invocation`, `context: fork`, ficheros de apoyo, los
  5.000/25.000 tokens al compactar, skill frente a `CLAUDE.md`.
- https://code.claude.com/docs/en/memory — jerarquía de `CLAUDE.md`, carga bajo demanda en
  subdirectorios, objetivo de 200 líneas, `.claude/rules/` con `paths`, `claudeMdExcludes`.
- https://code.claude.com/docs/en/monitoring-usage — métricas y atributos de tokens y coste.
- https://code.claude.com/docs/en/permissions — `Read(...)` en `permissions.deny`, sintaxis
  gitignore y profundidad de casado.
