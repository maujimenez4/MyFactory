# Fase 3 · Serena (MCP + LSP) para repositorios gigantes

Consultado el 2026-09-24. `oraios/serena`, 29.771 estrellas, último push 2026-09-24.
Confianza de cada dato: **oficial** (repo o docs del proyecto), **paper**, **divulgación** (terceros).

## Qué es y cómo funciona

Servidor MCP que corre **en local**, en la máquina donde está el código; Claude Code lo arranca por
stdio y no hay servicio remoto que contratar (oficial). Por debajo levanta un **language server**
por lenguaje, el mismo que usa un IDE. El LSP es el protocolo con el que un editor pregunta a un
analizador del lenguaje: dónde se define este símbolo, quién lo referencia, qué contiene este
fichero, qué tipo devuelve. El analizador ya parseó el proyecto, así que responde con posiciones
exactas, no con coincidencias de texto.

Serena lo expone como herramientas del agente (oficial, README): lectura (`find_symbol`,
`symbol_overview`, `find_referencing_symbols`, `find_declaration`, `find_implementations`,
`type_hierarchy`), edición simbólica (`replace_symbol_body`, `insert_after_symbol`, `rename`,
`move`, `safe_delete`) y básicas (`search_for_pattern`, `find_file`, `read_file`,
`execute_shell_command`).

Ahorra contexto por tres mecanismos distintos, que conviene no confundir. Uno: leer un símbolo trae
el cuerpo de una función; leer el fichero trae todo lo demás, y en 1.500 líneas para una función de
40 el resto es ruido que el modelo paga igual. Dos: `symbol_overview` devuelve el mapa del fichero
(nombres y firmas, sin cuerpos), así que el agente decide qué abrir **después** de ver el índice, en
vez de abrir para averiguar si le sirve. Tres: `find_referencing_symbols` responde en una llamada lo
que un `grep` responde con decenas de ficheros, falsos positivos y rondas de lectura para descartar.

En contra hay un coste fijo: las definiciones de ~21 herramientas ocupan contexto en cada sesión
(divulgación), y en repos pequeños no se amortiza. 40+ lenguajes con backend LSP libre, más un
backend por plugin de JetBrains **de pago** (oficial). La calidad no es igual en todos: que existan
guías específicas para C/C++, Scala, OCaml y GDScript indica que ahí instalar no es trivial.

## Cómo se instala y configura (NO instalado, solo documentado)

1. `uv tool install -p 3.13 serena-agent`, luego `serena init` (oficial).
2. Alta: `serena setup claude-code`, o a mano
   `claude mcp add serena -- serena start-mcp-server --context claude-code --project "$(pwd)"`
   (con `--scope user` y `--project-from-cwd` para todos los proyectos) (oficial).
3. `serena project index` antes de trabajar (oficial): sin índice, la primera consulta paga el
   arranque del language server, y en repos grandes son minutos.
4. Configuración por capas: global, CLI, `project.yml`, contexto y modos (oficial). Interesan
   `read_only: True` y la lista blanca de herramientas. Si tarda en arrancar, `MCP_TIMEOUT=60000`.
   El contexto `ide-assistant` está **deprecado** y hoy se mapea a `claude-code` (divulgación).
5. Panel web local en `http://localhost:24282/dashboard/index.html` con llamadas y logs (oficial):
   sirve para auditar qué tocó el agente.
6. Los docs avisan de que las descripciones de las herramientas nativas de Claude Code (~16k tokens)
   sesgan al modelo hacia ellas, y dan contrapeso con
   `serena prompts print-cc-system-prompt-override` más unos hooks en alfa (oficial).

## Ahorro de tokens esperado y cómo medirlo

**No hay cifra oficial medida por el proyecto.** El README solo afirma, cualitativamente, que la
edición simbólica es «mucho más eficiente en tokens que las alternativas típicas» (oficial).

Lo que circula es **divulgación** y nada reproducible tal cual: ~50,3% menos tokens *solo en
definiciones de herramientas* (fork `serena-slim`, que agrupa 29 tools en 18: mide el coste fijo, no
el ahorro por tarea); «hasta ~70%» en artículos de introducción, sin metodología; y el caso ManoMano
(«Project Aegis», 36K líneas de Java), donde Serena resolvió una refactorización con 4 subagentes
frente a «docenas» sin él, leyendo >69M tokens apoyados en caché — **no pude abrir el artículo (HTTP
403)**, va por fragmentos de buscador, así que es anécdota y no medida.

Cómo se mediría de verdad antes de prometer nada a un cliente: banco de 15-20 tareas reales de su
repo (localizar, explicar, cambiar, encontrar usos); A/B con el mismo modelo, con y sin Serena,
repetido por la varianza; y por tarea, tokens de entrada, salida y lectura de caché, coste, tiempo,
número de llamadas y **si la tarea salió bien**, juzgado por una persona. Datos de `/cost`, de la
exportación OTEL de Claude Code y del panel de Serena. El coste fijo de las definiciones se cuenta
aparte: con tareas pocas y cortas se come el ahorro. Y un ahorro de tokens con peor tasa de acierto
no es un ahorro: hay un experimento público que abandonó su pila de optimización por efectos
negativos en corrección (divulgación).

## Riesgos y limitaciones, incluida la seguridad del código del cliente

**Licencia: el punto que decide.** No soy asesor legal; esto es análisis técnico y debe validarlo
quien corresponda antes de firmar.

- Hoy (oficial, fichero `LICENSE`): SolidLSP (`src/solidlsp`) es **MIT**, el resto de la aplicación
  **GPL-3.0-or-later**; una distribución que combine ambos queda entera bajo GPL.
- No es retroactivo: **hasta la v1.7.0 incluida** (commit `74c38a65…`) sigue disponible bajo MIT; el
  copyleft entra con la v2, fusionada el 2026-09-15 (oficial).
- **Usarla como herramienta externa no contamina nada.** Corre como proceso separado y habla por MCP
  (JSON-RPC): no enlazamos su código con el del cliente ni con el nuestro. La GPL regula la
  **distribución** de obra derivada, no el uso, y la salida de un programa GPL no queda cubierta solo
  por haber pasado por él. **El código del cliente no adquiere ninguna obligación** por haber sido
  leído, indexado o editado con Serena; nuestro trabajo y nuestros informes, tampoco.
- **Dónde sí obliga.** (a) Si la **redistribuimos**: copiada en un repo que entregamos, o dentro de
  una imagen Docker o un instalador que dejamos al cliente. Entonces hay que entregar licencia,
  avisos de copyright y el código fuente correspondiente, incluidas nuestras modificaciones. (b) Si
  la **forkeamos** o le escribimos un plugin que carga en su proceso: obra derivada, sale bajo GPL.
  (c) Exponerla como servicio interno por red no es distribución en GPLv3 (no es AGPL); en cuanto el
  binario llega a la máquina del cliente, sí lo es.
- **No hay licencia comercial alternativa publicada.** El modelo es *open core*: lo de pago es el
  plugin de JetBrains, no una excepción a la GPL.
- **La vía MIT existe, pero es media trampa.** Fijar la v1.7.0 evita la GPL y congela el producto:
  sin parches ni lenguajes nuevos. Y la 1.7.0 es justo la versión que arregló el fallo de abajo.
- **Qué cerrar por escrito antes de ofrecerla:** permiso para instalar software GPL de terceros en su
  entorno; constancia de que no la incorporamos al entregable (o, si se instala en su máquina, quién
  es el licenciatario y cómo se le entregan licencia y fuente); versión y commit fijados con política
  de actualización de seguridad; destino del directorio `.serena` al cerrar el proyecto; y, aparte,
  el permiso explícito de que fragmentos de su código viajan al proveedor del modelo.

**Seguridad.** Vulnerabilidad crítica real: inyección de plantilla Jinja2 sin sandbox vía
`.serena/project.yml` de un repositorio malicioso, con ejecución al abrir el proyecto y **antes de
cualquier interacción**. Afecta a ≤1.6.1, corregido en 1.7.0 (GHSA-pp25-4cg4-qcr9, 2026-08-17; CVE
pendiente entonces). Consecuencia: **nunca abrir con Serena un repo que no sea de confianza**, y
exigir 1.7.0 como mínimo. Además, `execute_shell_command` y el REPL ejecutan con los permisos del
proceso, y `read_only` **no** protege frente al REPL: lo dicen los propios docs (oficial). Panel y
transporte HTTP escuchan en localhost por defecto; exponerlos se desaconseja (oficial).

**¿Sale el código de la máquina?** Serena no lo envía a ninguna parte: language servers e índice son
locales y no encontré documentación de telemetría — ausencia de evidencia, no evidencia de ausencia:
**queda por verificar en el código** antes de usarla con un cliente. Pero el agente **sí** manda al
proveedor del modelo cada fragmento que Serena le devuelve. Serena reduce ese volumen; no lo elimina.

**¿Qué índice y dónde?** Un `.serena/` dentro del propio repo: `project.yml`,
`cache/<lenguaje>/document_symbols_cache_*.pkl` con los símbolos y `memories/*.md` con notas del
agente (divulgación, coherente con los docs). La caché es **pickle de Python**, que no debe cargarse
de fuentes ajenas; y las *memories* son texto sobre el código del cliente que puede acabar
commiteado por descuido. A `.gitignore`, o se borra al cerrar.

**Otras limitaciones.** El agente olvida las herramientas de Serena tras compactar y hay que
recordárselo (divulgación; el proyecto lo admite al ofrecer su override). Calidad desigual por
lenguaje. Un language server sobre un monorepo de millones de líneas consume memoria y tarda en el
primer indexado: medirlo, no suponerlo.

## Cómo se integraría en MyFactory

No es una skill: es un servidor MCP con binario y dependencias. Su sitio es `tools/`, como evaluado
que **no** se copia a `.claude/skills/`.

- `tools/serena/` con esta ficha y la configuración de referencia: `project.yml` de ejemplo con
  `read_only: True` y lista blanca de herramientas, más el comando `claude mcp add`. Sin binarios.
- Fila de procedencia: `https://github.com/oraios/serena`, commit fijado en la fecha de evaluación,
  licencia **GPL-3.0-or-later** (SolidLSP, MIT), versión mínima **1.7.0** por seguridad. La fila
  lleva el aviso de que **no es permisiva**, para que nadie la copie a un entregable creyendo que
  es MIT.
- Lo que sí puede ser skill es el **uso**: una skill corta que diga cuándo preferir
  `find_symbol`/`find_referencing_symbols` a `Read`/`Grep` y lo recuerde tras compactar. Es nuestra,
  se escribe desde cero y no hereda la GPL. Antes de llevarla a un cliente: prueba de campo en repo
  propio con la medición de arriba, y el permiso por escrito del apartado de licencia.

## Fuentes

Todas consultadas el 2026-09-24.

- https://github.com/oraios/serena — README: herramientas, lenguajes, licencia.
- https://github.com/oraios/serena/blob/main/LICENSE — licencia por componentes, corte en v1.7.0.
- https://oraios.github.io/serena/02-usage/020_running.html — arranque e indexado.
- https://oraios.github.io/serena/02-usage/030_clients.html — alta en Claude Code, hooks, override.
- https://oraios.github.io/serena/02-usage/070_security.html — seguridad oficial.
- https://oraios.github.io/serena/02-usage/060_dashboard.html — panel local, puerto 24282.
- https://about.gitlab.com/blog/critical-rce-in-serena/ — SSTI/RCE, GHSA-pp25-4cg4-qcr9, fix en 1.7.0.
- https://github.com/mcpslim/serena-slim — 50,3% en definiciones de herramientas (tercero).
- https://medium.com/manomano-tech/project-aegis-benchmarking-ai-agents-and-why-serena-is-our-new-must-have-311673db35dd
  — 36K líneas de Java. **No accesible (HTTP 403)**; citado por fragmentos de buscador.
- https://github.com/shreyasht/token-optimization-stack — corrección degradada al optimizar tokens.
