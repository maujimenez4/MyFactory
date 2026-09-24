# Fase 2 · Clasificación

Investigación B, fase 2 (cerrar). Fecha: **2026-09-24**. Entrada: los 25 candidatos de
[`fase1-exploracion.md`](fase1-exploracion.md).

## Cómo se puntúa

Cinco criterios, de 1 a 5. **En los cinco, 5 es mejor** — también en riesgo, donde **5
significa riesgo bajo**. Se dice porque sumar una columna donde 5 fuera «muy peligroso»
premiaría lo peor.

| Criterio | Qué mide |
|---|---|
| **Ahorro** | Tokens que quita por tarea |
| **Madurez** | Actividad real y uso, comprobados en la API de GitHub el 2026-09-24 |
| **Encaje** | Lo que cuesta meterlo en MyFactory y mantenerlo |
| **Riesgo** | **5 = bajo.** Acceso al código del cliente, envío a terceros, permisos, licencia |
| **Escala** | Si sigue sirviendo con millones de líneas |

**Las puntuaciones de ahorro son estimaciones nuestras**, no medidas. Ninguna cifra de
la fase 1 la hemos comprobado. Por eso el ahorro pesa igual que los demás criterios y no
más: premiar la promesa de ahorro sería premiar el marketing.

---

## Ranking

| # | Candidato | Ahorro | Madurez | Encaje | Riesgo | Escala | **Total** |
|---|---|:--:|:--:|:--:|:--:|:--:|:--:|
| N-1 | Subagentes | 5 | 5 | 5 | 5 | 5 | **25** |
| N-6 | `CLAUDE.md` jerárquico por carpeta | 4 | 5 | 5 | 5 | 5 | **24** |
| C-1 | Prompt caching | 5 | 5 | 5 | 5 | 4 | **24** |
| N-2 | Skills con revelación progresiva | 4 | 5 | 5 | 5 | 4 | **23** |
| N-7 | Selección de modelo por tarea | 5 | 5 | 5 | 5 | 3 | **23** |
| S-1 | Empezar la sesión en el paquete | 4 | 4 | 5 | 5 | 5 | **23** |
| S-4 | Exploración por capas | 4 | 4 | 5 | 5 | 5 | **23** |
| S-6 | Tareas acotadas por módulo | 4 | 4 | 5 | 5 | 5 | **23** |
| S-2 | `CLAUDE.md` corto y repartido | 4 | 4 | 5 | 5 | 4 | **22** |
| S-3 | Ficheros de exclusión | 3 | 5 | 5 | 5 | 4 | **22** |
| S-5 | Documentación de arquitectura para la IA | 4 | 4 | 4 | 5 | 5 | **22** |
| C-2 | Ordenar el prompt para la caché | 4 | 5 | 4 | 5 | 4 | **22** |
| M-4 | **ast-grep** | 4 | 5 | 4 | 5 | 4 | **22** |
| M-1 | **Serena** | 5 | 5 | 3 | 4 | 5 | **22** |
| C-3 | Artefactos entre sesiones | 4 | 3 | 5 | 5 | 4 | **21** |
| N-5 | Compactación automática | 3 | 5 | 5 | 5 | 3 | **21** |
| N-3 | Plugins que se apagan | 2 | 5 | 4 | 4 | 3 | **18** |
| N-4 | Hooks de ciclo de vida | 3 | 5 | 3 | 4 | 3 | **18** |
| M-5 | github-mcp-server | 3 | 5 | 4 | 3 | 3 | **18** |
| E-1 | Repomix (solo `--compress`) | 3 | 5 | 4 | 4 | 2 | **18** |
| E-2 | Repo map de Aider (la idea) | 4 | 3 | 2 | 5 | 4 | **18** |
| M-3 | mcp-language-server | 4 | 2 | 3 | 5 | 3 | **17** |
| M-2 | **Claude Context (Zilliz)** | 4 | 3 | 2 | **1** | 4 | **14** |
| E-3 | Gitingest | 2 | 4 | 3 | 3 | 1 | **13** |
| E-4 | code2prompt | 2 | 4 | 3 | 3 | 1 | **13** |

---

## Grupos

### Incorporar ya

Nada de esto se instala: **son configuración y forma de trabajar**, y por eso entran sin
piloto. Lo que MyFactory aporta es una **skill propia que las reúna**, más una plantilla
de `CLAUDE.md` para repositorios grandes.

| Candidato | Por qué entra ya |
|---|---|
| **N-1 Subagentes** | Es la pieza nativa que resuelve el problema de raíz: explorar sin ensuciar la sesión. Ya lo usamos |
| **N-6 `CLAUDE.md` jerárquico** | Coste cero y el efecto es directo: el raíz se paga en **cada** sesión |
| **C-1 Prompt caching** | Claude Code lo aplica solo. Lo nuestro es C-2: no romperlo poniendo arriba lo que cambia |
| **N-2 Skills progresivas** | Ya es como fabricamos skills aquí |
| **N-7 Selección de modelo** | La palanca más directa sobre la factura |
| **S-1, S-2, S-4, S-6** | Cuatro reglas de trabajo, sin herramienta que mantener |
| **S-3 Exclusión** | Un fichero por repositorio |
| **S-5 Doc de arquitectura para la IA** | Ya lo hacemos en `ciberpunk-storymaker` con `docs/architecture.md` |
| **C-2 Orden del prompt** | Lo estable arriba, lo variable abajo |

### Probar en piloto

| Candidato | Qué hay que resolver en el piloto |
|---|---|
| **M-1 Serena** (22) | El mejor candidato técnico: símbolos en vez de líneas, 40+ lenguajes, LSP **en local** — el código no sale de la máquina. Su pega es la **licencia GPL-3.0-or-later** (su componente SolidLSP es MIT; la aplicación, no). Usarla como herramienta externa no obliga a nada sobre el código del cliente, pero eso hay que confirmarlo por escrito antes de ofrecerla a un tercero, no suponerlo |
| **M-4 ast-grep** (22) | Binario, MIT, activo hoy. Sustituye barridos de `grep` que devuelven cientos de líneas inútiles. El piloto debe medir cuánto ahorra de verdad frente a `Grep` a secas, que ya está bien afinado |
| **C-3 Artefactos entre sesiones** (21) | Funciona —la bitácora de hoy lo demuestra— pero no está escrito como método. El piloto es convertirlo en convención |
| **E-1 Repomix en `--compress`** (18) | Solo para módulos acotados, nunca el repositorio entero. Dos cosas a favor: Tree-sitter deja firmas en vez de cuerpos, y **Secretlint** evita colar credenciales. A medir: si su mapa comprimido gana a un `find` bien hecho |
| **E-2 Repo map de Aider** (18) | No se instala Aider: se copia la **idea** de mapa por firmas. Piloto barato: generarlo con Tree-sitter y ver si el agente lo usa |
| **N-4 Hooks** (18) | Útiles para lint y registro sin gastar contexto. Piloto pequeño, porque un hook mal puesto se dispara en cada llamada |

### Descartar

| Candidato | Motivo |
|---|---|
| **M-2 Claude Context** (14) | **Descartado por seguridad, no por calidad.** Necesita un proveedor de *embeddings* —OpenAI por defecto— y una base vectorial, con Zilliz Cloud como opción recomendada. Eso significa **mandar el código del cliente a dos terceros**. Existe un modo local (Ollama + Milvus) que la propia página no documenta. Para clientes con repositorios privados es inaceptable de partida; si algún día se ofrece, será con el modo local comprobado y por escrito. Además lleva **desde el 2026-07-14 sin tocarse** |
| **E-3 Gitingest** (13) | Vuelca el repositorio a texto plano. En millones de líneas no cabe, y no tiene modo comprimido como Repomix |
| **E-4 code2prompt** (13) | Lo mismo. Su recuento de tokens está bien, pero eso no justifica mantener una herramienta |
| **M-3 mcp-language-server** (17) | Hace lo mismo que Serena con menos alcance y **seis meses sin un commit** (2026-03-01, 1.599 ★). Si queremos LSP, es Serena |
| **M-5 github-mcp-server** (18) | Buena herramienta, **pero no resuelve nuestro problema**: ahorra clonar, no ahorra contexto. Se queda anotado para el caso de un cliente que no permita copia local |
| **N-3 Plugins que se apagan** (18) | No es un candidato, es higiene: no instalar lo que no se usa. Se recoge en la guía |
| **N-5 Compactación** (21) | Puntúa alto y aun así no es una medida de ahorro: **es una red de seguridad que pierde detalle**. Se deja activa, no se cuenta como estrategia |

---

## Lo que esta fase deja abierto

1. **Ninguna cifra de ahorro está medida.** Todo el ranking descansa en estimaciones y en
   lo que afirman las fuentes. La fase 3 tiene que decir **cómo se mide**, y la primera
   tarea del piloto es una línea base propia.
2. **La licencia de Serena** decide si el mejor candidato técnico es ofrecible a un
   cliente. Es lo primero que hay que cerrar.
3. **Dos candidatos de «incorporar ya» no se pueden comprobar con una fuente oficial**
   (S-1 y S-2 vienen de guías de terceros). Son razonables y baratos, pero conviene saber
   que se adoptan por criterio, no por evidencia.
