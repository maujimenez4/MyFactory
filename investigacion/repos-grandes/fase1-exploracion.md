# Fase 1 · Exploración: trabajar con Claude Code en repositorios gigantes

Investigación B, fase 1 (abrir). Fecha: **2026-09-24**.

Objetivo: reunir candidatos —funciones nativas, servidores MCP, herramientas y
estrategias— que reduzcan el gasto de tokens al trabajar sobre repositorios de millones
de líneas, para decidir después cuáles entran en MyFactory.

**Cómo se verificó.** Las fechas y las estrellas de los repositorios salen de consultar
la API de GitHub el 2026-09-24, no del resumen de un buscador. Lo que no se pudo
comprobar así se marca como **sin verificar**.

**Aviso sobre las cifras de ahorro.** Casi todas las que se citan aquí vienen del
material del propio proyecto o de artículos de divulgación, medidas en repositorios que
no son los nuestros y con tareas que nadie describe del todo. Se recogen como *lo que
afirma la fuente*, no como lo que vamos a obtener. La fase 3 tendrá que decir cómo se
miden de verdad.

---

## 1 · Funciones nativas de Claude Code

| # | Candidato | Qué es y qué problema resuelve | Fuente | Actualización |
|---|---|---|---|---|
| N-1 | **Subagentes** | Un trabajador con contexto propio que investiga o verifica y devuelve solo el resumen. El hallazgo no entra en la conversación principal: es la forma nativa de explorar un repositorio grande sin ensuciar la sesión | [docs](https://docs.anthropic.com/en/docs/claude-code/features-overview) | Doc viva |
| N-2 | **Skills con revelación progresiva** | Markdown que se carga solo cuando hace falta. El agente lee el índice y baja al detalle si lo necesita, así que el tamaño de una skill deja de estar limitado por el contexto | [Anthropic, Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | Doc viva |
| N-3 | **Plugins que se encienden y se apagan** | Empaquetan comandos, subagentes, MCP y hooks. Lo que importa aquí: **apagarlos reduce el prompt de sistema**, y en un repo grande se acumulan | [Anthropic, plugins](https://anthropic.com/news/claude-code-plugins) | Doc viva |
| N-4 | **Hooks de ciclo de vida** | Se disparan en eventos (uso de herramienta, compactación, envío de prompt). Sirven para efectos laterales —lint, registro— **sin gastar contexto** | [docs](https://docs.anthropic.com/en/docs/claude-code/features-overview) | Doc viva |
| N-5 | **Compactación automática** | Resume la conversación al acercarse al límite. Evita que una sesión larga muera, a cambio de perder detalle | [docs, costes](https://docs.anthropic.com/en/docs/claude-code/costs) | Doc viva |
| N-6 | **`CLAUDE.md` jerárquico por carpeta** | Claude lee desde el directorio actual hacia arriba: raíz con lo común, uno por paquete con lo suyo. En un monorepo evita cargar el contexto de todos los paquetes para tocar uno | [guía monorepo](https://dev.to/myougatheaxo/claude-code-in-monorepos-hierarchical-claudemd-and-package-scoped-instructions-1il9) | 2026, sin verificar |
| N-7 | **Selección de modelo por tarea** | Modelo barato para lo mecánico y caro para lo que exige juicio. Es la palanca más directa sobre el coste y no depende de ninguna herramienta | [docs, costes](https://docs.anthropic.com/en/docs/claude-code/costs) | Doc viva |

## 2 · Servidores MCP y navegación de código

| # | Candidato | Qué es y qué problema resuelve | Fuente | Actualización |
|---|---|---|---|---|
| M-1 | **Serena** | Servidor MCP con capa LSP sobre 40+ lenguajes. Da al agente `find_symbol`, `find_references` y refactorizaciones **sobre símbolos, no sobre líneas**: se lee la función que importa, no el fichero entero | [oraios/serena](https://github.com/oraios/serena) | **2026-09-24**, 29.771 ★, licencia no estándar |
| M-2 | **Claude Context (Zilliz)** | Búsqueda híbrida (BM25 + vectores) contra una base vectorial: recupera fragmentos por consulta en vez de cargar carpetas. La fuente afirma **~40 % menos tokens** a igual calidad de recuperación | [zilliztech/claude-context](https://github.com/zilliztech/claude-context) | 2026-07-14, 12.566 ★, MIT |
| M-3 | **mcp-language-server** | LSP expuesto como MCP, más pequeño y simple que Serena. Definiciones, referencias y diagnósticos sin índice propio | [isaacphi/mcp-language-server](https://github.com/isaacphi/mcp-language-server) | 2026-03-01, 1.599 ★, BSD-3 — **seis meses sin tocar** |
| M-4 | **ast-grep** | Búsqueda y reescritura **estructural** por patrones de AST. No es MCP, es un binario: sustituye barridos de `grep` que devuelven cientos de líneas irrelevantes | [ast-grep/ast-grep](https://github.com/ast-grep/ast-grep) | **2026-09-24**, 16.021 ★, MIT |
| M-5 | **github-mcp-server** | MCP oficial de GitHub: issues, PRs, código y búsqueda **sin clonar el repositorio**. Útil cuando el repo del cliente no se puede tener en local | [github/github-mcp-server](https://github.com/github/github-mcp-server) | 2026-09-22, 33.175 ★, MIT |

## 3 · Empaquetado y mapas de código

| # | Candidato | Qué es y qué problema resuelve | Fuente | Actualización |
|---|---|---|---|---|
| E-1 | **Repomix** | Empaqueta el repositorio en un fichero para IA. Lo interesante para nosotros es `--compress`, que con Tree-sitter deja **solo firmas y estructura**; y que pasa Secretlint para no incluir credenciales | [yamadashy/repomix](https://github.com/yamadashy/repomix) | **2026-09-23**, 28.480 ★, MIT |
| E-2 | **Repo map de Aider** | Mapa del repositorio con Tree-sitter: firmas y símbolos clave en vez de ficheros. Es la idea que conviene robar aunque no usemos Aider | [aider.chat/docs/repomap](https://aider.chat/docs/repomap.html) | Repo 2026-05-22, 49.151 ★, Apache-2.0 |
| E-3 | **Gitingest** | Convierte un repositorio en un texto plano para IA, ignorando artefactos de compilación | [coderamp-labs/gitingest](https://github.com/coderamp-labs/gitingest) | 2026-09-19, 15.613 ★, MIT |
| E-4 | **code2prompt** | Vuelca el repositorio a un prompt con plantillas y recuento de tokens | [mufeedvh/code2prompt](https://github.com/mufeedvh/code2prompt) | 2026-09-18, 7.695 ★, MIT |

**Cuidado con esta categoría entera.** Un volcado completo cuesta, según la comparativa
citada, **entre 50.000 y 500.000 tokens**. En un repositorio de millones de líneas eso
no cabe, así que estas herramientas solo sirven en su modo comprimido o acotadas a un
subárbol. Usadas como «mete el repo entero», empeoran justo lo que venimos a arreglar.

## 4 · Estrategias de trabajo

| # | Candidato | Qué es y qué problema resuelve | Fuente | Actualización |
|---|---|---|---|---|
| S-1 | **Empezar la sesión en el paquete, no en la raíz** | Para una tarea de un paquete, abrir Claude dentro de él: la jerarquía de `CLAUDE.md` carga solo lo suyo | [guía monorepo](https://www.lowcode.agency/blog/claude-code-monorepo) | 2026, sin verificar |
| S-2 | **`CLAUDE.md` corto y repartido** | La recomendación que circula es **menos de 10.000 palabras** en el raíz, y bajar lo específico a cada paquete. El raíz se paga en **cada** sesión | [ídem](https://sidsaladi.substack.com/p/the-ideal-project-structure-for-claude) | 2026, sin verificar |
| S-3 | **Ficheros de exclusión** | Sacar del alcance `dist/`, `vendor/`, ficheros generados y datos. Barato y reduce el ruido de toda búsqueda | [guía monorepo](https://thepromptshelf.dev/blog/claude-code-monorepo-setup/) | 2026, sin verificar |
| S-4 | **Exploración por capas** | Arquitectura → módulo → fichero, en pasos separados, cada uno acotado. Evita el barrido que lee cincuenta ficheros para tocar uno | [Anthropic, patrones avanzados](https://www.anthropic.com/webinars/claude-code-advanced-patterns) | Webinar, 2026 |
| S-5 | **Documentación de arquitectura escrita para la IA** | Un mapa mantenido a mano —qué módulo hace qué y dónde está— que el agente lee en vez de deducirlo. Es lo que ya hacemos en `ciberpunk-storymaker` con `docs/architecture.md` | [Anthropic, patrones avanzados](https://resources.anthropic.com/hubfs/Claude%20Code%20Advanced%20Patterns_%20Subagents,%20MCP,%20and%20Scaling%20to%20Real%20Codebases.pdf) | PDF, 2026 |
| S-6 | **Tareas acotadas por módulo** | Una tarea, un módulo. Cruzar tres paquetes en una sola sesión multiplica el contexto y el riesgo de deriva | [ídem](https://www.anthropic.com/webinars/claude-code-advanced-patterns) | Webinar, 2026 |

## 5 · Caché y reutilización entre sesiones

| # | Candidato | Qué es y qué problema resuelve | Fuente | Actualización |
|---|---|---|---|---|
| C-1 | **Prompt caching** | Reutiliza el prefijo ya enviado. Escribir en caché cuesta **+25 %**; leer, **el 10 %** del precio de entrada. Claude Code lo aplica solo | [docs, prompt caching](https://docs.claude.com/en/docs/build-with-claude/prompt-caching) | Doc viva |
| C-2 | **Ordenar el prompt para que la caché sirva** | Lo estable primero —instrucciones, mapa del repo— y lo variable al final. Si lo que cambia va arriba, **la caché no acierta nunca** | [ídem](https://docs.claude.com/en/docs/build-with-claude/prompt-caching) | Doc viva |
| C-3 | **Artefactos entre sesiones** | Guardar en fichero el mapa del módulo o el resumen de una investigación, y leerlo en la sesión siguiente en vez de repetir la exploración. Es lo que hace la bitácora que ya usamos | Práctica propia | — |

---

## Lo que esta fase no ha hecho

- **No se ha medido nada.** Ninguna cifra de ahorro de aquí está comprobada por nosotros.
- **No se ha instalado nada**, como pedía el encargo.
- **No se han revisado las licencias en detalle.** Una destaca ya: Serena figura como
  `NOASSERTION`, es decir, GitHub no reconoce una licencia estándar. Para usarla con
  código de un cliente eso hay que mirarlo antes que cualquier otra cosa.
- **Falta el ángulo de seguridad de los MCP con base vectorial**: dónde se guarda el
  índice y si el código del cliente sale de su máquina. Es la pregunta que decide si
  Claude Context es viable, y va a la fase 2.
