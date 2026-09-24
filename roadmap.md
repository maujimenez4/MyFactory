# Roadmap de investigación: optimización de tokens

## 1. Método: acordeón (divergente-convergente)

| Fase | Movimiento | Acción | Resultado |
|---|---|---|---|
| 1. Explorar | Abrir | Generar opciones | 3-5 opciones |
| 2. Clasificar | Cerrar | Ordenar por impacto, esfuerzo y riesgo | Ranking |
| 3. Profundizar | Abrir | Un subinforme por opción, cada uno con contexto limpio | Subinformes |
| 4. Consolidar | Cerrar | Unir, quitar duplicados, resolver contradicciones | Informe final |

Regla: cada subinforme se hace en un contexto limpio. En la consolidación solo se leen los subinformes.

## 2. Investigación A: optimización de tokens en solución agéntica y generación de código

**Es la investigación principal**, y por eso lleva el método completo antes que la B.

Estado: [x] Fase 1  [x] Fase 2  [x] Fase 3  [x] Fase 4 — **completada**

Métrica principal: coste por tarea completada con éxito (agente) y coste por historia/PR aceptada (código). No «tokens por llamada»: una estrategia que ahorra tokens y hace fracasar la tarea sale más cara, porque hay que repetirla entera.

### Encuadre del proyecto: tres formatos posibles

- Opción A: diagnóstico rápido con mejoras inmediatas (4 semanas). Recoger datos, detectar gasto excesivo, aplicar 3-5 mejoras rápidas y entregar un informe.
- Opción B: programa completo con experimentación (10-12 semanas). KPIs, línea base con benchmark de tareas reales, experimentos aislados, implantación con dashboards y guía, y transferencia.
- Opción C: piloto y escalado (6-8 semanas). Un flujo agéntico y 1-2 equipos, optimización a fondo y playbook replicable.

### Fase 1: estrategias exploradas

22 estrategias en [`investigacion/agentica-tokens/fase1-exploracion.md`](investigacion/agentica-tokens/fase1-exploracion.md), en cinco bloques: arquitectura del contexto, diseño de herramientas, modelo y forma de la llamada, proceso de trabajo y medición.

Cada cifra lleva su nivel de confianza —**oficial** de Anthropic, **paper**, o **divulgación** de un blog comercial—, porque esa diferencia pesa más que el número.

### Fase 2: clasificación

Ranking completo en [`investigacion/agentica-tokens/fase2-clasificacion.md`](investigacion/agentica-tokens/fase2-clasificacion.md), sobre impacto, esfuerzo, riesgo y **evidencia**. En los cuatro, 5 es mejor.

| Grupo | Estrategias |
|---|---|
| **Aplicar ya** (17-20) | Prompt caching y el orden del prompt · subagentes con contexto limpio · herramientas que no vuelcan · apagar las que no se usan · salidas estructuradas · búsquedas pequeñas · tareas acotadas · reintento dirigido · medir coste por tarea y tasa de éxito juntos |
| **Probar en piloto** (15-17) | Recuperación *just-in-time* · Batch API · notas fuera del contexto · techo de gasto con corte · línea base con tareas reales |
| **Con cuidado** | **Enrutamiento por complejidad** (impacto 5, evidencia 1) · **ejecución de código con MCP** (−98,7 % oficial, pero exige entorno de ejecución) · compactación (es red de seguridad, no ahorro) |

**Tres cosas que el ranking no dice y deciden el plan:**

1. **Las estrategias de contexto se pisan.** Subagentes, compactación, notas y *just-in-time* atacan el mismo gasto: sumar sus porcentajes da un ahorro imposible. Se aplica la mayor y se vuelve a medir.
2. **Las que sí se suman** son cuatro palancas sobre cuatro factores distintos: caché (precio del token), batch (precio de la llamada), enrutamiento (precio del modelo) y contexto (número de tokens).
3. **La medición va primera en el tiempo**, aunque no lidere la tabla. Sin línea base con tareas reales del cliente, ningún porcentaje significa nada — y el informe está lleno de porcentajes ajenos que lo demuestran.

### Fase 3: tres subinformes

Uno por opción, cada uno hecho en contexto limpio por un subagente distinto:

- [Ejecución de código con MCP](investigacion/agentica-tokens/fase3-ejecucion-codigo-mcp.md) — la cifra mayor de la investigación, y la que peor resiste el escrutinio.
- [Enrutamiento de modelos](investigacion/agentica-tokens/fase3-enrutamiento-modelos.md) — impacto estimado máximo, evidencia mínima.
- [Línea base de medición](investigacion/agentica-tokens/fase3-linea-base-medicion.md) — no ahorra nada; es la condición para afirmar que algo ahorra.

### Fase 4: informe final

**[`fase4-informe-final.md`](investigacion/agentica-tokens/fase4-informe-final.md)**, consolidado leyendo solo los subinformes.

Lo que la consolidación corrigió de la fase 2:

1. **Las tres cifras que abrieron la investigación no sobreviven de cerca.** El −98,7 % es un ejemplo ilustrativo sin metodología; el 73-87 % lo publica quien vende optimización y no se pudo reverificar.
2. **Las oficiales son menores y mejores:** −85 % con *tool search* y −37 % con *programmatic tool calling*, las dos **con el acierto subiendo**.
3. **Por tarea resuelta, el modelo caro sale más barato** —Opus 5.5 a esfuerzo medio, 92,8 % de SWE-bench Pro a $0,22/tarea frente a $1,19—. Eso invierte la premisa del enrutamiento.
4. **Lo barato ya está puesto:** *tool search* viene activo por defecto en Claude Code.
5. **Dos palancas que la fase 2 dio por independientes se pisan:** enrutar rompe la caché, porque las cachés son por modelo.
6. **Medir cuesta más de lo previsto:** detectar diez puntos de diferencia pide ~293 tareas por rama sin emparejar.

## 3. Investigación B: skills, plugins y estrategias para repositorios de millones de líneas

Estado: [x] Fase 1  [x] Fase 2  [ ] Fase 3  [ ] Fase 4

Trabajo en [`investigacion/repos-grandes/`](investigacion/repos-grandes/): 25 candidatos
explorados en [fase 1](investigacion/repos-grandes/fase1-exploracion.md) y puntuados en
[fase 2](investigacion/repos-grandes/fase2-clasificacion.md) sobre cinco criterios —ahorro,
madurez, encaje, riesgo y escala—, donde **5 siempre es mejor, también en riesgo**.

### Resumen del ranking

| Grupo | Candidatos |
|---|---|
| **Incorporar ya** (22-25) | Subagentes · `CLAUDE.md` jerárquico · prompt caching y el orden del prompt · skills progresivas · selección de modelo · empezar la sesión en el paquete · exploración por capas · tareas por módulo · ficheros de exclusión · doc de arquitectura para la IA |
| **Probar en piloto** (18-22) | **Serena** · **ast-grep** · artefactos entre sesiones · Repomix en `--compress` · la idea del repo map de Aider · hooks |
| **Descartar** | **Claude Context** (manda el código a OpenAI y Zilliz) · Gitingest y code2prompt (no caben en millones de líneas) · mcp-language-server (seis meses parado) · github-mcp-server (ahorra clonar, no contexto) |

Nada de lo de «incorporar ya» se instala: **son configuración y forma de trabajar**. Lo
que MyFactory aportaría es una skill que las reúna y una plantilla de `CLAUDE.md` para
repositorios grandes.

**Dos avisos que pesan más que el orden de la tabla:**

1. **Ninguna cifra de ahorro está medida por nosotros.** El ranking se apoya en
   estimaciones y en lo que afirman las fuentes. La primera tarea de cualquier piloto es
   una línea base propia.
2. **Serena, el mejor candidato técnico, es GPL-3.0-or-later.** Su componente SolidLSP es
   MIT; la aplicación, no. Antes de ofrecerla a un cliente hay que cerrar por escrito qué
   implica, no suponerlo.

## 4. Modificaciones futuras (no implementadas)

| Idea | Motivo | Estado |
|---|---|---|
| Gestión de contexto tipo acordeón (plegar/desplegar bloques) | Reducir tokens sin perder información | Propuesta |
| Enrutamiento automático de modelos por complejidad | Modelos baratos para tareas simples | Propuesta |
| Dashboard de consumo por equipo con alertas | Mantener el ahorro en el tiempo | Propuesta |
| Plantilla estándar de fichero de instrucciones para repositorios | Menos tokens por sesión | Propuesta |
| Skill `medir-coste-agente` con sus dos plantillas | Es la pieza que falta: el fallo típico no es recoger mal los datos, es concluir de una corrida | **Recomendada** (informe final A) |
| Skill `higiene-mcp` | Lo barato y sin infraestructura: definiciones que no se precargan, salidas acotadas, servidores apagados | **Recomendada** (informe final A) |
| `settings.json` de referencia en `tools/` | Reparto de modelos por subagente, con el aviso de no adoptarlo sin medir | Propuesta |
| Generador de envoltorios MCP→código | Ahorro grande, pero es infraestructura y no hay medición propia | **Descartada por ahora** |
| Pasarela de enrutamiento de modelos | Misma razón, y además rompe la caché | **Descartada por ahora** |
| Nota en cada skill que lanza subagentes | Una skill copiada hereda el modelo de subagente del proyecto destino: es parte de su contrato | Propuesta |

## 5. Registro de cambios y cómo revertir

| Versión | Fecha | Cambio | Commit |
|---|---|---|---|
| v0.1 | 2026-09-24 | Método e Investigación A fase 1 | `4a606ed` |
| v0.2 | 2026-09-24 | Investigación A fases 1 y 2, e Investigación B fases 1 y 2 | `7e8eec2`, `a15586b` |
| v0.3 | 2026-09-24 | Investigación A completa: tres subinformes e informe final | `1fc3177`, `9900df9` |

Para revertir:

```bash
git log --oneline -- roadmap.md
git checkout <hash> -- roadmap.md
git commit -m "revert(roadmap): volver a <version>"
```
