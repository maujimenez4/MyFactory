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

Estado: [x] Fase 1  [ ] Fase 2  [ ] Fase 3  [ ] Fase 4

Métrica principal: coste por tarea completada con éxito (agente) y coste por historia/PR aceptada (código).

### Fase 1: opciones exploradas

- Opción A: diagnóstico rápido con mejoras inmediatas (4 semanas). Recoger datos, detectar gasto excesivo, aplicar 3-5 mejoras rápidas y entregar un informe.
- Opción B: programa completo con experimentación (10-12 semanas). KPIs, línea base con benchmark de tareas reales, experimentos aislados, implantación con dashboards y guía, y transferencia.
- Opción C: piloto y escalado (6-8 semanas). Un flujo agéntico y 1-2 equipos, optimización a fondo y playbook replicable.

### Fase 2, 3 y 4

Pendiente.

## 3. Investigación B: skills, plugins y estrategias para repositorios de millones de líneas

(Se rellena en la Parte 2.)

## 4. Modificaciones futuras (no implementadas)

| Idea | Motivo | Estado |
|---|---|---|
| Gestión de contexto tipo acordeón (plegar/desplegar bloques) | Reducir tokens sin perder información | Propuesta |
| Enrutamiento automático de modelos por complejidad | Modelos baratos para tareas simples | Propuesta |
| Dashboard de consumo por equipo con alertas | Mantener el ahorro en el tiempo | Propuesta |
| Plantilla estándar de fichero de instrucciones para repositorios | Menos tokens por sesión | Propuesta |

## 5. Registro de cambios y cómo revertir

| Versión | Fecha | Cambio | Commit |
|---|---|---|---|
| v0.1 | 2026-09-24 | Método e Investigación A fase 1 | `4a606ed` |

Para revertir:

```bash
git log --oneline -- roadmap.md
git checkout <hash> -- roadmap.md
git commit -m "revert(roadmap): volver a <version>"
```
