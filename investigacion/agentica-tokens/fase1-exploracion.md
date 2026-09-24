# Fase 1 · Exploración: bajar el consumo de tokens en una solución agéntica

Investigación A, fase 1 (abrir). Fecha: **2026-09-24**.

Objetivo: reunir las **estrategias** que reducen el gasto de tokens de un sistema
agéntico y de la generación de código, para clasificarlas después y decidir cuáles
entran en el trabajo con clientes.

Métrica que manda, fijada en el roadmap: **coste por tarea completada con éxito** y
**coste por historia o PR aceptada**. No «tokens por llamada»: una estrategia que ahorra
tokens y hace fracasar la tarea sale más cara, porque hay que repetirla entera.

## Cómo leer las cifras de esta página

Están separadas en tres niveles de confianza, y la diferencia importa más que el número:

| Nivel | Qué significa |
|---|---|
| **Oficial** | Publicado por Anthropic en su documentación o en su blog de ingeniería |
| **Paper** | Artículo con método descrito; medido en su banco de pruebas, no en el nuestro |
| **Divulgación** | Blog comercial o artículo de terceros. **Se recoge, no se cree** |

Ninguna cifra está medida por nosotros. Esa es la primera tarea de cualquier piloto.

---

## 1 · Arquitectura del contexto

El principio que ordena todo este bloque, en palabras de Anthropic: buscar **el conjunto
más pequeño de tokens de alta señal** que consiga el resultado.

| # | Estrategia | Qué es y qué problema resuelve | Cifra | Fuente |
|---|---|---|---|---|
| A-1 | **Compactación** | Resumir la conversación cerca del límite y reabrir ventana con el resumen. Evita que una tarea larga muera; a cambio pierde detalle | — | [Anthropic, context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · **Oficial** |
| A-2 | **Notas estructuradas fuera del contexto** | El agente escribe su progreso en ficheros y los relee. La memoria deja de pagarse en cada turno | — | [ídem](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · **Oficial** |
| A-3 | **Subagentes con contexto limpio** | Un subagente explora gastando decenas de miles de tokens y devuelve **1.000-2.000**. Lo caro se quema fuera de la conversación principal | Devuelve ~1-2k | [ídem](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · **Oficial** |
| A-4 | **Recuperación *just-in-time*** | Guardar identificadores ligeros —rutas, consultas, enlaces— y cargar el dato al usarlo, en vez de precargarlo por si acaso | — | [ídem](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · **Oficial** |
| A-5 | **Contexto completo frente a contexto curado** | La comparación que da la magnitud del problema: un agente que lo mete todo gasta **1.480.996** tokens en un banco de 50 tareas frente a **553.374** uno optimizado | **2,68×** | [arXiv 2606.10209](https://arxiv.org/pdf/2606.10209) · **Paper** |

## 2 · Diseño de las herramientas

| # | Estrategia | Qué es y qué problema resuelve | Cifra | Fuente |
|---|---|---|---|---|
| B-1 | **Herramientas que no vuelcan** | Paginación, rangos, filtros y truncado **con valores por defecto sensatos**. Una herramienta que devuelve 10.000 filas se come la ventana sola | — | [Anthropic, tools para agentes](https://www.anthropic.com/engineering/writing-tools-for-agents) · **Oficial** |
| B-2 | **Ejecución de código con MCP** | El agente carga solo las herramientas que necesita y **filtra los datos en el entorno de ejecución** antes de devolverlos. En su ejemplo, de **150.000 a 2.000 tokens** | **−98,7 %** | [Anthropic, code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) · **Oficial** |
| B-3 | **Muchas búsquedas pequeñas, no una amplia** | Varias consultas dirigidas devuelven menos ruido que un barrido único. Es una instrucción al agente, no una herramienta | — | [Anthropic, context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · **Oficial** |
| B-4 | **Definiciones de herramientas que se apagan** | Cada herramienta declarada ocupa sitio en el prompt de sistema en **todas** las llamadas. Las que no se usan se quitan | — | [Anthropic, plugins](https://anthropic.com/news/claude-code-plugins) · **Oficial** |

## 3 · Modelo y forma de la llamada

| # | Estrategia | Qué es y qué problema resuelve | Cifra | Fuente |
|---|---|---|---|---|
| C-1 | **Enrutamiento por complejidad** | Modelo barato ejecuta, caro solo aconseja cuando hace falta. La fuente afirma sesiones **73-87 % más baratas** que usar el caro para todo | −73 a −87 % | [Morph, LLM cost optimization](https://www.morphllm.com/llm-cost-optimization) · **Divulgación** |
| C-2 | **Prompt caching** | Reutiliza el prefijo ya enviado. Escribir cuesta **+25 %**; leer, **el 10 %** del precio de entrada | Hasta −90 % en entrada | [docs, prompt caching](https://docs.claude.com/en/docs/build-with-claude/prompt-caching) · **Oficial** |
| C-3 | **Orden del prompt para que la caché acierte** | Lo estable arriba, lo variable abajo. Si lo que cambia va primero, **la caché no acierta nunca** y el +25 % de escritura se paga sin contrapartida | — | [ídem](https://docs.claude.com/en/docs/build-with-claude/prompt-caching) · **Oficial** |
| C-4 | **Batch API para lo que no es interactivo** | Trabajo de fondo —auditorías, evaluaciones, lotes— al **50 %** | −50 % | [docs, pricing](https://docs.anthropic.com/en/docs/about-claude/pricing) · **Oficial** |
| C-5 | **Salidas estructuradas** | Un esquema estricto acorta la respuesta y, sobre todo, **quita reintentos**: cada respuesta malformada es una llamada entera pagada dos veces | — | [Anthropic, advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use) · **Oficial** |

## 4 · Proceso de trabajo

| # | Estrategia | Qué es y qué problema resuelve | Cifra | Fuente |
|---|---|---|---|---|
| D-1 | **Tareas acotadas** | Una tarea, un ámbito. Cruzar tres áreas en una sesión multiplica el contexto y la deriva | — | [Anthropic, patrones avanzados](https://www.anthropic.com/webinars/claude-code-advanced-patterns) · **Oficial** |
| D-2 | **Reintento dirigido, nunca genérico** | «Mejóralo» reenvía todo el contexto para nada. El reintento lleva el defecto concreto con su cita. Ya es regla nuestra en `ciberpunk-storymaker` (§15) | — | Práctica propia |
| D-3 | **Techo de gasto por tarea, con corte** | Un presupuesto que, al agotarse, **para y escala a una persona** en vez de seguir reintentando. Sin corte, un bucle malo se come el ahorro de todo lo demás | — | Práctica propia |
| D-4 | **Instrucciones cortas y repartidas** | El fichero de instrucciones se paga en **cada** sesión. Lo común arriba, lo específico junto a lo que describe | — | [guía monorepo](https://www.lowcode.agency/blog/claude-code-monorepo) · **Divulgación** |

## 5 · Medición, sin la cual nada de lo anterior se puede afirmar

| # | Estrategia | Qué es y qué problema resuelve | Fuente |
|---|---|---|---|
| E-1 | **Coste por tarea completada con éxito** | La métrica del roadmap. Cuenta los reintentos y los fracasos, que es donde se esconde el gasto | Roadmap |
| E-2 | **Línea base con tareas reales** | Un banco de tareas del cliente, medido **antes** de tocar nada. Sin esto, cualquier porcentaje posterior es una opinión | [arXiv 2606.10209](https://arxiv.org/pdf/2606.10209) · **Paper** |
| E-3 | **Traza por rol y por herramienta** | Un span por agente y por llamada permite ver **dónde** se van los tokens. Es lo que ya hacemos con Langfuse en `ciberpunk-storymaker` | Práctica propia |
| E-4 | **Vigilar la tasa de éxito junto al coste** | Una estrategia que baja tokens y sube fracasos encarece la tarea. Las dos cifras se leen juntas o no se leen | Roadmap |

---

## Lo que esta fase deja abierto

1. **Ninguna cifra es nuestra.** Las de Anthropic son fiables en su contexto; las de
   divulgación —el 73-87 % del enrutamiento, el 70-85 % de «cinco palancas»— vienen de
   quien vende la optimización.
2. **Casi todas las estrategias se solapan.** Subagentes, compactación y notas atacan el
   mismo gasto; sumar sus porcentajes daría un ahorro imposible. La fase 2 tiene que
   decir cuáles se pisan.
3. **Falta el coste de implantar cada una.** Apagar herramientas es gratis; montar
   ejecución de código con MCP, no.
4. **No hay ninguna cifra sobre generación de código** —coste por PR aceptada— en las
   fuentes encontradas. Es la mitad del encargo y está sin cubrir.
