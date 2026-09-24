# Fase 3 · La línea base de medición

Investigación A, fase 3. Fecha: **2026-09-24**. Entrada: E-1 y E-2 de [`fase2-clasificacion.md`](fase2-clasificacion.md).

Esta pieza no ahorra tokens. Es **la condición para poder afirmar que alguna de las otras ahorra**. Las fases 1 y 2 están llenas de porcentajes ajenos —40 %, 73-87 %, 98 %— medidos en sistemas que no son el nuestro. Ninguno se traslada sin medición propia.

## Qué es y cómo funciona

Tres piezas. Faltando una, no es una línea base.

**1 · Banco de tareas representativas.** De 30 a 60 tareas sacadas de tickets reales ya cerrados, no inventadas. Cada una: enunciado, estado inicial del repositorio (commit fijo) y qué cuenta como terminada. Se versiona y se congela: si el banco cambia entre el antes y el después, no hay comparación. Mezclar tamaños, porque el gasto vive en las tareas largas. Kapoor et al. avisan de que un banco sin conjunto reservado se convierte en el objetivo y se le hace *overfitting* sin querer (*paper*); se aparta un tercio y no se mira hasta el final.

**2 · Criterio de éxito por tarea, escrito antes de la primera corrida.** Tres formas, de barata a cara: comprobación por código (tests, salida esperada), juez con modelo, revisión humana. Anthropic recomienda código siempre que la tarea lo permita, por rápido y fiable, y modelo cuando no (*oficial*). Para código, el criterio que manda —PR aceptada— solo lo da un humano, y eso limita cuántas corridas caben: se usan tests como aproximación barata y se muestrea la aceptación real sobre un subconjunto. Es el hueco que dejaba la fase 2, cerrado declarándolo.

**3 · Registro por ejecución**, una fila por intento y no por tarea. Los reintentos son coste.

La métrica es **coste por tarea completada con éxito** = coste de *todos* los intentos ÷ número de éxitos; y en código, **coste por historia o PR aceptada**. Kapoor et al. lo formulan como frontera de Pareto coste-exactitud y muestran que agentes complejos y caros no ganan a líneas base simples cuando se grafican las dos juntas (*paper*).

## Cómo se monta

Sin instalar ni ejecutar nada; esto es el diseño.

1. Elegir las tareas y escribir su criterio de éxito en el propio fichero del banco.
2. Fijar el entorno: commit, versión del CLI, identificador exacto de modelo, nivel de esfuerzo. Cambiar cualquiera invalida la comparación.
3. Activar instrumentación. Claude Code exporta por OpenTelemetry `claude_code.token.usage` —atributo `type` en `input`, `output`, `cacheRead`, `cacheCreation`— y `claude_code.cost.usage` en USD, ambos con `model`, `session.id`, `query_source` (`main`/`subagent`/`auxiliary`), `agent.name` y `skill.name` (*oficial*). El bloque Session de `/usage` da lo mismo por sesión más el resumen de caché (*oficial*). Para un agente propio, Langfuse registra unidades y coste por tipo, y los valores ingeridos mandan sobre los inferidos (*oficial*).
4. Volcar a tabla propia. Aunque haya panel, la conclusión se saca de un CSV o una SQLite que se guarda junto al informe.

**Campos mínimos por ejecución:**

| Campo | Por qué |
|---|---|
| `id_ejecucion`, `id_tarea`, `version_banco` | Sin esto no se repite nada |
| `variante` (A/B), `semilla` | Qué rama del experimento es |
| `tokens_entrada`, `tokens_salida` | Precios distintos |
| `tokens_cache_lectura`, `tokens_cache_escritura` | Precios distintos otra vez; sin separarlos, la caché no se puede evaluar |
| `n_llamadas`, `n_reintentos` | El reintento es el coste que las medias esconden |
| `resultado` (éxito/fallo/parcial), `juez` (código/modelo/humano) | La métrica es un cociente: sin denominador no hay nada |
| `modelo`, `version_cli`, `esfuerzo` | Cambian el resultado y no se recuerdan a los tres meses |
| `coste_usd`, `tarifa_usada` | Ver el aviso de abajo |
| `duracion_s`, `intervencion_humana_min` | El tiempo de persona suele costar más que los tokens |

**Aviso sobre el coste.** Claude Code calcula sus cifras a **tarifa pública** salvo que un administrador ponga `modelPricing` en ajustes gestionados; con tarifas contratadas, lo que muestra no cuadra con la factura (*oficial*). Es estimación, no extracto.

## Qué permite afirmar, y qué no

**Una corrida antes y otra después no demuestran nada.** El éxito de una tarea es una moneda, no una medida: el mismo modelo con el mismo prompt falla unas veces y acierta otras. Reportar una sola pasada (pass@1) es práctica común y engañosa por la varianza entre corridas independientes sobre la misma tarea (*divulgación*).

Cuánto hace falta, con la fórmula estándar de dos proporciones (α = 0,05 bilateral, potencia 0,80), sin emparejar:

| Diferencia a detectar | Tareas por rama |
|---|---|
| 80 % vs 70 % de éxito | ≈ 293 |
| 80 % vs 60 % de éxito | ≈ 82 |

Con 40 tareas no se detecta una mejora de 10 puntos. Se hace mucho mejor **emparejando**: mismas tareas en las dos ramas, comparadas tarea a tarea. Miller (Anthropic) recomienda exactamente eso —diferencias emparejadas, análisis de potencia para fijar el número de preguntas, y varias respuestas por pregunta para bajar el ruido— (*paper*). La guía práctica de repetición: mínimo tres semillas por configuración e intervalos por *bootstrap* en cada tabla; si los intervalos se solapan, no hay diferencia, por grande que parezca el punto (*divulgación*).

**El coste no es una proporción**, es continuo y con cola larga: unas pocas tareas atascadas dominan la media. Se reportan mediana e intervalo por *bootstrap*, no solo la media.

**Y las dos se mueven juntas.** Aritmética con el banco delante: 100 tareas, 80 éxitos, 100 € → 1,25 €/éxito. Una estrategia que recorta el 30 % de tokens deja el gasto en 70 €, y deja de compensar en cuanto el éxito cae por debajo de **56 %**, porque 70/56 = 1,25. Un ahorro del 30 % se evapora entero con 24 puntos menos de éxito. Por eso el numerador solo no vale.

**Sí permite afirmar:** el gasto actual por tarea y su dispersión, dónde se concentra, y si una diferencia sobrevive a su intervalo de confianza. **No permite afirmar:** que el porcentaje se traslade a otro repositorio, a otro equipo o al mes siguiente.

## Riesgos y limitaciones

- **Seguridad, lo primero.** Un banco hecho con código real de un cliente **contiene el código del cliente**, aunque esté troceado. Vive donde vive ese código, con el mismo control de acceso, y no se copia a MyFactory: allí solo entran el método, el esquema de campos y tareas sintéticas.
- **Qué sale hacia la observabilidad.** Las métricas de OpenTelemetry son contadores: llevan atributos, no texto. Los eventos y las trazas sí pueden llevar prompts y fragmentos de código, así que hay que revisar la configuración antes de exportar. Langfuse ofrece enmascarado en cliente (redacta antes de enviar, el dato no sale de la aplicación), enmascarado en la ingesta para instalaciones propias, y ventanas de retención (*oficial*). Con código de cliente, lo prudente es Langfuse autoalojado.
- **Ninguna clave en el banco.** El estado inicial se fija con un commit, nunca con un fichero de entorno copiado.
- **El banco envejece.** Cambian modelo, repositorio y CLI: una línea base de hace seis meses no es comparable. Se vuelve a medir, no se reutiliza el número.
- **La caché contamina el orden.** Una tarea que sigue a otra parecida encuentra caché caliente y sale más barata. Aleatorizar el orden o arrancar siempre en frío, y decir cuál de las dos.
- **Tests verdes ≠ PR aceptada.** El criterio barato mide otra cosa; se usa, pero se declara.
- **Medir cuesta.** Trescientas ejecuciones no son gratis: se presupuestan, y por eso se emparejan ramas y se reutiliza el banco.
- **No encontrado:** ninguna fuente pública con cifras de coste por PR aceptada. Anthropic publica coste por persona y día —unos 13 USD por desarrollador y día activo, 150-250 USD al mes, bajo 30 USD/día en el 90 % de los casos (*oficial*)—, pero eso es referencia externa, no línea base: no tiene denominador de tareas.

## Cómo se integraría en MyFactory

Las tres cosas a la vez, y conviene no confundirlas:

1. **Una skill**, `medir-coste-agente`. Es lo que debe existir, porque el fallo típico no es recoger mal los datos: es concluir de una corrida. Se dispara al pedir medir, comparar antes y después o justificar un ahorro; lleva los campos mínimos, los tamaños de muestra de arriba y la regla de no concluir con intervalos solapados.
2. **Dos plantillas dentro de la skill**: el fichero del banco de tareas (enunciado, commit, criterio de éxito) y el esquema del registro por ejecución. Vacías, sin datos de cliente.
3. **Un procedimiento**, este documento, enlazado desde la skill. El razonamiento estadístico no cabe dentro sin inflarla, y es lo que hay que releer cuando el resultado no cuadra.

Al instalarla se copia a `.claude/skills/` y se anota su procedencia donde el proyecto lleve el registro de skills. Una skill sin esa anotación no está instalada: está copiada.

## Fuentes

Todas consultadas el **2026-09-24**.

- *Manage costs effectively*, Claude Code — `/usage`, `modelPricing`, coste por desarrollador. *Oficial*. https://code.claude.com/docs/en/costs
- *Monitoring usage* (OpenTelemetry) — `claude_code.token.usage`, `claude_code.cost.usage` y atributos. *Oficial*. https://code.claude.com/docs/en/monitoring-usage
- *Define success criteria and build evaluations*. *Oficial*. https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- *Building evals* (cookbook) — graduación por código, modelo y humano. *Oficial*. https://github.com/anthropics/anthropic-cookbook/blob/main/misc/building_evals.ipynb
- E. Miller, *Adding Error Bars to Evals*, arXiv:2411.00640 — diferencias emparejadas, análisis de potencia. *Paper*. https://arxiv.org/abs/2411.00640 · https://www.anthropic.com/research/statistical-approach-to-model-evals
- S. Kapoor et al., *AI Agents That Matter*, arXiv:2407.01502 (TMLR) — coste controlado, Pareto, conjuntos reservados. *Paper*. https://arxiv.org/abs/2407.01502
- MLflow, *Benchmarking AI Agent Performance: A Practical Protocol* — tres semillas mínimo, *bootstrap*, coste por tarea de primera clase. *Divulgación*. https://mlflow.org/articles/benchmarking-ai-agent-performance/
- Langfuse, *Token & Cost Tracking*. *Oficial*. https://langfuse.com/docs/observability/features/token-and-cost-tracking
- Langfuse, *Masking* y *Data Masking (self-hosted)*. *Oficial*. https://langfuse.com/docs/observability/features/masking · https://langfuse.com/self-hosting/security/data-masking
