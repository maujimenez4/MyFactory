# Fase 2 · Clasificación

Investigación A, fase 2 (cerrar). Fecha: **2026-09-24**. Entrada: las 22 estrategias de
[`fase1-exploracion.md`](fase1-exploracion.md).

## Cómo se puntúa

El método pide ordenar por **impacto, esfuerzo y riesgo**. Se añade un cuarto criterio,
**evidencia**, porque en esta investigación es lo que más separa unas estrategias de
otras: hay cifras de Anthropic y hay cifras de quien vende consultoría de optimización.

De 1 a 5, y **en los cuatro 5 es mejor**: 5 en esfuerzo significa *cuesta poco*, y 5 en
riesgo significa *riesgo bajo*.

| Criterio | Qué mide |
|---|---|
| **Impacto** | Cuánto baja el coste por tarea completada |
| **Esfuerzo** | 5 = se aplica hoy sin construir nada |
| **Riesgo** | 5 = no degrada la tarea ni añade fragilidad |
| **Evidencia** | 5 = medido y publicado por Anthropic · 3 = paper · 1 = blog comercial |

---

## Ranking

| # | Estrategia | Impacto | Esfuerzo | Riesgo | Evidencia | **Total** |
|---|---|:--:|:--:|:--:|:--:|:--:|
| C-2 | Prompt caching | 5 | 5 | 5 | 5 | **20** |
| A-3 | Subagentes con contexto limpio | 5 | 5 | 4 | 5 | **19** |
| B-1 | Herramientas que no vuelcan | 5 | 4 | 5 | 5 | **19** |
| C-3 | Orden del prompt para la caché | 4 | 5 | 5 | 5 | **19** |
| B-4 | Apagar las herramientas que no se usan | 3 | 5 | 5 | 5 | **18** |
| C-5 | Salidas estructuradas | 4 | 4 | 5 | 5 | **18** |
| A-4 | Recuperación *just-in-time* | 4 | 4 | 4 | 5 | **17** |
| D-1 | Tareas acotadas | 4 | 5 | 4 | 4 | **17** |
| D-2 | Reintento dirigido, nunca genérico | 4 | 4 | 5 | 4 | **17** |
| B-3 | Muchas búsquedas pequeñas | 3 | 5 | 4 | 5 | **17** |
| E-1 | Coste por tarea completada con éxito | 4 | 4 | 5 | 4 | **17** |
| C-4 | Batch API para lo no interactivo | 4 | 4 | 4 | 5 | **17** |
| A-2 | Notas estructuradas fuera del contexto | 4 | 4 | 3 | 5 | **16** |
| E-4 | Leer coste y tasa de éxito juntos | 3 | 4 | 5 | 4 | **16** |
| D-3 | Techo de gasto por tarea, con corte | 4 | 3 | 5 | 3 | **15** |
| E-3 | Traza por rol y herramienta | 3 | 3 | 4 | 5 | **15** |
| E-2 | Línea base con tareas reales | 3 | 2 | 5 | 5 | **15** |
| D-4 | Instrucciones cortas y repartidas | 3 | 5 | 4 | 2 | **14** |
| A-1 | Compactación | 3 | 5 | 2 | 5 | **15** |
| C-1 | Enrutamiento por complejidad | 5 | 3 | 3 | **1** | **12** |
| B-2 | Ejecución de código con MCP | 5 | 1 | 2 | 5 | **13** |
| A-5 | Contexto curado frente a completo | — | — | — | 3 | *(no es estrategia: es la medida del problema)* |

---

## Grupos

### Aplicar ya

Nada de esto exige construir: es configuración y orden de trabajo.

| Estrategia | Por qué entra ya |
|---|---|
| **C-2 + C-3 Caché y orden del prompt** | El ahorro mayor con el menor esfuerzo. Y van juntas: la caché sin el orden correcto **cobra el +25 % de escritura y no acierta nunca** |
| **A-3 Subagentes** | Gasta decenas de miles fuera y devuelve mil. Es la pieza de mayor impacto estructural |
| **B-1 Herramientas que no vuelcan** | Paginar y truncar por defecto. Se arregla una vez y sirve siempre |
| **B-4 Apagar lo que no se usa** | Gratis. Cada herramienta declarada se paga en cada llamada |
| **C-5 Salidas estructuradas** | Ahorra sobre todo en **reintentos**, que es donde se esconde el coste |
| **B-3, D-1, D-2** | Tres reglas de trabajo sin nada que mantener |
| **E-1, E-4** | La medición entra **desde el principio**, no al final: sin línea base, lo demás no se puede afirmar |

### Probar en piloto

| Estrategia | Qué hay que resolver |
|---|---|
| **A-4 Just-in-time** (17) | Cambia cómo se escriben las herramientas. Piloto: un flujo, medir antes y después |
| **C-4 Batch** (17) | 50 % es mucho, pero solo sirve para lo que tolera espera. Hay que identificar qué parte del trabajo lo tolera |
| **A-2 Notas fuera del contexto** (16) | Funciona —la bitácora de `ciberpunk-storymaker` lo demuestra— pero mal hecha dispersa el estado en ficheros que nadie relee |
| **D-3 Techo con corte** (15) | Alto valor y poca evidencia publicada. Es la red que evita que un bucle malo se coma el ahorro de todo lo demás |
| **E-2 Línea base** (15) | Esfuerzo alto —hay que construir el banco de tareas— y es **condición** de todo lo demás. Va primero en el tiempo aunque no lidere la tabla |

### Tratar con cuidado, no descartar

| Estrategia | Por qué |
|---|---|
| **C-1 Enrutamiento por complejidad** (12) | **Impacto 5 y evidencia 1.** La cifra del 73-87 % viene de un blog comercial. La idea es sólida y probablemente la palanca mayor, pero **el número no es creíble hasta medirlo**. El riesgo tampoco es cero: enrutar mal manda a un modelo barato lo que necesitaba juicio, y eso se paga en reintentos. Piloto con medición propia, no adopción directa |
| **B-2 Ejecución de código con MCP** (13) | **−98,7 % es la cifra más grande de toda la investigación y es oficial.** Baja por esfuerzo y riesgo: exige entorno de ejecución, con lo que eso implica en seguridad al correr código sobre datos de cliente. Candidata clara a fase 3, no a aplicar ya |
| **A-1 Compactación** (15) | Puntúa decente pero **no es una estrategia de ahorro: es una red de seguridad que pierde detalle**. Se deja activa; no se cuenta como palanca |
| **D-4 Instrucciones cortas** (14) | Sensata y barata, pero su respaldo es divulgación. Se adopta por criterio, sabiendo que no hay evidencia detrás |

---

## Dos cosas que el ranking por sí solo no dice

**1 · Las estrategias se pisan entre ellas.** Subagentes, compactación, notas fuera del
contexto y just-in-time atacan **el mismo gasto**: el contexto que se arrastra turno a
turno. Sumar sus porcentajes daría un ahorro imposible. El orden sensato es aplicar
primero la de mayor impacto y **volver a medir**, no apilarlas.

Las que sí son independientes y se suman de verdad: **caché** (precio del token),
**batch** (precio de la llamada), **enrutamiento** (precio del modelo) y **contexto**
(número de tokens). Cuatro palancas sobre cuatro factores distintos.

**2 · La medición no es una fase, es la primera tarea.** E-2 puntúa 15 y va la primera en
el tiempo. Sin línea base con tareas reales del cliente, ningún porcentaje posterior
significa nada — y este informe está lleno de porcentajes ajenos que lo demuestran.

## Lo que sigue sin cubrirse

**La mitad de generación de código.** La métrica del roadmap habla de *coste por historia
o PR aceptada*, y no se ha encontrado ninguna fuente con cifras sobre eso. Todo lo
recogido mide tokens por tarea de agente. Es un hueco que la fase 3 debería cerrar, o
declarar explícitamente que se medirá en el piloto y no antes.
