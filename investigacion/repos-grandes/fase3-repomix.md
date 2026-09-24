# Fase 3 · Repomix en modo `--compress`

Investigación B, fase 3. Fecha: **2026-09-24**. Candidato **E-1**, 18/25 en
[`fase2-clasificacion.md`](fase2-clasificacion.md): *probar en piloto, solo en subárbol*.
**No se ha instalado ni ejecutado nada.** GitHub verificado contra la API el 2026-09-24:
`yamadashy/repomix`, push 2026-09-23, **28.480 ★**, **MIT**. *(oficial)*

La advertencia de fase 2 sigue en pie: un volcado completo cuesta **50.000-500.000 tokens**,
así que en millones de líneas Repomix solo existe comprimido y acotado.

## Qué es y cómo funciona

Repomix empaqueta ficheros en **un solo documento** (XML por defecto; también `markdown`, `json`,
`plain`) con resumen, árbol de directorios y el contenido de cada fichero. *(oficial)*

`--compress` parsea cada fichero con **Tree-sitter** y deja solo el esqueleto. *(oficial)*

| Conserva | Tira |
| --- | --- |
| Firmas de funciones y métodos | Cuerpos de función y método |
| Clases, propiedades, estructuras | Bucles y condicionales por dentro |
| Interfaces y definiciones de tipo | Variables locales y asignaciones |
| `import` / `export` | Detalle de implementación |

Lo eliminado se marca con `⋮----`, así que se ve **dónde** falta código. Los comentarios se
conservan salvo que se pida `removeComments`. El resultado es un **mapa de superficie**: qué
existe, cómo se llama y qué firma tiene. No dice qué hace por dentro ni quién llama a quién.

**Es experimental por declaración propia**, y depende de que exista parser: JS/TS, Python, Rust,
Go, C/C++, C#, Ruby, Java, PHP y Swift. *(oficial)* Fuera de esa lista Repomix empaqueta pero
**no comprime**: el fichero entra entero, que es justo lo que queríamos evitar.

## Cómo se instala y configura

1. **Sin instalar:** `npx repomix@latest`. También `npm i -g repomix`, `brew install repomix` o
   imagen Docker. *(oficial)* En repo de cliente, `npx` con versión fijada no deja un global.
2. **Acotar a un subárbol** — lo único que lo hace viable aquí. Dos vías:
   `npx repomix@latest src/modulo --compress` (ruta posicional) o
   `--include "src/backend/app/features/**/*.py"` con globs separados por coma. *(oficial)*
3. **Ignorar:** `-i, --ignore "*.test.ts,docs/**"`; respeta `.gitignore` y `.repomixignore` salvo
   `--no-gitignore` / `--no-default-patterns`. *(oficial)*
4. **Salida y ruido:** `--style markdown`, `-o mapa.md`, `--no-file-summary`,
   `--remove-empty-lines`, `--top-files-len`. *(oficial)*
5. **Medir antes de gastar:** `--token-count-tree` da el reparto de tokens por carpeta y fichero;
   `--token-count-encoding o200k_base|cl100k_base` elige tokenizador. *(oficial)*
6. **Fichero de proyecto** `repomix.config.json` con `output.compress`, `include`,
   `ignore.customPatterns`, `security.enableSecurityCheck`, `tokenCount.encoding`. La línea de
   órdenes manda sobre el fichero. *(oficial)*

**Hay MCP (`repomix --mcp`)** con `pack_codebase`, `pack_remote_repository`,
`read_repomix_output` y `grep_repomix_output`. *(oficial)* **No lo recomiendo**, por lo mismo que
con ast-grep: mete su descripción de herramientas en toda sesión. La línea de órdenes por `Bash`
hace lo mismo y solo cuando se pide.

## Ahorro de tokens esperado y cómo medirlo

| Cifra | Origen | Confianza |
| --- | --- | --- |
| **«~70 % menos tokens»** con `--compress` | README y web de Repomix | *oficial, **autodeclarada**: sin metodología ni corpus publicados* |
| 48-70 % sobre 4 repos reales (ky, hono, tailwindcss, gin), medido 2026-09-12 | `Sitrozyi/repomix-semantic-compressor` | *divulgación, y **no mide `--compress`*** |
| Volcado completo: 50.000-500.000 tokens | nuestra fase 2 | *nuestro* |

**La segunda fila engaña y conviene decirlo.** Esa tabla compara *salida cruda de Repomix* contra
*un post-procesador de terceros*, no contra `repomix --compress`; además cuenta tokens por
aproximación de bytes (`bytes / 3,8`) salvo con `--exact-tokens`. **No es evidencia del ahorro de
`--compress`.** Y el contador propio de Repomix usa tokenizadores de OpenAI: para Claude, todas
sus cifras son estimación.

Aritmética nuestra, **no medida**: si un módulo de cliente vuelca 200.000 tokens, un 70 % lo deja
en **~60.000**. Sigue sin caber cómodamente en una sesión de trabajo, y se paga **entero y por
adelantado**, se use o no.

**El experimento que decide** —mapa precargado contra el agente explorando solo—:

1. **Tres tareas reales** por módulo: una de localización («dónde se valida X»), una de
   arquitectura («qué capas hay y cómo se hablan») y una de cambio («añade un endpoint como Y»).
2. **Rama A:** sesión limpia, solo `Glob`/`Grep`/`Read`. **Rama B:** la misma tarea con el mapa
   comprimido ya en el contexto. **Rama C** (la interesante): el mapa **en disco**, no en el
   contexto, y el agente lo consulta con `Grep`.
3. **Métrica:** tokens de entrada **acumulados** de la sesión, número de turnos y si la respuesta
   fue correcta. El coste del mapa se imputa entero a B desde el turno 1.
4. **Umbral:** B solo gana si su total baja de A **en las tres tareas**. Si gana en la de
   arquitectura y pierde en las otras dos, la conclusión no es «usar Repomix», es «usarlo solo
   para preguntas globales».

**Mi predicción, y es desfavorable.** `Glob`/`Grep`/`Read` son **perezosos**: pagan por lo que se
lee. Una tarea de localización bien acotada gasta del orden de **3.000-15.000 tokens** de lectura.
Un mapa comprimido de un módulo grande cuesta **decenas de miles** antes de la primera pregunta, y
la mayor parte no se usará nunca. **Precargar el mapa pierde casi siempre**; la rama C —el mapa
como fichero grepable, jamás volcado— es la única que puede ganar, y lo que aporta frente a `Grep`
sobre el código fuente es densidad: aciertos de firma sin cuerpos alrededor.

Donde Repomix gana sin discusión es **donde no hay sistema de ficheros**: pegar contexto en un
chat, o mandar un módulo a un modelo sin herramientas. Ese no es nuestro caso.

## Riesgos y limitaciones, incluida la seguridad del código del cliente

- **Secretlint reduce el riesgo; no lo elimina.** Activo por defecto, detecta patrones *conocidos*
  —AWS, GCP, Azure, GitHub, GitLab, Slack, Stripe, OpenAI, Anthropic, claves privadas, cadenas de
  conexión— y excluye del volcado el fichero sospechoso. *(oficial)* Es **casamiento de patrones**:
  un token interno del cliente con formato propio, una contraseña en un `.properties` raro o un
  secreto partido en dos constantes **no casan con nada y pasan**. No se publica tasa de falsos
  negativos, así que no hay cifra que citar.
- **`--compress` ayuda de refilón, y no hay que fiarse.** Al tirar los cuerpos de función caen los
  literales que viven dentro; pero las constantes de módulo y los valores por defecto de los
  parámetros **son firma y sobreviven**. *(razonamiento nuestro, no medido.)*
- **El volcado queda en disco.** `repomix-output.xml` es una copia en claro del código del
  cliente. Si acaba en un commit, en un backup o en un adjunto, el incidente ya ocurrió. Va a
  `.gitignore` y se borra al terminar.
- **Lo que entra al contexto va al proveedor.** Un volcado es una **copia a granel** del módulo;
  la exploración manda solo lo leído. Repomix **aumenta** la superficie expuesta, y con
  trazabilidad activada (Langfuse o equivalente) ese volcado queda además en la traza, que es un
  tercero más. Con un cliente que tenga cláusula de confidencialidad, esto se pregunta por escrito
  antes, no después.
- **`--no-security-check` existe.** Basta un agente con prisa. Si se adopta, el flag se prohíbe en
  la guía y se fija `enableSecurityCheck: true` en el config del repo.
- **Cobertura de compresión desigual** (ver arriba) y **estado experimental**: la salida puede
  cambiar entre versiones, así que la versión se fija.
- **El mapa envejece.** Un volcado es una foto: al tercer commit miente, y un agente que se fía de
  un mapa viejo afirma cosas falsas con seguridad.

## Cómo se integraría en MyFactory

**Dos entradas, como con ast-grep, y no se mezclan:**

| Qué | Dónde | Por qué |
| --- | --- | --- |
| El paquete npm `repomix` | **`tools/`**, evaluado y **no copiado** | Un paquete npm **no es una skill** |
| El skill oficial `repomix-explorer` | `.claude/skills/`, copiable | Es documentación: enseña el flujo y qué opciones usar |

Filas de procedencia obligatorias: la del paquete en `tools/SOURCES.md` (`yamadashy/repomix`,
versión exacta, MIT, qué es), la del skill en `.claude/Skills/SOURCES.md` (`npx skills add
yamadashy/repomix --skill repomix-explorer`, o el plugin
`/plugin marketplace add yamadashy/repomix`). **Y el acoplamiento anotado:** el skill no sirve sin
el paquete.

**Qué no meter:** el servidor MCP, por su coste de contexto en toda sesión.

**La guía de uso pesa más que la herramienta.** Si se copia el skill tal cual, un agente
empaquetará repos enteros, porque es lo que el skill propone. La entrada de MyFactory debe llevar
encima la restricción nuestra: **solo `--compress`, solo con `--include` acotado, salida a fichero
y consultada con `Grep`, nunca volcada al contexto**.

## Fuentes

Todas consultadas el **2026-09-24**.

- <https://repomix.com/guide/code-compress> — qué conserva y qué tira Tree-sitter, estado experimental
- <https://repomix.com/guide/command-line-options> — `--compress`, `--include`, `-i`, `--style`, `--token-count-tree`, `--no-security-check`
- <https://repomix.com/guide/configuration> — `repomix.config.json`, `security.enableSecurityCheck`, `tokenCount.encoding`
- <https://repomix.com/guide/security> — Secretlint activo por defecto, qué detecta
- <https://repomix.com/guide/mcp-server> — `--mcp`, `pack_codebase`, `grep_repomix_output`
- <https://repomix.com/guide/repomix-explorer-skill> — skill oficial y su instalación
- <https://github.com/yamadashy/repomix> — API de GitHub: 28.480 ★, MIT, push 2026-09-23; «~70 %» autodeclarado
- <https://github.com/secretlint/secretlint> — preset recomendado, cobertura por proveedor, MIT
- <https://github.com/Sitrozyi/repomix-semantic-compressor> — tabla 48-70 % *(divulgación; mide **otra** herramienta)*
