# Fase 4 · Informe final: bajar el consumo de tokens en una solución agéntica

Investigación A, fase 4 (cerrar). Fecha: **2026-09-24**.

Consolidado leyendo **solo los tres subinformes de la fase 3**, como pide el método:
[ejecución de código con MCP](fase3-ejecucion-codigo-mcp.md), [enrutamiento de
modelos](fase3-enrutamiento-modelos.md) y [línea base de medición](fase3-linea-base-medicion.md).

## Resumen ejecutivo

1. **Las tres cifras que abrieron esta investigación no sobreviven a mirarlas de cerca.** El
   −98,7 % de la ejecución de código es un ejemplo ilustrativo sin metodología publicada; el
   73-87 % del enrutamiento sale de quien vende optimización y no se pudo ni reverificar.
2. Las cifras **oficiales y con banco de pruebas detrás son mucho menores, y mejores**:
   −85 % con *tool search* y −37 % con *programmatic tool calling*, ambas **con el acierto
   subiendo**, no bajando.
3. **El dato que más cambia el plan:** por tarea resuelta, el modelo caro sale **más barato**
   —Opus 5.5 a esfuerzo medio, 92,8 % de SWE-bench Pro a $0,22 por tarea, frente a $1,19 de
   Fable 5.1—. Antes de montar nada multimodelo hay que batir esa cifra.
4. **Lo barato ya está puesto:** en Claude Code *tool search* viene activo por defecto, así
   que buena parte del ahorro de contexto se consigue **configurando**, sin entorno de ejecución.
5. **Dos palancas que creíamos independientes se pisan:** el enrutamiento rompe la caché,
   porque las cachés son por modelo. Puede destruir más de lo que ahorra.
6. **Nada de esto se puede afirmar sin medir**, y medir cuesta más de lo que parece: detectar
   diez puntos de diferencia pide ~293 tareas por rama sin emparejar.

## Kit recomendado para MyFactory

| Pieza | Dónde va | Qué es |
|---|---|---|
| **`medir-coste-agente`** | `.claude/skills/` | La skill que falta y la que más rinde. Lleva los campos mínimos del registro, los tamaños de muestra y **la regla de no concluir con intervalos solapados** — porque el fallo típico no es recoger mal los datos, es concluir de una sola corrida |
| **Dos plantillas**, dentro de esa skill | `.claude/skills/medir-coste-agente/` | El fichero del banco de tareas y el esquema del registro por ejecución. **Vacías, sin datos de cliente** |
| **`higiene-mcp`** | `.claude/skills/` | Skill corta con lo barato y sin infraestructura: no usar `alwaysLoad`, acotar `MAX_MCP_OUTPUT_TOKENS`, activar `MCP_DISCOVERY_CACHE`, apagar los servidores que no se usan |
| **`settings.json` de referencia** | `tools/` | Comentado, con el reparto de modelos por subagente y el aviso de que no se adopta sin medir. En `tools/` porque se evalúa, no se instala |
| **Los tres subinformes y este** | `investigacion/agentica-tokens/` | El procedimiento que se relee cuando un resultado no cuadra |

**Lo que deliberadamente no entra:** ningún generador de envoltorios MCP→código, ninguna
pasarela de enrutamiento. Son infraestructura, no texto que se copia, y ninguno tiene todavía
una medición nuestra que lo justifique.

**Nota que afecta a todas las skills que ya tenemos:** una skill copiada **hereda el modelo de
subagente del proyecto destino**. El reparto de modelos es parte de su contrato, aunque no lo
diga.

## Guía de trabajo, en siete pasos

1. **Mide antes de tocar nada.** Banco de 30-60 tareas reales con commit fijo y criterio de
   éxito escrito **antes** de la primera corrida. Un tercio reservado, que no se mira hasta el final.
2. **Registra una fila por intento, no por tarea.** Los reintentos son el coste que las medias
   esconden. Separa los cuatro tipos de token: entrada, salida, lectura y escritura de caché.
3. **Barre primero el esfuerzo del modelo que ya usas.** Es el experimento más barato y a
   veces agota el problema.
4. **Aplica la higiene de MCP.** Definiciones que no se precargan, salidas acotadas, servidores
   apagados. Coste cero y sin entorno nuevo.
5. **Ordena el prompt para que la caché acierte:** lo estable arriba, lo variable abajo. Y
   antes de romper la caché con cualquier otra cosa, mide lo que la caché te está dando.
6. **Un cambio cada vez, y vuelve a medir.** Las estrategias de contexto se pisan entre sí;
   apilarlas hace imposible saber cuál funcionó.
7. **Decide por coste ÷ tareas superadas.** Si el coste por llamada baja un 60 % y el coste por
   tarea completada baja un 5 %, se lo comieron los reintentos.

## Plan de incorporación por fases

**Fase I — medir (primero, y sin excepción).** Construir la skill `medir-coste-agente` con sus
plantillas y montar el banco. `ciberpunk-storymaker` es el piloto más barato: ya enruta
estáticamente y ya tiene puertas de calidad automáticas, así que el criterio de éxito existe.
Salida: nuestra primera cifra propia.

**Fase II — lo gratis.** Higiene de MCP y orden del prompt para la caché. Sin entorno nuevo,
sin contratos. Se mide contra la fase I.

**Fase III — el esfuerzo del modelo actual.** Barrido de niveles de esfuerzo sobre el banco.
Antes de cualquier multimodelo, hay que conocer la curva completa del modelo único.

**Fase IV — los dos candidatos grandes, y solo si la fase III no agotó el problema.**
Ejecución de código con MCP y enrutamiento, cada uno con tres brazos sobre las mismas tareas.
El listón del enrutamiento está fijado: **batir al modelo fuerte solo a esfuerzo bajo**.

## Riesgos y cómo mitigarlos

| Riesgo | Mitigación |
|---|---|
| **Adoptar un porcentaje ajeno.** Cada blog elige la línea base que le conviene: uno titula «80 %» y en su propio cuerpo el número es 11,9 % | Ninguna cifra entra en una propuesta sin decir contra qué compara. Las tres etiquetas —oficial, paper, divulgación— viajan con el número |
| **Ahorro aparente.** Bajar tokens y subir fracasos encarece la tarea: con 100 € y 80 éxitos son 1,25 €/éxito, y un recorte del 30 % deja de compensar en cuanto el éxito baja del 56 % | Coste **÷ tareas superadas**, siempre. Nunca el numerador solo |
| **Concluir de una corrida.** El éxito de una tarea es una moneda, no una medida | Mínimo tres semillas, ramas emparejadas sobre las mismas tareas e intervalos por *bootstrap*. Si se solapan, no hay diferencia |
| **Código del cliente fuera de su casa.** El contenedor de `code_execution` es de Anthropic, con retención de hasta 30 días y **no elegible para ZDR** | Se pone por escrito **antes** de la demo, no después. Alternativa: autoalojar y asumir el aislamiento |
| **Un banco que contiene código del cliente** | Vive en su infraestructura con su control de acceso. A MyFactory suben método, esquema y tareas sintéticas — nunca el banco |
| **Trazas con prosa y código.** Las métricas son contadores, pero los eventos y trazas sí llevan texto | Revisar la exportación antes de activarla. Con código de cliente, observabilidad autoalojada y enmascarado en cliente |
| **Enrutar rompe la caché** — son por modelo, y la caché es la palanca mayor | Medir la caché **antes** de enrutar, y contar el gasto del asesor, que no aparece en el `usage` de nivel superior |
| **Código generado ejecutándose sin revisión** sobre datos del cliente | Aislamiento real. `allowed_callers` **no es una frontera de seguridad**: lo dice la propia documentación |
| **El banco envejece** con el modelo, el repositorio y el CLI | Se vuelve a medir; no se reutiliza un número de hace seis meses |

## Lo que sigue sin saberse

- **Coste por PR aceptada: no existe fuente pública.** Lo más cercano es el coste por
  desarrollador y día de Anthropic (~13 USD/día activo), que no sirve como línea base porque no
  tiene denominador de tareas. Queda cerrado **declarándolo**: se aproxima con tests y se
  muestrea la aceptación real sobre un subconjunto.
- **El −98,7 % no lo ha reproducido nadie** de forma independiente.
- **TRIAGE aporta la condición aritmética del enrutamiento** —el acierto del nivel barato debe
  superar la razón de coste entre niveles— pero **no tiene resultados**: plantea 2.700
  ejecuciones aún sin hacer. Es un protocolo, no un número.
