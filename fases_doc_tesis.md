# Plan de Escritura — Documento de Tesis ArchIA

**Duración total:** 4 semanas (1 mes)
**Fecha estimada de inicio:** 28 de abril, 2026
**Fecha estimada de entrega:** 26 de mayo, 2026
**Autores:** Santiago Casasbuenas, Juan Pablo Castro, Manuel Gomez

---

## Principio rector

Cada sección del documento debe responder al argumento fundamental de la tesis:

> *"Un sistema de agentes de IA especializados, orquestados por fases de ADD 3.0 y respaldados por RAG sobre literatura reconocida, puede asistir al arquitecto de software en la formalización de drivers arquitectónicos — transformando requisitos informales en ASRs estructurados, estilos evaluados y tácticas trazables."*

Todo lo que se escriba debe pasar este filtro: ¿esto soporta o es necesario para defender ese argumento? Si no, se recorta o se mueve a trabajo futuro.

---

## Semana 1 (Abril 28 – Mayo 4): Bases y Fundamentos

**Objetivo:** Tener el marco teórico completo y las secciones de contexto escritas.

### Tareas

| # | Sección | Responsable | Páginas est. | Entregable |
|---|---------|-------------|--------------|------------|
| 1.1 | Marco Teórico — IAG y LLMs | [ASIGNAR] | 4-5 pp | Borrador completo |
| 1.2 | Marco Teórico — Sistemas Multi-Agente | [ASIGNAR] | 2-3 pp | Borrador completo |
| 1.3 | Marco Teórico — RAG | [ASIGNAR] | 2-3 pp | Borrador completo |
| 1.4 | Marco Teórico — Memoria Persistente | [ASIGNAR] | 1-2 pp | Borrador completo |
| 1.5 | Marco Teórico — ADD 3.0 | [ASIGNAR] | 3-4 pp | Borrador completo |
| 1.6 | Antecedentes (estado del arte) | [ASIGNAR] | 3-4 pp | Borrador + tabla comparativa |
| 1.7 | Recopilar referencias (.bib) | [ASIGNAR] | — | Archivo referencias.bib con mínimo 20 fuentes |

### Notas

- **ADD 3.0 es la subsección más crítica.** Debe explicar con claridad los 7 pasos del método y marcar explícitamente cuáles cubre ArchIA (pasos 1-4) y cuáles no.
- Las subsecciones de IAG, LLM y RAG se pueden escribir en paralelo — son independientes.
- La subsección de ADD 3.0 debe escribirse antes del Problema (semana 2), porque el problema se define en términos de ADD.
- Cada subsección debe cerrar con un párrafo de "conexión con ArchIA" que justifique por qué ese concepto es relevante para el proyecto.

### Entregable de la semana
- Marco teórico completo (~15 páginas)
- Antecedentes completo (~4 páginas)
- Archivo referencias.bib funcional

---

## Semana 2 (Mayo 5 – Mayo 11): Problema, Objetivos y Alcance

**Objetivo:** Tener el corazón argumentativo del documento definido con precisión.

### Tareas

| # | Sección | Responsable | Páginas est. | Entregable |
|---|---------|-------------|--------------|------------|
| 2.1 | Problema | [ASIGNAR] | 2-3 pp | Borrador completo |
| 2.2 | Objetivos (general + específicos) | [ASIGNAR] | 1-2 pp | Borrador completo |
| 2.3 | Definición de Problemática y Alcance | [ASIGNAR] | 2-3 pp | Borrador con tablas de E/S |
| 2.4 | Limitaciones del Proyecto | [ASIGNAR] | 1-2 pp | Borrador completo |
| 2.5 | Introducción | [ASIGNAR] | 2-3 pp | Borrador completo |
| 2.6 | Revisión cruzada semana 1 | TODOS | — | Correcciones del marco teórico |

### Notas

- **Problema y Alcance se escriben juntos** porque son dos caras de la misma moneda: el problema define la brecha, el alcance define qué porción de esa brecha ataca ArchIA.
- La sección de Alcance DEBE incluir las tablas de entradas y salidas que pidieron los arquitectos de Pragma. Sin esto, la tesis no pasa la revisión.
- La Introducción se escribe después del Problema y Alcance, porque resume el documento completo.
- Las Limitaciones deben ser honestas. No intentar esconder lo que no se hizo — presentarlo como decisiones de alcance conscientes.

### Entregable de la semana
- Problema + Objetivos + Alcance + Limitaciones (~8 páginas)
- Introducción (~3 páginas)
- Marco teórico revisado y corregido

---

## Semana 3 (Mayo 12 – Mayo 18): Diseño, Implementación y Descripción

**Objetivo:** Documentar todo lo técnico — esta es la sección más larga del documento.

### Tareas

| # | Sección | Responsable | Páginas est. | Entregable |
|---|---------|-------------|--------------|------------|
| 3.1 | Arquitectura General del Sistema | [ASIGNAR] | 2-3 pp | Borrador + diagrama de arquitectura |
| 3.2 | Backend (FastAPI, RAG, LLM) | [ASIGNAR] | 4-5 pp | Borrador + fragmentos de código |
| 3.3 | Frontend (React, Atomic Design) | [ASIGNAR] | 3-4 pp | Borrador + capturas de pantalla |
| 3.4 | Diseño Anterior del Grafo | [ASIGNAR] | 2-3 pp | Borrador + diagrama del grafo v1 |
| 3.5 | Diseño Final del Grafo | [ASIGNAR] | 3-4 pp | Borrador + diagrama del grafo final |
| 3.6 | Descripción de la Herramienta (walkthrough) | [ASIGNAR] | 3-4 pp | Borrador + capturas del flujo completo |
| 3.7 | Preparar casos de prueba para Resultados | TODOS | — | 3-5 escenarios definidos y ejecutados |

### Notas

- **Dividir el trabajo por conocimiento:** quien hizo el backend escribe el backend, quien hizo el frontend escribe el frontend.
- **Diseño Anterior vs. Final del Grafo** es una sección valiosa porque muestra evolución y aprendizaje. Incluir el feedback de Darío como motivación del rediseño.
- **El walkthrough (Descripción de la Herramienta)** es lo que van a presentar a los evaluadores. Debe ser claro, con capturas reales, y debe seguir el flujo ADD: descripción → ASR → estilo → tácticas → diagrama.
- Los casos de prueba (tarea 3.7) deben prepararse esta semana para poder documentar resultados en la semana 4. Definir al menos:
  - Caso e-commerce (latencia/escalabilidad)
  - Caso API pública (escalabilidad)
  - Caso sistema crítico (disponibilidad)
  - Caso con PDF cargado (modo DOC-ONLY)

### Entregable de la semana
- Diseño e Implementación completo (~18 páginas)
- Descripción de la Herramienta completo (~4 páginas)
- Casos de prueba ejecutados y documentados (listos para semana 4)

---

## Semana 4 (Mayo 19 – Mayo 26): Resultados, Conclusiones y Cierre

**Objetivo:** Cerrar el documento con evidencia, análisis y reflexión.

### Tareas

| # | Sección | Responsable | Páginas est. | Entregable |
|---|---------|-------------|--------------|------------|
| 4.1 | Resultados | [ASIGNAR] | 5-7 pp | Borrador con tablas y análisis |
| 4.2 | Pruebas | [ASIGNAR] | 3-4 pp | Borrador con tablas de pruebas |
| 4.3 | Conclusiones | [ASIGNAR] | 2-3 pp | Borrador completo |
| 4.4 | Trabajo Futuro | [ASIGNAR] | 1-2 pp | Borrador completo |
| 4.5 | Agradecimientos | TODOS | 1 pp | Texto final |
| 4.6 | Anexos (prompts, ejemplo completo) | [ASIGNAR] | 5-10 pp | Anexos completos |
| 4.7 | Revisión final completa | TODOS | — | Documento pulido |
| 4.8 | Verificar compilación LaTeX | [ASIGNAR] | — | PDF sin errores |

### Notas

- **Resultados es la sección que defiende la tesis.** Para cada caso de prueba, mostrar: entrada → ASR generado → estilo recomendado → tácticas seleccionadas. Luego analizar: ¿el ASR tiene las 6 partes? ¿Las métricas son realistas? ¿El estilo es pertinente? ¿Las tácticas son trazables?
- Si tienen feedback formal de los arquitectos de Pragma, incluirlo aquí. Si no, documentar el feedback de las reuniones (ya lo tienen en las notas de la reunión del 22 de abril).
- **Conclusiones** debe retomar la pregunta de investigación de la sección Problema y responderla explícitamente con evidencia de la sección Resultados.
- **Revisión final (tarea 4.7):** cada integrante lee el documento completo buscando inconsistencias, errores ortográficos, referencias rotas y coherencia narrativa.

### Entregable de la semana
- Resultados + Pruebas completo (~10 páginas)
- Conclusiones + Trabajo Futuro completo (~4 páginas)
- Anexos completos
- **DOCUMENTO FINAL COMPILADO** (~55-65 páginas)

---

## Resumen de páginas estimadas

| Sección | Páginas |
|---------|---------|
| Introducción | 2-3 |
| Agradecimientos | 1 |
| Marco Teórico | 12-15 |
| Problema | 2-3 |
| Objetivos | 1-2 |
| Antecedentes | 3-4 |
| Alcance | 2-3 |
| Limitaciones | 1-2 |
| Diseño e Implementación | 15-20 |
| Descripción de la Herramienta | 3-4 |
| Resultados | 5-7 |
| Pruebas | 3-4 |
| Conclusiones | 2-3 |
| Trabajo Futuro | 1-2 |
| Anexos | 5-10 |
| **TOTAL** | **~58-82** |

---

## Dependencias críticas

```
Semana 1: Marco Teórico + Antecedentes
    │
    ├── ADD 3.0 (subsección) ──→ Problema (semana 2)
    │                              │
    │                              ├──→ Objetivos
    │                              └──→ Alcance
    │
    └── RAG + Multi-Agente ────→ Backend (semana 3)
                                   │
Semana 3: Diseño + Casos ─────→ Resultados (semana 4)
                                   │
                                   └──→ Conclusiones
```

## Reglas del equipo

1. **Cada tarea tiene UN responsable.** No "todos hacemos todo". Asignar nombres en las tablas de arriba.
2. **Revisión cruzada semanal.** Cada viernes, todos leen lo que los demás escribieron y dejan comentarios.
3. **Compilar LaTeX cada 2-3 días.** No esperar al final para descubrir errores de compilación.
4. **No escribir más de lo necesario.** Si una sección se pasa de las páginas estimadas, probablemente está incluyendo contenido que no aporta al argumento central.
5. **Los \TODO{} del esqueleto son la guía.** Cada tarea consiste en reemplazar un \TODO{} con contenido real siguiendo las instrucciones en los comentarios LaTeX.

---

## Registro de avance

### Sesión del 6 de mayo de 2026

**Contexto:** sesión de trabajo con asistente IA para definir el alcance, crear el esqueleto y escribir las primeras secciones técnicas del documento.

#### Decisiones tomadas

1. **Argumento fundamental definido:** "Un sistema de agentes de IA especializados, orquestados por fases de ADD 3.0 y respaldados por RAG sobre literatura reconocida, puede asistir al arquitecto de software en la formalización de drivers arquitectónicos."
2. **Alcance acotado:** ArchIA cubre los pasos 1-4 de ADD 3.0 (ASR, estilo, tácticas). No se presenta como plataforma SaaS ni como herramienta de ADD completo.
3. **Entradas y salidas formalizadas:** se definieron con precisión las 4 entradas (descripción informal, atributo de calidad, contexto de negocio, PDF) y las 4 salidas (ASR formal, estilo recomendado, 3 tácticas, diagrama complementario).

#### Archivos creados

| Archivo | Descripción |
|---------|-------------|
| `tesis.tex` | Esqueleto completo del documento LaTeX con todas las secciones, paquetes y guías en comentarios |
| `fases_doc_tesis.md` | Este archivo — plan de 4 semanas |
| `.gitignore` | Ignora la carpeta `build/` de compilación LaTeX |

#### Secciones escritas en `tesis.tex`

| Sección | Estado | Páginas aprox. | Fuente principal |
|---------|--------|:-:|---|
| Diseño Anterior del Grafo | **Escrita** | ~2 | `archia_resumen_consolidado.md` §2 |
| Backend (modularización, RAG indexado, optimizaciones) | **Escrita** | ~4 | §3, §6, §9 del resumen |
| Diseño Final del Grafo (flujo, nodos, multi-intent, QA routing, Ledger, supersession) | **Escrita** | ~4 | §3, §4, §7, §10-11 del resumen |
| Trabajo Futuro | **Escrita** | ~2 | §13 del resumen |

#### Fragmentos de código citados en el documento

Cada listing incluye el path relativo del archivo y las líneas exactas verificadas contra el estado actual del repositorio.

| Listing | Archivo | Líneas | Contenido |
|---------|---------|--------|-----------|
| `lst:estructura-graph` | `back/src/graph/` | — | Árbol de directorios del paquete modularizado |
| `lst:tag-chunk` | `back/build_vectorstore.py` | 44-70 | Algoritmo de auto-tagging de chunks |
| `lst:indexed-retriever` | `back/src/graph/nodes/tactics/common.py` | 52-56 | Retriever con filtros de metadata |
| `lst:rag-parallel` | `back/src/graph/nodes/styles/common.py` | 48-63 | Paralelización RAG con ThreadPoolExecutor |
| `lst:lru-cache` | `back/src/graph/nodes/classifier.py` | 81-83 | Decorador @lru_cache en clasificador |
| `lst:render-async` | `back/src/services/diagram_render.py` | 254-290 | Renderizado async de Graphviz con caché |
| `lst:workflow-wiring` | `back/src/graph/workflow.py` | 182-203 | Cableado de nodos y edges del grafo |
| `lst:multi-intent-state` | `back/src/graph/state.py` | 152-154 | Campos de planificación multi-intent |
| `lst:qa-dynamic` | `back/src/graph/workflow.py` | 162-169 | Registro dinámico de nodos por QA |
| `lst:decision-type` | `back/src/ledger/types.py` | 43-58 | Estructura Decision del Design Ledger |

#### Secciones pendientes (con \TODO{} en el esqueleto)

| Sección | Semana planeada | Notas |
|---------|:---:|---|
| Introducción | 2 | Escribir después de Problema y Alcance |
| Marco Teórico (6 subsecciones) | 1 | Requiere redacción académica original |
| Problema | 2 | Pregunta de investigación ya definida en los comentarios |
| Objetivos | 2 | General y específicos ya propuestos en los comentarios |
| Antecedentes | 1 | Requiere revisión de literatura + tabla comparativa |
| Alcance | 2 | Tablas de E/S ya definidas conceptualmente |
| Limitaciones | 2 | Puntos ya listados en los comentarios |
| Arquitectura General | 3 | Diagrama de alto nivel + tabla de stack |
| Frontend | 3 | Requiere capturas de pantalla de la interfaz |
| Descripción de la Herramienta | 3 | Walkthrough con caso TravelHub del resumen §12 |
| Resultados | 4 | Casos de prueba + evaluación con arquitectos |
| Pruebas | 4 | Tablas de pruebas funcionales y rendimiento |
| Conclusiones | 4 | Retomar pregunta de investigación con evidencia |
| Agradecimientos | 4 | Personal |
| Anexos | 4 | Prompts completos, ejemplo de ejecución |

#### Configuración del entorno

- **LaTeX:** MacTeX instalado via `brew install --cask mactex`
- **Editor:** VS Code + extensión LaTeX Workshop
- **Build output:** configurado a `%DIR%/build` para mantener la raíz limpia
- **Git:** `.gitignore` con `build/` para excluir artefactos de compilación
