# Fase 3 · B-2: ejecución de código con MCP

Subinforme técnico. Fecha: **2026-09-24**. Entrada: la estrategia **B-2** de
[`fase2-clasificacion.md`](fase2-clasificacion.md), clasificada como *tratar con cuidado*
(impacto 5, esfuerzo 1, riesgo 2). Manda el **coste por tarea completada con éxito**, no los tokens por llamada.

## Qué es y cómo funciona

El coste no está en la llamada. Está en dos sitios que casi nadie mira:

1. **Las definiciones de herramientas se cargan enteras y por adelantado.** El cliente MCP pide `tools/list`
   a cada servidor y mete en el prompt nombre, descripción y esquema JSON completo de *todas* las
   herramientas, se usen o no. Con decenas de servidores eso ocupa el contexto antes de empezar.
2. **Todo resultado intermedio pasa por el modelo.** El ejemplo de Anthropic: leer una transcripción de
   Google Drive y escribirla en Salesforce hace que el documento cruce el contexto **dos veces**, al leerlo
   y al reenviarlo. Con un documento largo, unos 50.000 tokens que el modelo no necesita razonar.

Al ejecutar código, en vez de invocar herramientas una a una el modelo **escribe un programa** que las
llama dentro de un entorno de ejecución. Dos consecuencias:

- **Descubrimiento por sistema de ficheros.** Cada herramienta MCP se presenta como un fichero
  (`./servers/google-drive/getDocument.ts`) que envuelve la llamada real. El modelo lista el directorio y
  lee solo las definiciones que va a usar: divulgación progresiva.
- **Los datos se quedan en el entorno.** El programa filtra y agrega ahí dentro; al contexto sube solo lo
  que el código imprime. De 10.000 filas de una hoja, el agente ve las cinco que importan. Anthropic añade
  bucles, condicionales y manejo de errores en código en vez de encadenar llamadas, y *tokenizar* datos
  personales a marcadores antes de que el modelo los vea.

En la API de Anthropic ya viene empaquetado: **programmatic tool calling** (el modelo escribe Python en el
contenedor de `code_execution` y llama desde ahí a tus herramientas) y **tool search tool** (las
definiciones se buscan, no se precargan).

## Cómo se instala y configura

Documentación, no ejecución: **no se ha instalado ni ejecutado nada**.

**Opción A — API de Anthropic, la de menos trabajo.** Está en GA. Se incluye en `tools` el bloque
`{"type": "code_execution_20260120", "name": "code_execution"}` —esa versión o posterior— y se marca cada
herramienta propia con `"allowed_callers": ["code_execution_20260120"]`; con `["direct", "code_execution_20260120"]`
valen ambas vías, aunque la documentación recomienda elegir una sola por herramienta. Las herramientas se
exponen al código como funciones **async** de Python; el bloque `tool_use` vuelve con un campo `caller` que
dice si lo pidió el modelo o el código. Haiku 4.5 acepta la versión pero **no soporta** esta función.

**Opción B — el patrón del artículo, autoalojado.** Generar el árbol `servers/<servidor>/<herramienta>.ts`
desde los esquemas MCP, montar un intérprete aislado (Cloudflare usa aislados V8 sin salida a internet) y
dar al agente acceso de lectura a ese árbol. El entorno de ejecución es tuyo; el trabajo también.

**Opción C — Claude Code, sin construir nada.** Buena parte del ahorro de la pata 1 ya viene de serie:
**tool search está activo por defecto** y difiere la carga de definiciones MCP. Queda no poner
`"alwaysLoad": true` en `.mcp.json`, que precarga todo; acotar `MAX_MCP_OUTPUT_TOKENS` (25.000 por defecto,
lo que pasa de ahí va a disco referenciado); y activar `MCP_DISCOVERY_CACHE=1` para no repedir la lista cada
sesión. El sandbox de Bash se apoya en primitivas del sistema: bubblewrap en Linux, seatbelt en macOS.

## Ahorro esperado y cómo medirlo

| Cifra | Fuente | Confianza |
|---|---|---|
| 150.000 → 2.000 tokens (**−98,7 %**), flujo Drive→Salesforce | *Code execution with MCP* | **Oficial**, pero **ejemplo ilustrativo**: sin metodología ni benchmark publicados |
| Tool search: ~77.000 → ~8.700 tokens (**−85 %**) | *Advanced tool use* | **Oficial**, sobre evaluaciones MCP |
| Tool search: acierto 49 %→74 % (Opus 4), 79,5 %→88,1 % (Opus 4.5) | ídem | **Oficial** |
| Programmatic tool calling: 43.588 → 27.297 tokens (**−37 %**); GIA 46,5 %→51,2 % | ídem | **Oficial** |
| BrowseComp y DeepSearchQA: **+11 %** de rendimiento con **−24 %** de tokens de entrada | Docs de Anthropic | **Oficial** |
| «Code mode»: −58,2 % con 96 herramientas, −92,8 % con 508 | getmaxim.ai | **Divulgación** y **no verificada**: solo apareció en el resumen del buscador |
| Toolsets dinámicos: −91/−97 % de tokens, pero **2-3× más llamadas** y **~50 % más de tiempo** | Speakeasy | **Divulgación** (vendedor) |

**Ojo con el −98,7 %:** es el mejor caso de un flujo con un documento grande cruzando el contexto dos veces.
No es una media, y **no hemos encontrado ningún paper independiente** que lo reproduzca.

**Cómo se mediría en un cliente**, nunca comparando tokens por llamada:

1. **Línea base con tareas reales:** 20-30 tareas del cliente, con criterio de éxito verificable por alguien
   que no sea el modelo, y modelo, versión de prompt y semilla fijos.
2. **Dos brazos sobre las mismas tareas:** (A) herramientas MCP directas, (B) ejecución de código.
3. **Se compara** tokens de entrada y salida **por tarea completada con éxito**, euros por tarea con éxito,
   tasa de éxito, reintentos y latencia de punta a punta.
4. **Criterio de decisión:** B gana solo si baja el coste por tarea con éxito **sin bajar la tasa de éxito**.
   Si gasta la mitad y acierta un 10 % menos, B ha perdido.
5. **Sesgo de facturación que hay que descontar:** los resultados de herramientas llamadas desde el código
   **no cuentan como tokens de entrada**. El contador baja de verdad, pero el contenedor se factura aparte
   —1.550 horas gratis al mes por organización, después 0,05 $ por hora y contenedor—.

## Riesgos y limitaciones

- **Hace falta un entorno de ejecución.** Es lo que le dio esfuerzo 1 en la fase 2: sandbox, límites de
  recursos y monitorización. Anthropic lo dice en su artículo: el beneficio se pesa contra ese coste
  operativo y de seguridad que las llamadas directas no tienen.
- **Código generado corriendo sobre datos del cliente.** El modelo escribe el programa y nadie lo revisa
  antes de ejecutarlo. Sin aislamiento, una inyección de prompt metida en un documento leído se convierte en
  ejecución arbitraria con las credenciales del agente.
- **Qué sale de la máquina del cliente.** El contenedor de `code_execution` de la API es **de Anthropic, no
  del cliente**: sin acceso a internet y aislado del anfitrión, pero los datos que el código toca **salen de
  casa**, con retención de hasta **30 días** y la función marcada **no elegible para ZDR**. Con código
  privado eso es decisión contractual, no técnica. La variante autoalojada evita el problema y nos
  transfiere entero el trabajo de aislamiento.
- **`allowed_callers` no es una frontera de seguridad.** La documentación lo advierte: guía al modelo, pero
  no bloquea a nivel de API. Hay que seguir preparado para una invocación directa.
- **Depurar cuesta más, y lo que el modelo no ve no lo razona.** Un fallo ya no es «la herramienta devolvió
  mal»: puede ser el código generado. Si el filtrado descarta el dato clave, el agente falla sin saber por qué.
- **No siempre gana.** Anthropic desaconseja tool search con menos de ~10 herramientas, y programmatic tool
  calling para llamadas simples o cuando el modelo *debe* razonar sobre los resultados intermedios. Límites
  del contenedor: 1 CPU, 5 GiB de RAM, 5 GiB de disco.

## Cómo se integraría en MyFactory

No como skill: una skill es texto que se copia a `.claude/skills/`, y esto necesita un entorno que se ejecuta.

1. **Nada en `.claude/skills/` por ahora.** Lo que sí cabe es la parte barata y sin entorno: una skill corta
   de *higiene de MCP* con las reglas de la opción C —no usar `alwaysLoad`, acotar `MAX_MCP_OUTPUT_TOKENS`,
   apagar los servidores que no se usan—. Es configuración: se copia tal cual y no arrastra infraestructura.
2. **En `tools/`, evaluado y sin instalar**, el trato que ya recibieron `ponytail` y `rtk`: si se prueba un
   generador de envoltorios MCP→código, vive ahí con su fila en `tools/SOURCES.md` —repositorio, commit,
   licencia y qué es— y medido sobre un repositorio real, no sobre el benchmark de su autor. Una herramienta
   sin su fila está copiada, no instalada.
3. **Con clientes de código privado, la opción A no se propone sin permiso escrito.** La retención de 30 días
   y la no elegibilidad para ZDR se ponen sobre la mesa antes de la demo, no después.

## Fuentes

Todas consultadas el **2026-09-24**.

- Anthropic, *Code execution with MCP* (04-11-2025) — https://www.anthropic.com/engineering/code-execution-with-mcp
- Anthropic, *Introducing advanced tool use* (24-11-2025) — https://www.anthropic.com/engineering/advanced-tool-use
- Claude Platform Docs, *Programmatic tool calling* — https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling
- Claude Platform Docs, *Code execution tool* — https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool
- Claude Code Docs, *Connect Claude Code to tools via MCP* — https://code.claude.com/docs/en/mcp
- Claude Code Docs, *Configure the sandboxed Bash tool* — https://code.claude.com/docs/en/sandboxing
- Cloudflare, *Code Mode* (26-09-2025) — https://blog.cloudflare.com/code-mode/
- Speakeasy, *Reducing MCP token usage by 100x — you don't need code mode* (18-11-2025) — https://www.speakeasy.com/blog/how-we-reduced-token-usage-by-100x-dynamic-toolsets-v2
- getmaxim.ai, *Code Execution with MCP: How Code Mode Cuts Agent Token Costs by 90%+* — https://www.getmaxim.ai/articles/code-execution-with-mcp-how-code-mode-cuts-agent-token-costs-by-90/ (cifras **no verificadas**: no se abrió la página)
