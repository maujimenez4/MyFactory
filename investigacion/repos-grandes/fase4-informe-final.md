# Fase 4 · Informe final: Claude Code en repositorios de millones de líneas

Investigación B, fase 4 (cerrar). Fecha: **2026-09-24**.

Consolidado leyendo **solo los cuatro subinformes de la fase 3**, como pide el método:
[Serena](fase3-serena.md), [ast-grep](fase3-ast-grep.md), [la skill de
prácticas](fase3-skill-practicas.md) y [Repomix](fase3-repomix.md).

## Resumen ejecutivo

1. **Ninguna de las tres herramientas tiene una cifra de ahorro publicada y creíble.** Serena no
   publica ninguna medida; el «~70 %» de Repomix es autodeclarado sin metodología; de ast-grep
   frente al `Grep` que ya trae Claude Code **no existe cifra, ni oficial ni en paper**.
2. **Serena es la única que cambia la naturaleza del trabajo**: responde «quién llama a esto» en
   una llamada, que es la pregunta cara en millones de líneas.
3. **Y trae el hallazgo más serio de toda la investigación:** una vulnerabilidad de ejecución de
   código al **abrir** un repositorio malicioso, antes de cualquier interacción (≤1.6.1, corregida
   en 1.7.0). Con versión mínima y repositorios de confianza está cerrada, pero condiciona el uso.
4. **La licencia de Serena, resuelta:** GPL-3.0-or-later **no toca el código del cliente**. Regula
   la distribución, no el uso, y Serena corre como proceso aparte. Solo obliga si la
   redistribuimos, la forkeamos o le escribimos un plugin.
5. **Repomix pierde** contra dejar que el agente explore solo — salvo en una forma concreta.
6. **La pieza que más rinde no es ninguna herramienta:** es la skill `repo-grande`, que no se
   instala y no tiene licencia que negociar.

## Kit recomendado para MyFactory

| Pieza | Dónde | Qué es y con qué condición |
|---|---|---|
| **`repo-grande`** | `.claude/Skills/` | La skill propia. Seis decisiones de sesión en menos de 120 líneas, con tres ficheros de apoyo que solo se leen si hacen falta. Lleva frontera explícita de «cuándo no»: sin ella, una skill se carga siempre y se paga siempre |
| **`serena`** | `tools/` | Servidor MCP, **no se copia** a `.claude/skills/`. Fila de procedencia con **GPL-3.0-or-later marcada como no permisiva** y **versión mínima 1.7.0** por seguridad |
| **Skill de uso de Serena** | `.claude/Skills/` | Nuestra, escrita desde cero, sin heredar la GPL: cuándo preferir `find_symbol` a `Read`, y recordarlo **tras compactar**, porque el agente olvida las herramientas de Serena |
| **`ast-grep`** | `tools/` | Binario evaluado y **no instalado por defecto**. Se saca solo cuando la búsqueda es estructural |
| **`repomix`** | `tools/` | Solo en la variante que puede ganar (abajo). La entrada lleva **encima nuestra restricción**, porque su skill oficial invita justo a lo contrario |

**Dos acoplamientos que hay que anotar y se olvidan:** tanto ast-grep como Repomix tienen una skill
oficial copiable **que exige su binario instalado**. Son dos filas de procedencia en dos ficheros
distintos, y la dependencia entre ellas no se ve desde ninguno de los dos.

## Guía de trabajo, en ocho pasos

1. **Arranca la sesión dentro del paquete**, no en la raíz del monorepo.
2. **Explora en tres capas** —arquitectura → módulo → fichero— y no abras un fichero hasta tener el mapa.
3. **Delega el barrido a un subagente** y quédate con la conclusión, no con los cincuenta ficheros.
4. **Acota la tarea a un módulo** y dilo en el prompt.
5. **Deja que el agente explore solo** con `Glob`, `Grep` y `Read`: es perezoso y paga por lo que lee.
   No le precargues un mapa.
6. **Para la pregunta cara —«quién usa esto»— usa Serena**, si el cliente lo ha autorizado. Ahí el
   `grep` cuesta decenas de ficheros y rondas de descarte.
7. **Saca ast-grep solo cuando el texto no discrimina**: buscar `raise HTTPException($$$)` devuelve
   las infracciones reales; `rg "HTTPException"` devuelve imports, tests y docstrings.
8. **Si usas Repomix, que su salida vaya a fichero y se consulte con `Grep`.** Nunca al contexto.

## Plan de incorporación por fases

**Fase I — lo que no se instala.** Escribir `repo-grande` con sus tres ficheros de apoyo y probarla
en un repositorio propio. Es la única pieza sin licencia que negociar, sin binario y sin riesgo de
seguridad. Medición: A/B de 15-20 tareas reales.

**Fase II — ast-grep.** El más barato de los tres: MIT, binario local, sin red. Se evalúa en
`tools/` y se mide con el método de tres pasos del subinforme —aislado, en sesión, y descontando
los `Read`—, porque el paso «en sesión» es el único que revela los patrones fallidos.

**Fase III — Serena, y solo tras cerrar tres cosas por escrito.** Permiso para instalar software
GPL en el entorno del cliente; constancia de que no la incorporamos al entregable; y permiso
explícito de que fragmentos de su código viajan al proveedor del modelo. Antes, prueba A/B en
repositorio propio.

**Fase IV — Repomix, si sobra tiempo.** Solo la rama C: `--compress` + `--include` acotado, salida a
fichero. Contra la rama A (el agente solo) y la B (mapa precargado). Es el candidato con menos
probabilidad de ganar.

## Riesgos y cómo mitigarlos

| Riesgo | Mitigación |
|---|---|
| **Ejecución de código al abrir un repositorio con Serena** (SSTI vía `.serena/project.yml`, ≤1.6.1, antes de cualquier interacción) | **Versión mínima 1.7.0** y nunca abrir con Serena un repositorio que no sea de confianza. `read_only` **no** protege frente al REPL: lo dicen sus propios documentos |
| **El índice de Serena vive dentro del repositorio del cliente** (`.serena/`, caché *pickle* y notas en markdown sobre su código) | A `.gitignore`, o se borra al cerrar el proyecto. Si no, acaba commiteado por descuido |
| **Creer que Serena es MIT.** Lo fue hasta la v1.7.0; el copyleft entró después | La fila de procedencia marca **GPL-3.0-or-later, no permisiva**. Fijar la v1.7.0 para esquivarla congela el producto sin parches — y es justo la versión que arregló el fallo de seguridad |
| **Un volcado de Repomix en claro en disco**, y de ahí al contexto y a la traza del proveedor | Salida a fichero consultado con `Grep`, nunca al contexto. **`--no-security-check` prohibido por guía**. Y Secretlint no elimina el riesgo: casa ~25 patrones conocidos, así que un token interno con formato propio pasa |
| **Un patrón AST fallido cuesta más que el grep ruidoso** — dos o tres turnos de depuración | Usar ast-grep solo donde el texto no discrimina. En búsqueda literal, ripgrep gana |
| **Una skill que se carga siempre y no aporta** es contexto pagado en cada sesión | Frontera de «cuándo no» en la propia descripción. Si estorba, hay salidas oficiales: acotar por `paths` o desactivar la invocación por modelo |
| **Adoptar una cifra ajena.** El «48-70 %» que circula de Repomix no compara contra `--compress`: es de un post-procesador de terceros midiendo contra la salida cruda | Ninguna cifra entra en una propuesta sin decir contra qué compara |
| **Optimizar y empeorar.** Hay un experimento público que abandonó su pila de optimización por degradar la corrección | Coste **por tarea completada con éxito**, juzgada por una persona. Un ahorro con peor acierto no es un ahorro |

## Lo que sigue sin saberse

- **Si Serena tiene telemetría.** No se encontró documentación de que envíe nada, pero eso es
  ausencia de evidencia, no evidencia de ausencia. **Queda por verificar en el código** antes de
  usarla con un cliente.
- **Cuánto ahorra cada una de verdad.** Las tres carecen de medición aplicable. Los tres subinformes
  dejan el método en lugar del número, que es lo honesto.
- **El caso ManoMano**, el único relato de Serena en un repositorio grande de verdad, **no fue
  accesible** (HTTP 403). Se cita como anécdota, no como medida.
