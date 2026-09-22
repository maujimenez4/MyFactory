---
name: coherencia-docs
description: Detecta y resuelve inconsistencias entre los documentos de docs/ (architecture.md, definitions.md, domain-knowledge.md, verification.md): contradicciones factuales, deriva terminológica, referencias rotas, solapamiento de autoridad y contenido colocado en el documento equivocado. Usar cuando el usuario pida revisar la coherencia de la documentación, sospeche que dos documentos no concuerdan, antes de un merge que toque docs/, o tras un cambio de decisión que deba propagarse. Produce primero un informe, después un plan, y solo aplica cambios con aprobación explícita.
---

# Coherencia documental

Comparas los documentos de `docs/` entre sí, localizas inconsistencias, las clasificas y las resuelves. **Nunca editas sin aprobación explícita.**

## Reglas innegociables

1. **Informe antes que plan, plan antes que edición.** Tres pasos separados, con parada en cada uno. No propongas soluciones mientras aún estás leyendo: ancla el diagnóstico.
2. **Nada de escribir con el árbol sucio.** Ejecuta `git status` al empezar. Si hay cambios sin commitear en `docs/`, avisa y pide confirmación antes de tocar nada.
3. **Una incidencia = una edición atómica con ID.** El usuario debe poder aprobar por lotes.
4. **Ante la duda, informar en vez de resolver.** Una resolución equivocada en la fuente de verdad es peor que una inconsistencia detectada.
5. **No inventes contenido.** Si la resolución exige información que no está en ningún documento, es una decisión del usuario, no tuya.
6. **Cita siempre los dos lados.** Fichero, sección y texto literal de cada parte del conflicto. Sin cita, el usuario no puede juzgar.

---

## Jerarquía de autoridad

Cuando dos documentos se contradicen, decide **por tipo de afirmación**, no por antigüedad del fichero:

| Tipo de afirmación | Manda | Los demás documentos |
| --- | --- | --- |
| Qué significa un término, cómo se llama un concepto | `definitions.md` | Lo usan, no lo redefinen |
| Cómo funciona el dominio: estructura narrativa, género, arcos, reglas del mundo | `domain-knowledge.md` | Lo aplican |
| Decisiones técnicas: stack, estructura de código, agentes, proceso, almacenes | `architecture.md` | Lo referencian |
| Cómo se comprueba algo: criterios de aceptación, validaciones, umbrales, pruebas | `verification.md` | Lo invocan |

Reglas derivadas:

- Si `verification.md` valida una regla que `domain-knowledge.md` enuncia de otra forma, **manda la regla**: lo que se corrige es la verificación.
- Si `architecture.md` usa un término que no está en `definitions.md`, es un **hueco**, no una contradicción: se propone la definición, no se cambia la arquitectura.
- Si `definitions.md` define algo que nadie usa, es un **huérfano**: se informa, no se borra.
- Si dos documentos desarrollan a fondo el mismo asunto, gana el que manda según la tabla; el otro se reduce a una línea con enlace.

Si el repositorio tiene `docs/_authority.md`, esa tabla prevalece sobre esta.

---

## Criterio de pertenencia

Se usa para detectar contenido colocado en el documento equivocado:

- Si la frase cambiaría al cambiar de framework, de modelo o de base de datos → `architecture.md`.
- Si cambiaría aunque la novela se escribiera a mano → `domain-knowledge.md`.
- Si es «X significa Y» → `definitions.md`.
- Si describe cómo se comprueba que algo se cumple, con qué umbral o con qué prueba → `verification.md`.

---

## Taxonomía de inconsistencias

| Código | Tipo | Qué es | Resolución |
| --- | --- | --- | --- |
| `TER` | Terminológica | El mismo concepto con nombres distintos, o un término usado sin definir | Automática si hay forma canónica clara en `definitions.md` |
| `FAC` | Factual | Dos documentos afirman cosas incompatibles | **Decisión del usuario** |
| `EST` | Estructural | Referencias a secciones o ficheros inexistentes, numeración descuadrada, enlaces rotos | Automática |
| `AUT` | Autoridad | El mismo asunto desarrollado a fondo en dos documentos | Propuesta: cuál manda, el otro enlaza |
| `ALC` | Alcance | Contenido en el documento equivocado según el criterio de pertenencia | Propuesta de movimiento, nunca automática |
| `OBS` | Obsolescencia | Un documento refleja una decisión que otro ya cambió | **Decisión del usuario** |
| `HUE` | Hueco | Término definido que nadie usa, o concepto usado que nadie define | Solo informar |
| `VER` | Verificación desalineada | `verification.md` comprueba algo que ya no se corresponde con la regla de dominio o con la arquitectura | **Decisión del usuario** |

**Severidad:**

- **Alta** — la contradicción puede hacer que alguien implemente algo incorrecto (`FAC`, `OBS`, `VER`).
- **Media** — confunde pero no induce a error (`TER`, `AUT`, `ALC`).
- **Baja** — ruido (`EST`, `HUE`).

**Cubos de resolución:**

- **Automática** — grafías, enlaces rotos, referencias de sección. Se aplican juntas tras una sola aprobación.
- **Requiere decisión** — hay que elegir qué es verdad. Una pregunta por incidencia, con las dos opciones y la recomendación según la jerarquía de autoridad.
- **Solo informar** — sospecha sin confianza suficiente, o algo que necesita información que no está en el repositorio.

---

## Procedimiento

### Fase 0 — Preparación

1. `git status`; si `docs/` tiene cambios sin commitear, avisa.
2. Lee los cuatro documentos y anota, por cada uno: secciones (con su nivel), tamaño y fecha del último commit que lo tocó (`git log -1 --format=%cd -- docs/<fichero>`). La fecha importa: el más reciente suele ser el correcto en los conflictos de tipo `OBS`.
3. Comprueba si existe `docs/_authority.md`.

### Fase 1 — Pasada determinista

Barata, antes de gastar contexto en razonar. Busca con `grep`/`rg`:

- **Índice de términos.** Extrae los términos definidos en `definitions.md` (encabezados, entradas de tabla, texto en negrita). Para cada uno, cuenta apariciones en los otros tres documentos. Cero apariciones → candidato a `HUE`.
- **Variantes de escritura.** Mismo término con y sin acento, singular/plural, mayúscula/minúscula, `snake_case` frente a prosa, inglés frente a castellano (`beat` / `latido`, `scene` / `escena`). Candidatos a `TER`.
- **Enlaces y referencias.** Rutas relativas a ficheros que no existen, anclas `#seccion` que no corresponden a ningún encabezado, referencias tipo «§4.2» o «ver sección X» que no existen en el documento citado. Candidatos a `EST`.
- **Números y umbrales.** Extrae todas las cifras con unidad (tokens, porcentajes, palabras, plazos) y agrúpalas por concepto. Si el mismo concepto tiene dos cifras distintas en documentos distintos, candidato a `FAC` de severidad alta.
- **Listas enumeradas.** Las mismas listas (roles, fases, tipos, códigos) en varios documentos: compara elementos y orden. Diferencias → `FAC` u `OBS`.
- **Marcadores.** `TODO`, `TBD`, `pendiente`, `por confirmar`, casillas sin marcar.

### Fase 2 — Pasada semántica

Solo sobre lo que la Fase 1 no puede ver. Compara **por pares**, en este orden de prioridad:

1. `definitions.md` ↔ `domain-knowledge.md` (el par que más deriva)
2. `domain-knowledge.md` ↔ `verification.md` (reglas frente a cómo se comprueban)
3. `architecture.md` ↔ `verification.md` (mecanismos frente a criterios)
4. `definitions.md` ↔ `architecture.md`
5. `domain-knowledge.md` ↔ `architecture.md`
6. `definitions.md` ↔ `verification.md`

Para cada par, no releas ambos documentos enteros. Trabaja sobre los **conceptos compartidos** que dio el índice de términos: para cada concepto, extrae lo que afirma cada documento y compáralo.

En cada par, busca específicamente:

- Afirmaciones incompatibles sobre el mismo concepto (`FAC`).
- Un documento que asume una decisión que el otro ya revisó (`OBS`).
- Solapamiento: ambos explican lo mismo a fondo (`AUT`).
- Contenido que, por el criterio de pertenencia, vive en el documento equivocado (`ALC`).
- Reglas enunciadas sin verificación asociada, o verificaciones de reglas que ya no existen (`VER`).

### Fase 3 — Informe (PARADA)

Presenta el informe y **detente**. No propongas resoluciones todavía más allá de la columna de propuesta.

```markdown
# Informe de coherencia — docs/
Documentos: 4 · Conceptos comparados: N · Incidencias: N (alta A / media M / baja B)

## Resumen
Dos o tres frases con el patrón dominante. Por ejemplo: «La mayoría de las
incidencias son de alcance: architecture.md contiene material de dominio.»

## Incidencias

### Alta severidad
| ID | Tipo | Concepto | Documentos | Cubo |
| --- | --- | --- | --- | --- |
| INC-001 | FAC | Límite de contexto | architecture.md §2.1 ↔ verification.md §3 | Decisión |

**INC-001 · FAC · Límite de contexto**
- `architecture.md` §2.1: «[cita literal]»
- `verification.md` §3: «[cita literal]»
- Según la jerarquía, manda architecture.md en decisiones técnicas → la
  verificación debería comprobar el valor de architecture.md.
- Confianza: alta.

[… una entrada por incidencia …]

## Huecos detectados
Términos usados sin definir · Términos definidos sin usar.

## No revisado
Qué ha quedado fuera y por qué.
```

Termina preguntando: **«¿Sigo con el plan de resolución, o quieres ajustar algo del diagnóstico primero?»**

### Fase 4 — Plan (PARADA)

Agrupado por cubo, no por documento:

- **Lote A · Automáticas** (`TER`, `EST`): lista de ediciones concretas, fichero y línea. Una sola aprobación para todo el lote.
- **Lote B · Requieren decisión** (`FAC`, `OBS`, `VER`): una pregunta por incidencia, con las dos opciones, la recomendación según la jerarquía y qué ficheros cambiarían con cada una.
- **Lote C · Reestructuración** (`AUT`, `ALC`): movimientos de contenido propuestos, con origen y destino. Siempre revisión individual: mover texto entre documentos es lo que más fácil rompe la narrativa de un documento.
- **Lote D · Solo informar**: sin acción.

Indica en cada lote cuántas líneas se tocarían. Si el plan supera las 30 incidencias, propón hacerlo en dos tandas empezando por severidad alta.

### Fase 5 — Aplicación

Solo tras aprobación explícita, y solo de los lotes aprobados.

1. Aplica lote por lote, en orden A → B → C.
2. Cada edición conserva el estilo del documento: tono, formato de tabla, nivel de encabezado, longitud de párrafo. No reescribas secciones enteras para arreglar una frase.
3. Tras cada lote, muestra el `git diff --stat`.
4. Si al aplicar una edición aparece una inconsistencia nueva que no estaba en el informe, **para y repórtala**; no la resuelvas sobre la marcha.
5. Al terminar: propón un commit por lote, con mensaje que cite los IDs (`docs: resolver INC-003, INC-007 (terminología)`). No commitees sin permiso.

### Fase 6 — Cierre

Tres líneas: qué se resolvió, qué quedó pendiente de decisión y qué debería revisar el usuario a mano.

---

## Uso del contexto

Con cuatro documentos que pueden ser largos, no cargues todo a la vez:

- La Fase 1 trabaja con `grep` y `git`, no con los documentos en contexto.
- La Fase 2 trabaja sobre **extractos por concepto**, no sobre documentos completos.
- Si un documento supera lo que cabe cómodamente, procésalo por secciones y mantén un índice de sus afirmaciones en lugar de releerlo.
- Nunca compares los cuatro documentos a la vez: siempre por pares.

## Qué no es esto

- No es un corrector de estilo. No propongas mejoras de redacción que no resuelvan una inconsistencia.
- No es un generador de documentación. No rellenes huecos escribiendo contenido nuevo.
- No es un linter de markdown. Formato, ortografía y longitud de línea quedan fuera salvo que rompan una referencia.