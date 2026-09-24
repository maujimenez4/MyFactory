# Fase 3 · Enrutamiento de modelos por complejidad

Investigación A, fase 3. Estrategia **C-1**. Fecha: **2026-09-24**.
En [fase 2](fase2-clasificacion.md) quedó como *tratar con cuidado*: impacto 5, evidencia 1.

La métrica que manda es **coste por tarea completada con éxito**. Anthropic lo dice igual:
«compare models on cost per completed task» (oficial). Un token barato que falla no es barato.

## Qué es y cómo funciona

Tres cosas distintas bajo el mismo nombre.

| Variante | Quién decide | Cuándo | Coste del error |
|---|---|---|---|
| **Estático por tipo de tarea** | Tú, al configurar | Antes | Se paga entero: si el barato falla, falla |
| **Escalado (cascada)** | Una puerta de verificación | Tras fallar | El intento barato **se suma** al caro |
| **Asesor puntual** | El propio ejecutor | Durante | Consulta de más, o de menos |

El **estático** no tiene lógica de decisión: es configuración. El **escalado** exige una
puerta automática fiable; sin señal de fallo es un reintento a ciegas (es el diseño de
TRIAGE). En el **asesor**, el caro **no genera la salida final** — ahí está el ahorro; de
esta variante salen las cifras grandes, y es producto oficial de Anthropic (en beta).

## Cómo se configura

*Solo lectura de documentación: no se ha instalado ni ejecutado nada.*

**Claude Code** — `settings.json`: `model`, `modelSettings` con `effortLevel` por modelo,
`availableModels`, `fallbackModel`. Entorno: `CLAUDE_CODE_SUBAGENT_MODEL` (y su `_FORCE`),
más `ANTHROPIC_DEFAULT_OPUS_MODEL` / `_SONNET_MODEL` / `_HAIKU_MODEL`, que fijan a qué
resuelve cada alias. Cada subagente declara `model:` en su frontmatter (`haiku`, `sonnet`,
`opus`, `inherit`). `/model opusplan` ya es enrutamiento estático por fase: planifica en
Opus, ejecuta en Sonnet.

**API** — herramienta `advisor`, beta `advisor-tool-2026-03-01`:

```json
{"model": "claude-sonnet-5",
 "tools": [{"type": "advisor_20260301", "name": "advisor", "model": "claude-opus-5"}]}
```

Cuatro reglas que condicionan el diseño: el asesor debe ser **al menos tan capaz** como el
ejecutor (par inválido → `400`); `max_uses` limita consultas **por petición**, no por
conversación; el `caching` del asesor solo compensa **a partir de tres consultas**; y los
tokens del asesor **no entran** en el `usage` de nivel superior — van en `usage.iterations[]`.
Un contador que sume solo el `usage` superior **no ve el gasto del asesor**.

## Ahorro esperado y cómo medirlo

| Cifra | Qué compara de verdad | Confianza |
|---|---|---|
| **−11,9 %** coste/tarea, **+2,7 pp** en SWE-bench Multilingual | Sonnet 4.6 + asesor Opus **vs Sonnet solo** | **Oficial** |
| **−85 %** coste/tarea; BrowseComp 41,2 % vs 19,7 % en solitario, pero **−29 % de puntuación** | Haiku + asesor Opus **vs Sonnet solo** | **Oficial** |
| Opus 5.5 a `medium`: 92,8 % de SWE-bench Pro a **$0,22**/tarea resuelta; Fable 5.1, 92,3 % a **$1,19** | El caro, más barato **por tarea** | **Oficial** |
| Sonnet 5: 77,4 % a $0,84; Fable 5.1 a `low`: 88,6 % a **$0,54** | El caro gana también aquí | **Oficial** |
| Haiku: 63 % en GPQA Diamond frente a 92 % de Opus 5.5, a ~1/5 del coste | El barato, a dos tercios del acierto | **Oficial** |
| **−73 a −87 %** por sesión | Morph, **vende optimización**. Origen de la cifra de fase 1; HTTP 429 el 2026-09-24, no reverificada | **Divulgación** |
| **−43 %** | OrcaRouter, **vende una pasarela de enrutamiento**. Declara: «Nothing below is a measurement» | **Divulgación** |
| Titular «80 % menos» | Builder.io, **vende herramientas para agentes**. En el cuerpo el número es 11,9 % y 85 % | **Divulgación** |

**El 73-87 % no se sostiene.** Nadie publica su método. Las oficiales comparables son −11,9 %
(contra Sonnet solo) y −85 % (Haiku contra Sonnet solo, **perdiendo 29 % de puntuación**).
Cada blog elige la línea base que más le conviene, y ninguno mide.

**El experimento que separa el ahorro real del aparente.** Tres brazos sobre **el mismo banco
de tareas reales** (E-2), con puerta de verificación automática: caro solo, barato solo,
enrutado. La métrica no es el coste medio por llamada, sino **coste total del brazo ÷ tareas
superadas**. El denominador delata el ahorro aparente: **las tareas fallidas facturan sus
tokens y siguen en el numerador**, con sus reintentos detrás. Si el coste por llamada baja un
60 % y el coste por tarea completada baja un 5 %, el ahorro se lo comieron los reintentos.
Cuatro condiciones más, todas oficiales:

- **Barrer primero el esfuerzo** en el modelo actual: es el experimento más barato.
- **Precio del modelo fuerte solo, a esfuerzo `low`**: ese es «the number an advisor pairing
  has to beat». Si el multimodelo no bate la curva completa del modelo único, no se adopta.
- **Precia la cola, no la mediana:** comparar sobre el décimo más difícil. En una corrida de
  WideSearch de 20 problemas, **dos concentraron el 43 % del gasto**.
- Registrar `usage.iterations[]`, no el `usage` superior.

## Riesgos y limitaciones

**El riesgo central, formalizado.** TRIAGE lo escribe como condición aritmética: la tasa de
acierto del nivel barato **debe superar la razón de coste entre niveles** (con precios
Haiku→Opus, >20 %). Por debajo, enrutar sale más caro que no enrutar. Aviso sobre el paper:
son 5 páginas, declara «This paper presents a new idea, not yet fully proven» y **no tiene
resultados** — plantea 2.700 ejecuciones sobre SWE-bench Lite aún sin hacer. Aporta un
protocolo, no un número *(paper)*.

**Qué se degrada.** Bucles agénticos largos: Haiku «falls much further behind on long agentic
loops» *(oficial)*. Investigación con contexto grande. Y todo lo que dependa de acertar una
decisión temprana, porque el error se arrastra turnos.

**Fallos propios del asesor** *(todos oficiales)*:

- Un ejecutor a esfuerzo bajo **deja de detectar que está atascado**: pasa de consultar casi
  siempre a casi nunca, y entonces **puntúa por debajo del ejecutor solo**.
- Forzar consultas pronto tampoco arregla: un empujón en el turno 2, cuando la línea base
  consultaba en el turno 7, costó **3-4 pp**.
- Con asesor Opus 5 el consejo vuelve **cifrado** (`advisor_redacted_result`): no es
  auditable. Para leerlo hay que usar un asesor que devuelva texto plano.

**Y el riesgo que nadie menciona: la caché.** Las cachés son por modelo, así que una cascada
**pierde la reutilización de caché entre sus modelos** — y la caché es la palanca nº 1 de fase
2. Enrutar puede destruir más de lo que ahorra. La herramienta está además **en beta**.

## Cómo se integraría en MyFactory

**Esto no da una skill, y conviene decirlo.** Lo que se copia a `.claude/skills/` son
instrucciones para el modelo; el enrutamiento es **configuración del arnés** más un
experimento. No hay skill que añadir, así que **no hay tres filas que escribir** en
`SOURCES.md`, el README y `architecture.md`. Se queda como informe hasta que un piloto dé
número propio. Lo que sí encaja:

1. **Un `settings.json` de referencia comentado** en `tools/` —la carpeta de lo que se evalúa
   y no se instala—, con `CLAUDE_CODE_SUBAGENT_MODEL` y el `model:` por tipo de subagente, y
   la advertencia de que no se adopta sin medir.
2. **Una nota en cada skill que lanza subagentes:** toda skill copiada hereda el modelo de
   subagente del proyecto destino, así que el reparto de modelos es parte de su contrato.
3. **El arnés de medición sí es una herramienta.** El banco de E-2 con los tres brazos se
   construye una vez y sirve para todas las estrategias. Es la dependencia real de la fase 3.
4. **Línea base gratis:** `ciberpunk-storymaker` ya enruta estáticamente y ya tiene puertas de
   calidad automáticas. Es el piloto con menos montaje.

## Fuentes

Consultadas el **2026-09-24**.

**Oficial (Anthropic)**

- [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence) — coste por tarea, precia la cola, cifras de SWE-bench Pro y GPQA.
- [Advisor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) — configuración, pares válidos, `max_uses`, facturación, fallos del empujón.
- [The advisor strategy](https://claude.com/blog/the-advisor-strategy) — −11,9 %, +2,7 pp, BrowseComp, −85 %.
- [Model configuration · Claude Code](https://code.claude.com/docs/en/model-config) — `settings.json`, entorno, frontmatter, `opusplan`.

**Paper**

- [Triage: Routing SE Tasks to Cost-Effective LLM Tiers](https://arxiv.org/abs/2604.07494) (arXiv 2604.07494) — condición de la razón de coste. **Propuesta sin resultados.**

**Divulgación (sitios que venden algo)**

- [Morph, LLM cost optimization](https://www.morphllm.com/llm-cost-optimization) — origen del 73-87 %. **HTTP 429: no reverificada.**
- [OrcaRouter, Pricing the Advisor Loop](https://www.orcarouter.ai/blog/claude-opus-5-5-advisor-tool-playbook) — −43 %, admite que no es medición.
- [Builder.io, The Claude advisor pattern](https://www.builder.io/blog/the-claude-advisor-pattern) — titular 80 %, cuerpo 11,9 % y 85 %.
