# Fase 3 · ast-grep

Investigación B, fase 3. Fecha: **2026-09-24**. Candidato **M-4**, empatado a 22/25 en
[`fase2-clasificacion.md`](fase2-clasificacion.md). **No se ha instalado ni ejecutado nada.**
GitHub verificado contra la API el 2026-09-24: `ast-grep/ast-grep`, push 2026-09-24,
**16.021 ★**, **MIT**. *(oficial)*

## Qué es y cómo funciona

`grep` y `ripgrep` buscan **bytes**. ast-grep parsea con tree-sitter y busca **nodos del árbol
sintáctico**; el patrón se escribe en el propio lenguaje, con `$X` para un nodo y `$$$` para una
lista. *(oficial)* Por eso **no ve** comentarios ni cadenas, y distingue la posición sintáctica:
justo el ruido que un `grep` no puede quitar. El mismo motor reescribe, con `--rewrite`.

**Ejemplo concreto.** Regla de `ciberpunk-storymaker` §6: ningún `service.py` lanza
`HTTPException`. Con texto, `rg "HTTPException" src/backend` devuelve los `import`, el handler de
`commons/errors/`, las menciones en docstrings, los tests y los `except HTTPException`. Ninguna
es la infracción. Con AST:

```
ast-grep --lang python -p 'raise HTTPException($$$)' src/backend
```

solo los `raise` reales, con cualquier formato. Un `rg "raise HTTPException"` se acerca, pero
falla con la llamada partida en dos líneas y sigue cazando el ejemplo de un docstring.

## Cómo se instala y configura

1. **Binario:** `npm i -g @ast-grep/cli`, `cargo install ast-grep --locked`, `brew install
   ast-grep` o el binario de la release. En repo de cliente, **release fijada por versión**: menos
   cadena de dependencias que `npm -g`. *(oficial)*
2. **Uso suelto:** `ast-grep --lang <lg> -p '<patrón>' <ruta>`, con `--json` para encadenar. Sin
   configuración. *(oficial)*
3. **Uso de proyecto:** `ast-grep new` crea `sgconfig.yml` y reglas YAML; `ast-grep scan` las
   aplica, `ast-grep test` las prueba. Solo compensa si las reglas se reutilizan. *(oficial)*

**¿MCP?** Sí: `ast-grep/ast-grep-mcp`, oficial, MIT, Python, `uvx --from git+… ast-grep-server`,
con `find_code`, `find_code_by_rule`, `dump_syntax_tree` y `test_match_code_rule`. **Sus autores
lo marcan experimental.** *(oficial)* Un MCP mete su descripción de herramientas en cada sesión,
que es lo contrario del objetivo de esta investigación. **Recomendación: línea de órdenes por
`Bash`, no MCP.** Existe además un **skill oficial** (`ast-grep/agent-skill`, MIT, `npx skills add
ast-grep/agent-skill`) que solo enseña a escribir patrones y **exige el binario ya instalado**.

## Ahorro de tokens esperado y cómo medirlo

**No hay ninguna cifra publicada del ahorro de ast-grep frente al `Grep` de Claude Code.** Ni
oficial ni en paper. Lo que circula mide *otras* herramientas de índice AST: un A/B independiente
da −35 % en consultas de localización y **paridad en una mezcla realista de tareas**, con la
conclusión que importa aquí: **buscar es una fracción pequeña del gasto; lo que domina es leer el
código encontrado**. *(divulgación, y sobre otra herramienta)*

Aritmética nuestra, **no medida**: una línea de resultado ronda **15-25 tokens**; pasar de 300
coincidencias a 8 ahorra del orden de **5.000-7.000 tokens**. Real, pero en **una** llamada y solo
donde el texto no discrimina.

**Cómo medirlo de verdad**, en este orden:

1. **Aislado:** 15 preguntas reales sobre el repo del cliente; contar con el contador de la API
   los tokens de la salida de `rg` frente a los de `ast-grep`. Mide lo único que ast-grep cambia
   por sí solo: el tamaño del resultado.
2. **En sesión:** A/B con las mismas 10 tareas, una rama con instrucción de usar ast-grep y otra
   sin ella, comparando **tokens de entrada acumulados y número de turnos**. Aquí aparece lo que
   el paso 1 esconde: los patrones fallidos y sus reintentos.
3. **Descontar:** medir también los tokens de `Read`. Si acaba leyendo los mismos ficheros, la
   lectura se come el ahorro de la búsqueda.

Sin el paso 2, cualquier porcentaje nuestro sería tan poco fiable como los que criticamos.

## Riesgos y limitaciones, incluida la seguridad del código del cliente

- **¿Sale algo de la máquina? No.** Binario local, sin red, que lee ficheros; el MCP también corre
  en local. *(oficial)* El código del cliente llega a Anthropic por la misma vía que ya llega con
  `Grep`: dentro del contexto. Enviar 8 coincidencias en vez de 300 **reduce** la exposición.
- **Cadena de suministro.** MIT y muy activo, pero `npm -g` o `cargo` meten un árbol de
  dependencias en la máquina del cliente. Binario de release, fijado por versión y hash.
- **Curva de aprendizaje, y es el riesgo caro.** Acertar un patrón AST a la primera es más difícil
  que una regex. Un patrón fallido cuesta dos o tres turnos de depuración, y **eso puede gastar
  más tokens de los que habría gastado un grep ruidoso**.
- **Cobertura.** 30 y pico lenguajes de serie *(oficial)*; lo propietario o exótico —COBOL, ABAP,
  DSL internos— necesita parser tree-sitter propio, y en repos gigantes eso no es raro.
- **Solo sintaxis.** Sus autores lo declaran: sin tipos, sin flujo de control ni de datos.
  *(oficial)* No responde «quién llama a esto», que es la pregunta cara en millones de líneas;
  para eso está Serena (M-1).
- **Mantenimiento.** Las reglas YAML son código que envejece con el repo. Sin dueño, se pudren.

## Cómo se integraría en MyFactory

**Dos entradas distintas, y no conviene mezclarlas:**

| Qué | Dónde | Por qué |
| --- | --- | --- |
| El binario `ast-grep` | **`tools/`**, evaluado y **no copiado** | Un binario externo **no es una skill**. El caso de `rtk` otra vez |
| El skill `ast-grep/agent-skill` | `.claude/skills/`, copiable | Sí es skill: solo documentación y patrones, MIT |

**Ambas necesitan su fila de procedencia:** la del binario en `tools/SOURCES.md` (repositorio,
versión de release o commit, licencia, qué es), la del skill en `.claude/Skills/SOURCES.md`. Y hay
que anotar el acoplamiento: **el skill no sirve sin el binario**, así que copiarlo donde
`ast-grep --version` falla deja una skill que promete lo que no puede cumplir.

**Qué no meter:** el servidor MCP. Experimental por declaración propia, y su descripción de
herramientas ocupa contexto en toda sesión.

## Fuentes

Todas consultadas el **2026-09-24**.

- <https://ast-grep.github.io/guide/quick-start.html> — instalación, patrones, `--rewrite`
- <https://ast-grep.github.io/guide/tooling-overview.html> — `--json`, `sgconfig.yml`, `scan`, `test`
- <https://ast-grep.github.io/reference/languages.html> — lenguajes de serie y parsers propios
- <https://ast-grep.github.io/advanced/tool-comparison.html> — límites declarados: solo sintaxis
- <https://ast-grep.github.io/advanced/prompting.html> — guía oficial de uso con agentes
- <https://github.com/ast-grep/ast-grep-mcp> — MCP oficial, marcado experimental
- <https://github.com/ast-grep/agent-skill> — skill oficial para Claude Code, MIT
- <https://github.com/ast-grep/ast-grep> — API de GitHub: 16.021 ★, MIT, push 2026-09-24
- <https://github.com/defendend/Claude-ast-index-search/issues/69> — A/B independiente **de otra
  herramienta**: −35 % en localización, paridad en mezcla realista *(divulgación)*
