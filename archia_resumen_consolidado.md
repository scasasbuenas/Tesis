# ArchIA — Resumen Consolidado del Trabajo del Semestre

> Documento de síntesis de las 7 presentaciones realizadas durante el desarrollo de ArchIA. Organizado cronológicamente y por línea de trabajo. Sirve como insumo para la redacción del documento final del proyecto de grado.
>
> **Universidad de los Andes — MISW4102 — 2026**

---

## Tabla de Contenidos

1. [Descripción del Proyecto](#1-descripción-del-proyecto)
2. [Estado Inicial — Reunión de Apertura](#2-estado-inicial--reunión-de-apertura)
3. [Refactor Estructural del Backend](#3-refactor-estructural-del-backend)
4. [Multi-Intent y Preservación del Lenguaje](#4-multi-intent-y-preservación-del-lenguaje)
5. [Migración del Sistema de Diagramas: Mermaid → Graphviz](#5-migración-del-sistema-de-diagramas-mermaid--graphviz)
6. [Reestructuración del RAG con Índices por Atributos de Calidad](#6-reestructuración-del-rag-con-índices-por-atributos-de-calidad)
7. [Refactor de Nodos por Atributo de Calidad (QA)](#7-refactor-de-nodos-por-atributo-de-calidad-qa)
8. [Sistema de Diagramas con Niveles de Detalle](#8-sistema-de-diagramas-con-niveles-de-detalle)
9. [Optimizaciones de Rendimiento](#9-optimizaciones-de-rendimiento)
10. [Design Ledger — Trazabilidad de Decisiones](#10-design-ledger--trazabilidad-de-decisiones)
11. [ASR Write-back y Supersession](#11-asr-write-back-y-supersession)
12. [Demostración Integral — Caso TravelHub](#12-demostración-integral--caso-travelhub)
13. [Trabajo Pendiente y Próximos Pasos](#13-trabajo-pendiente-y-próximos-pasos)
14. [Referencias Citadas en las Presentaciones](#14-referencias-citadas-en-las-presentaciones)

---

## 1. Descripción del Proyecto

**ArchIA** es un asistente inteligente de arquitectura de software basado en la metodología **ADD 3.0 (Attribute-Driven Design)** de Bass, Clements & Kazman. Guía al usuario (arquitecto de software) a través de un flujo estructurado:

1. **ASR** (Architecture Significant Requirement) — Extracción de requisitos arquitectónicamente significativos en formato canónico de 6 partes (Source, Stimulus, Environment, Artifact, Response, Response Measure).
2. **Estilos arquitectónicos** — Sugerencia de estilos candidatos según los ASR identificados.
3. **Tácticas** — Propuesta de tácticas concretas para satisfacer los atributos de calidad.
4. **Diagramas** — Generación de representaciones visuales de la arquitectura propuesta.

### Visión

> "Latinoamérica se consolida como referente en arquitectura de software gracias a ArchIA, una plataforma creada por LA COMUNIDAD ARQUITECTI para arquitectos."

ArchIA acelera el diseño de sistemas complejos, mejora la calidad de las decisiones arquitectónicas y reduce semanas de trabajo a conversaciones guiadas por inteligencia artificial.

### Stack Tecnológico (estado consolidado)

| Capa | Tecnología | Rol |
|------|------------|-----|
| **Backend** | Python + FastAPI + Uvicorn | API REST |
| **Orquestación** | LangChain + LangGraph 1.0+ | Máquina de estados con grafo de agentes |
| **LLM** | Azure OpenAI / Ollama | Multi-proveedor |
| **RAG** | ChromaDB + Embeddings | Vector store sobre documentación PDF (libros de Bass et al.) |
| **Diagramas** | Graphviz (DOT) | Renderizado SVG / DOT / draw.io XML |
| **Persistencia** | SQLite | Sesiones, feedback y *checkpointing* del grafo |
| **Frontend** | React 19 + Vite | Interfaz de chat |

---

## 2. Estado Inicial — Reunión de Apertura

*Fuente: `ARCHIA.pdf` — Reunión de Apertura del proyecto*

### 2.1. Flujo del Sistema (versión inicial)

1. El usuario escribe en el chat y opcionalmente adjunta imágenes o PDFs.
2. El frontend envía `POST /message` al backend con el texto, `session_id` y archivos.
3. El backend:
   - Guarda adjuntos (imágenes o PDFs).
   - Si hay PDF, extrae texto y lo usa como contexto.
   - Recupera memoria previa (ASR, estilo, tácticas).
   - Detecta idioma/intención y configura el flujo.
4. **LangGraph** decide qué agente ejecutar:
   - **ASR** (si aún no existe).
   - **Style** (si pide estilos).
   - **Tactics** (si pide tácticas).
   - **Diagram** (si pide diagrama).
   - **Investigator** (si hace preguntas técnicas y necesita RAG).
5. Si corresponde, el agente RAG consulta Chroma con PDFs locales y agrega fuentes.
6. El **Unifier** compone la respuesta final y sugerencias.
7. El backend devuelve `endMessage` (texto/Markdown), `mermaidCode` (si hay diagrama), `suggestions` y metadatos.
8. El frontend renderiza la respuesta y permite dar feedback (`/feedback`).

### 2.2. Grafo de Agentes (estado inicial)

- **Arquitectura:** LangGraph define una máquina de estados con `classifier` + `supervisor`/`router` que decide qué agente ejecutar.
- **Flujo base:** `START → boot → classifier → supervisor → router → agente especializado → supervisor / unifier → END`.
- **Agentes especializados:** `investigator`, `creator`, `evaluator`, `diagram_agent`, `asr`, `style`, `tactics`, `unifier`.
- **Control de flujo:**
  - El router aplica reglas de "visitar una vez".
  - Algunos intents pasan primero por `investigator` (ej. ASR + RAG).
  - `asr`/`style`/`tactics` van directo a `unifier`.
- **Persistencia:** estado del grafo y conversaciones guardadas con SQLite *checkpointing*.
- **Diagram_agent (inicial):** solo generaba código Mermaid usando el contexto; no renderizaba imágenes.

### 2.3. Diagnóstico Inicial — Fortalezas y Debilidades

**Fortalezas identificadas:**
- Bases sólidas y agentes especializados.
- Fundación del RAG implementada.
- Arquitectura LangGraph fuerte.
- Despliegue fácil.
- Stack tecnológico coherente.

**Debilidades identificadas:**
- **Alto acoplamiento → baja mantenibilidad** (todo concentrado en `graph.py`).
- Documentación deficiente.
- Respuestas no óptimas y poco específicas.
- Falta de pruebas (sin tests unitarios ni E2E).
- Lentitud de respuesta.

### 2.4. Plan Propuesto (4 semanas)

| Semana | Trabajo planeado |
|--------|------------------|
| **1** | Modularizar nodos · Extraer utilidades · Limpiar `graph.py` |
| **2** | Aislar prompts · Generar infraestructura de pruebas · Primeras pruebas |
| **3** | Pre-commit hooks · Tests E2E · Mejora de prompts · Extensión del RAG |
| **4** | Implementación de agente de reflexión · Hybrid search · Validación de calidad · Documentación |

**Soluciones propuestas en este punto:**
- Separar repositorios de front y back.
- Arquitectura modular para `graph.py` (estado centralizado, grafo limpio, agentes modulares, utilidades extraídas, prompts aislados).
- Testing (unitario y E2E).
- Calidad del código (hooks de higiene automática).
- Calidad de la respuesta:
  - Implementar agente de reflexión (auto-corrección).
  - Hybrid search para el RAG (precisión semántica y exacta).
  - Aumentar/mejorar la fuente de información del RAG.
  - Mejorar especificidad mediante *prompt engineering*.
  - Mejorar manejo del contexto mediante un nuevo nodo, *flow engineering* y *tweaks* generales.

---

## 3. Refactor Estructural del Backend

*Fuente: `ARCHIA_reunión_2.pdf` — Reunión de seguimiento*

### 3.1. Modularización de `graph.py`

Se descompuso el archivo monolítico `graph.py` en una estructura de paquete:

```
graph/
├── __init__.py
├── workflow.py         # Define la estructura del grafo
├── state.py            # Estado del grafo + validaciones de salida de cada nodo
├── consts.py           # Prompts del sistema y esquemas para tácticas/diagramas
├── utils.py            # Funciones auxiliares (tácticas y diagramas)
├── resources.py        # Recursos compartidos (LLM, RAG, persistencia del grafo)
└── nodes/              # Directorio de los nodos
```

#### Descomposición del directorio `nodes/`

| Archivo | Responsabilidad |
|---------|-----------------|
| `supervisor.py` | Orquestador principal — decide el siguiente paso según la intención del usuario |
| `classifier.py` | Determina la intención e idioma del mensaje inicial |
| `investigator.py` | Realiza búsquedas en el RAG sobre conceptos arquitectónicos |
| `asr.py` | Genera y propone ASRs |
| `style.py` | Sugiere los estilos arquitectónicos más adecuados para cumplir los ASR |
| `tactics.py` | Propone tácticas de diseño específicas para satisfacer los atributos de calidad |
| `diagram.py` | Compila ASR + estilo + tácticas + contexto adicional para generar diagramas |
| `evaluator.py` | Analiza y critica las propuestas arquitectónicas, identifica riesgos o mejoras |
| `unifier.py` | Consolida todas las respuestas de los nodos previos en un mensaje final coherente |
| `tools.py` | Herramientas adicionales que los nodos pueden invocar durante su ejecución |

**Beneficios obtenidos:**
- **Mantenibilidad:** código distribuido en módulos pequeños — más fácil localizar, entender y modificar cada responsabilidad.
- **Escalabilidad funcional:** agregar o reemplazar nodos es tan simple como añadir un archivo y conectarlo en `workflow.py`.
- **Pruebas y depuración:** cada nodo y *helper* puede probarse aisladamente.

### 3.2. Migración `SqliteSaver` → `MemorySaver`

**Problema detectado:**
- `SqliteSaver` retornaba un *context manager*, no una instancia directa.
- El grafo se compilaba fuera de un `with`, generando errores como `AttributeError: __enter__` y conexiones cerradas.
- SQLite (basado en archivo) generaba *locks* en entorno asíncrono (FastAPI/Uvicorn).

**Solución implementada:**
- Reemplazo por `MemorySaver`.
- Sin *context management*.
- Sin archivos físicos.
- Compatible directamente con `graph.compile()`.

**Lo que se ganó con el refactor:**
- **Arquitectura limpia:** *Single Responsibility*, fuente única de verdad para el estado, separación lógica vs. *wiring*.
- **Escalabilidad para ADD 3.0:** fácil agregar nuevos nodos, fácil extender el supervisor, prompts desacoplados de la lógica.
- **Reducción de riesgos:** sin más imports circulares, sin fallos por `.env`, *checkpointing* estable.

> **Nota:** En presentaciones posteriores se observa que el *checkpointing* persistente se restituye con `sqlite_saver` para soportar el Design Ledger. La migración a `MemorySaver` resolvió la ruptura inmediata del flujo de compilación.

---

## 4. Multi-Intent y Preservación del Lenguaje

*Fuente: `ARCHIA_reunión_2.pdf`*

### 4.1. Problemas identificados

**Antes del cambio:**
- Las respuestas con varios requisitos (ASR, estilos, tácticas o diagramas en un solo *prompt*) se cortaban o generaban respuestas incorrectas.
- Se generaban diagramas cuando no se pedían.
- A veces se retornaba el mismo *prompt* (efecto eco).
- El lenguaje podía cambiar durante la ejecución (mezcla ES/EN).

### 4.2. Solución multi-intent

- Se añadieron campos al `state`: `requested`, `completed` y `pending`.
- Se añadió un **Scheduler multi-intent** al supervisor con *helpers* nuevos: `infer_requested`, `augment_completed`, `append_unique`.
- Los nodos van primero al supervisor antes que al unifier.
- Se modificó el *unifier* para mostrar las respuestas completas.

### 4.3. Solución diagrama (evitar generación no solicitada)

- Se cambió la detección del *intent*, forzándolo a usar palabras clave que denoten la necesidad del diagrama: `"diagrama"`, `"graphviz"`, `"visualización"`, etc.

### 4.4. Solución eco

- Se cambió el *scheduler* para que, si un ASR era pedido y aún no se había generado, se priorizara este.

### 4.5. Solución lenguaje

- El lenguaje de respuesta usado en todo agente especializado se toma de `state[language]`.

### 4.6. Estado al cierre de la reunión

| Categoría | Estado |
|-----------|--------|
| **Solucionado** | Soporte multi-intent · Diagramas generados solo cuando se solicitan · Preservación del lenguaje |
| **Por mejorar** | Calidad de respuesta (poco específica al contexto) · Tiempo de respuesta (hasta 350 s en ciertos *prompts*) · Formato de respuesta (difícil de leer y seguir) |

Se introdujo **LangSmith** como herramienta de tracing para diagnóstico.

---

## 5. Migración del Sistema de Diagramas: Mermaid → Graphviz

*Fuente: `ARCHIA_reunión_2.pdf`*

### 5.1. Problemas con la implementación Mermaid

- **Renderizado en el frontend:** si fallaba la sintaxis o la librería JS, el usuario no veía nada.
- **Sin validación desde backend:** no había control de errores antes de enviar al cliente.
- **Diagrama generado solo con la pregunta actual:** ignoraba ASR, estilos, tácticas y contexto acumulado.

### 5.2. Solución con Graphviz

- **Pipeline completo *server-side*:** generación DOT → sanitización → renderizado SVG.
- **Diagrama basado en contexto arquitectónico completo:** ASR + estilo + tácticas + memoria + RAG.
- **Validación en backend:** errores capturados antes de enviar al cliente.
- **Salida portable:** SVG sin dependencias JS en el frontend.

---

## 6. Reestructuración del RAG con Índices por Atributos de Calidad

*Fuente: `ArchIA_RAG_Indexado.pdf` (28 de febrero de 2026) y `ArchIA_Sistema_Diagramas____Repaired.pdf`*

### 6.1. El Problema: RAG sin Diferenciación Temática

**Sistema anterior:**
- Una sola colección Chroma sin índices.
- *Retrieval* disperso entre `asr.py`, `tactics.py`, `investigator.py`, `evaluator.py`.
- *Queries* genéricos sin contexto temático.
- Sin trazabilidad: la misma pregunta retornaba el mismo *pool* de documentos.

**Consecuencia clave:** *"¿Qué tácticas de escalabilidad?"* y *"¿Cómo reduzco latencia?"* obtenían los **mismos** documentos. *Retrieval* poco enfocado y respuestas menos precisas para tácticas específicas.

### 6.2. Objetivos de la Reestructuración

1. **Resolución automática de índice:** detectar automáticamente qué atributo de calidad (QA) aplica a la pregunta del usuario mediante inferencia LLM.
2. **Recuperación diferenciada por tipo:** para cada índice QA, recuperar contenido diferenciado (ASR, Estilos, Tácticas).
3. **Extensibilidad sin cambios de código:** añadir un nuevo atributo de calidad solo requiere editar `indices.json` y reconstruir el vectorstore.

### 6.3. Decisiones de Diseño

| Decisión | Elección | Justificación |
|----------|----------|---------------|
| Resolución de índice | LLM inference (no *keywords*) | Mayor precisión con preguntas complejas, soporte bilingüe (ES/EN), coherente con el stack existente. *Fallback* a `'general'` ante errores. |
| Fuente de documentos | Re-etiquetar PDFs existentes | Aprovecha el corpus disponible (*Software Architecture in Practice*, *Evaluating Software Architecture*). El *auto-tagging* por *keywords* ocurre en la fase de ingesta. |
| Estrategia Chroma | Una colección + filtros de metadata | No rompe el código existente. Cambio aditivo: se añaden campos `quality_attribute` y `content_type` a cada *chunk*. Compatible con el `_LazyRetriever` global. |

### 6.4. Arquitectura del Flujo Nuevo

```
Usuario: "¿Qué tácticas de escalabilidad debería usar?"
    ↓
boot_node → classifier_node → resolve_quality_attribute() [LLM]
    ↓                              ↓
intent='tactics'            state["resolved_index"] = "escalabilidad"  [NUEVO]
force_rag=true                     ↓
    ↓                       supervisor_node → tactics_node
                                   ↓
                            get_indexed_retriever()
                              filtra por QA + tipo contenido
                              k=6 chunks relevantes
                              (quality_attribute='escalabilidad',
                               content_type='tacticas')
                                   ↓
                            unifier_node → Respuesta + fuentes indexadas
```

### 6.5. Componentes Nuevos y Modificados

**Archivos nuevos:**
- `back/config/indices.json` — configuración declarativa de QAs y tipos de contenido.
- `back/src/graph/index_resolver.py` — módulo de resolución LLM. Expone `resolve_quality_attribute()` y `get_available_indices()`.

**Archivos modificados:**
- `build_vectorstore.py` — *auto-tagging* de *chunks* con `quality_attribute` + `content_type`.
- `rag_agent.py` — nueva función `get_indexed_retriever(qa, type, k)`.
- `graph/state.py` — campo `resolved_index: str` en `GraphState`.
- `nodes/classifier.py` — llama `resolve_quality_attribute()`, incluye en `return`.
- `nodes/asr.py` — usa `get_indexed_retriever(resolved_index, 'asr', 6)`.
- `nodes/tactics.py` — usa `get_indexed_retriever(resolved_index, 'tacticas', 6)`.

### 6.6. Auto-tagging de Chunks

**Metadata añadida a cada *chunk*:**
- `quality_attribute`: `"escalabilidad"`, `"latencia"`, `"general"`.
- `content_type`: `"asr"`, `"estilos"`, `"tacticas"`, `"general"`.

**Algoritmo `_tag_chunk(text, config)`:**
1. Tokenizar texto a *lowercase*.
2. Para cada QA en `indices.json`: contar *matches* de `keywords_en` + `keywords_es`.
3. QA con más *matches* → `quality_attribute` (si ninguno → `'general'`).
4. Para cada `content_type`: contar *matches* de *keywords*.
5. Tipo con más *matches* → `content_type` (si ninguno → `'general'`).

### 6.7. Distribución Inicial de Tags (658 chunks)

| `quality_attribute` | `content_type` | Chunks | % total |
|---------------------|----------------|-------:|--------:|
| general             | asr            | 288    | 43.8%   |
| general             | general        | 130    | 19.8%   |
| latencia            | asr            | 57     | 8.7%    |
| escalabilidad       | asr            | 46     | 7.0%    |
| general             | estilos        | 31     | 4.7%    |
| latencia            | tacticas       | 27     | 4.1%    |
| general             | tacticas       | 18     | 2.7%    |
| latencia            | general        | 18     | 2.7%    |
| escalabilidad       | general        | 16     | 2.4%    |
| escalabilidad       | tacticas       | 12     | 1.8%    |
| latencia            | estilos        | 8      | 1.2%    |
| escalabilidad       | estilos        | 7      | 1.1%    |
| **TOTAL**           |                | **658**| **100%**|

### 6.8. Optimización de Keywords (sesión 28 feb 2026)

**Problema identificado tras el primer *rebuild*:** solo 9 de 658 *chunks* (1.4%) fueron etiquetados como `content_type='tacticas'`.

**Causa raíz:** en libros de arquitectura, las tácticas se explican dentro de escenarios QAS → las *keywords* ASR ganaban el conteo. Además, `'environment'` y `'scenario'` en la lista ASR eran demasiado genéricos.

**Cambio 1 — Eliminar *keywords* genéricas de ASR:**
- Se eliminaron `"environment"` y `"scenario"` de la lista de *keywords* ASR.
- Justificación: aparecen en *"cloud environment"*, *"runtime environment"* — contaminan *chunks* de tácticas. Los *chunks* genuinamente ASR siguen capturándose por `'stimulus'`, `'response measure'`, `'QAS'`, etc.

**Cambio 2 — Ampliar *keywords* de Tácticas:**
- De 24 a 57 términos (+33 nuevos), organizados por categoría:
  - **Performance/recursos:** *manage resource demand, schedule resources, introduce concurrency*…
  - **Disponibilidad:** *detect faults, active redundancy, graceful degradation, rollback*…
  - **Modificabilidad:** *encapsulate, use an intermediary, defer binding*…
  - **Infraestructura:** *queue, cache, thread pool, pipeline, redundancy*…

**Resultados antes vs. después:**

| `content_type` | Antes | Después | Cambio  |
|----------------|------:|--------:|---------|
| tacticas       | 9     | 57      | +533% ↑ |
| asr            | 450   | 391     | -13% ↓  |
| estilos        | 45    | 46      | +2% →   |
| general        | 151   | 164     | +9% →   |

**Impacto en `tactics_node`:**
- `latencia/tacticas`: 6 → 27 chunks.
- `escalabilidad/tacticas`: 3 → 12 chunks.
- → Contexto real disponible para el LLM.

### 6.9. Bugs Identificados y Corregidos

**BUG 1 — CRÍTICO — `index_resolver.py`: ruta al config incorrecta.**
- **Síntoma:** el resolver retornaba `'general'` para **todas** las preguntas. El RAG indexado era completamente inoperante.
- **Causa:** `_CONFIG_PATH` tenía 4 saltos de directorio (`.parent ×4`) en lugar de 3 → archivo no encontrado → `qa_list` vacía → *fallback* `'general'`.
- **Antes:** `...parent.parent.parent.parent / 'config' / 'indices.json'` ❌
- **Después:** `...parent.parent.parent / 'config' / 'indices.json'` ✓

**BUG 2 — `build_vectorstore.py`: sin limpieza antes del rebuild.**
- **Síntoma:** el vectorstore tenía 1316 *chunks* en lugar de 658 — exactamente el doble. `Chroma.from_documents()` **añade** en lugar de reemplazar.
- **Consecuencia:** la misma búsqueda recuperaba documentos duplicados, reduciendo la diversidad del contexto y desperdiciando *tokens*.
- **Fix:** `shutil.rmtree(PERSIST_DIR)` al inicio de `main()` antes de crear el vectorstore.

### 6.10. Verificación End-to-End

**Parte 1 — Metadatos Chroma:**
- Total *chunks*: 658.
- *Chunks* con `quality_attribute`: 658/658 ✓
- *Chunks* con `content_type`: 658/658 ✓

**Parte 2 — Filtros del Retriever (todos verificados ✓):**

| Filtro aplicado | Resultado |
|-----------------|-----------|
| Sin filtros | 4 docs de cualquier tipo |
| `quality_attribute=latencia` | Solo *chunks* `latencia/*` |
| `quality_attribute=escalabilidad` | Solo *chunks* `escalabilidad/*` |
| `content_type=tacticas` | Solo *chunks* `*/tacticas` |
| `latencia + tacticas` | Solo `latencia/tacticas` |
| `escalabilidad + tacticas` | Solo `escalabilidad/tacticas` |
| `latencia + asr` | Solo `latencia/asr` |

**Parte 3 — Index Resolver (LLM): 5/5 *test cases* pasados.**
- ✓ `escalabilidad` ← *"¿Sistema escalable?"*
- ✓ `latencia` ← *"Reducir latencia API <200ms"*
- ✓ `general` ← *"¿Qué es ADD 3.0?"*
- ✓ `latencia` ← *"What tactics reduce response time?"*
- ✓ `escalabilidad` ← *"How do I scale horizontally?"*

### 6.11. Extensibilidad

Para añadir un nuevo atributo (ej. `'disponibilidad'`):

1. Editar `back/config/indices.json` (añadir `id`, `display_name`, `keywords_en`, `keywords_es`, `description`).
2. Reconstruir el vectorstore: `cd /back && python build_vectorstore.py`.
3. Listo, sin cambios de código. El LLM resolver detecta el nuevo índice automáticamente gracias a la configuración declarativa.

### 6.12. Estado Final del RAG Indexado

- ✓ Implementado — 28 feb 2026.
- RAG indexado con resolución LLM del atributo de calidad activo en producción.
- Vectorstore con 658 *chunks* etiquetados: `tacticas` 8.7% · `asr` 59.4% · `estilos` 7.0% · `general` 24.9%.
- Todos los filtros de metadata verificados · 5/5 *test cases* del LLM resolver pasados · 0 bugs pendientes.

---

## 7. Refactor de Nodos por Atributo de Calidad (QA)

*Fuente: `ArchIA_RAG_Indexado.pdf` y `ArchIA_Sistema_Diagramas____Repaired.pdf`*

### 7.1. Motivación

Una vez disponible la dimensión QA en el estado, los nodos `style.py` y `tactics.py` monolíticos se reorganizaron en paquetes por dominio para permitir lógica diferenciada por atributo de calidad sin condicionales dispersos.

### 7.2. Nueva Estructura de Nodos

```
nodes/styles/
├── style.py                  # Nodo general + factoría make_style_qa_node
├── latency_style.py          # style_latency_node
├── scalability_style.py      # style_scalability_node
└── common.py                 # style_node_impl + resolución QA

nodes/tactics/
├── tactics.py                # Nodo general + factoría make_tactics_qa_node
├── latency_tactics.py        # tactics_latency_node
├── scalability_tactics.py    # tactics_scalability_node
└── common.py                 # tactics_node_impl + resolución QA
```

**Eliminados:** `nodes/style.py` · `nodes/tactics.py`.

### 7.3. Cambios en el Clasificador y el Estado

- `ClassifyOut` ahora incluye `quality_attribute: str`.
- El *prompt* del clasificador pide QA junto a `intent` y `language`.
- QA normalizado y guardado en el estado.
- `resolved_index` prioriza el QA clasificado.

### 7.4. Registro Central — `qa_registry.py` [NUEVO]

Único punto de verdad para normalización y *naming*:

- `supported_qas()`
- `normalize_qa(value)`
- `style_node_name_for_qa(qa)`
- `tactics_node_name_for_qa(qa)`
- `qa_to_focus_label(qa, default)`

### 7.5. Enrutamiento Dinámico

```
classifier_node → router() → workflow.py (dinámico)
                                  ↓
                        ┌─────────┴──────────┐
                        ↓                    ↓
              style_latency_node    style_scalability_node
              tactics_latency_node  tactics_scalability_node
              ...
              Fallback → style / tactics (general)
```

- Nodos registrados dinámicamente desde `supported_qas()`.
- *Edges* de retorno → supervisor también dinámicos.

### 7.6. Comportamiento Resultante

- El *classifier* define `intent` + QA en una sola pasada.
- El *router* traduce `nextNode` lógico a nodo QA-específico.
- QA desconocido → nodo general (sin romper flujo).

### 7.7. Aprendizaje Clave

> Cuando una dimensión de negocio (como QA) impacta múltiples nodos, *prompts* y rutas, conviene modelarla como entidad de primer nivel — en estado + registro central — en lugar de resolverla con condicionales dispersos. Esto reduce acoplamiento y simplifica futuras implementaciones.

---

## 8. Sistema de Diagramas con Niveles de Detalle

*Fuente: `ArchIA_Sistema_Diagramas____Repaired.pdf`*

### 8.1. Niveles de Detalle

| Nivel | Nombre | Descripción |
|-------|--------|-------------|
| **1** | **Overview (default)** | Vista de alto nivel por defecto. 5–15 nodos máximo: subsistemas, capas y servicios mayores. Los *clusters* se colapsan en un solo nodo. |
| **2** | **Detailed (bajo demanda)** | Diagrama completo con todos los nodos y *edges*. Se activa con palabras clave: *"detallado"*, *"full"*, *"expand"*. |
| **3** | **Focused (expansión focalizada)** | Expande un grupo específico del *overview*. Muestra nodos internos + conexiones externas marcadas `[ext]`. *Endpoint:* `detail_level=detailed&focus=grp_backend`. |

### 8.2. Flujo de Generación (request → archivo)

| Paso | Acción |
|------|--------|
| 1. Request | `POST /message` detecta *keywords* `diagrama`, `diagram`, `componentes`. |
| 2–4. Orquestación + LLM + IR | `diagram_orchestrator_node()` decide nivel → LLM genera DOT (`DOT_SYSTEM` o `DOT_SYSTEM_OVERVIEW`) → `parse_dot_to_model()` convierte a `DiagramModel` (IR). |
| 5. Simplificación | Si *overview* y >20 nodos: colapso programático. Retorna `(overview_model, mapping)`. |
| 6. Rendering | IR → DOT final + SVG base64 vía `diagram_render.py`. |
| 7. Almacenamiento | `state[diagram]` guarda `svg_b64`, `dot`, `overview_mapping` en SQLite. |
| 8. Exportación | `GET /diagram/export` re-aplica nivel y devuelve SVG, DOT o draw.io. |

### 8.3. Componentes Clave

**Archivos nuevos:**
- `diagram_ir.py` — IR: `DiagramModel`, `parse_dot_to_model()`, `build_overview()`, `build_expanded_view()`.
- `diagram.py` (node) — Nodo LangGraph: decide nivel, llama LLM, parsea, colapsa, renderiza y guarda en estado.

**Archivos modificados:**
- `consts.py` — *prompts* del sistema: `DOT_SYSTEM` (detailed) y `DOT_SYSTEM_OVERVIEW` (max 15 nodos).
- `main.py` — *endpoints* REST: `POST /message` y `GET /diagram/export` (línea 287).
- `state.py` — *schema* del estado: campo `diagram` almacena metadata + `overview_mapping`.
- `memory.py` — persistencia: `save_arch_flow()` / `load_arch_flow()` sobre SQLite (`state_db/memory.db`).
- `workflow.py` — compilación del grafo con *checkpointer*: `graph = builder.compile(checkpointer=sqlite_saver)`.

### 8.4. Decisiones de Diseño

- **IR separada del *rendering*:** `DiagramModel` como capa intermedia entre el DOT del LLM y el *output* final.
- **Simplificación programática:** colapso automático si >20 nodos, sin re-promptear al LLM.
- ***Mapping*** *overview ↔ detailed* persistido en sesión: garantiza coherencia Level 1 → Level 2.
- **`session_id` como identificador único:** sin `diagram_id`, `conversation_id` ni `request_id`.

> **Pipeline en producción posterior (visto en demo):** LLM → DOT → IR → Render. *Pipeline* paralelo: `render_svg_b64()` (Graphviz) + `render_dot()` (DOT limpio) + `render_dot_drawio()` (XML mxGraph). Regla de invalidación: si se regenera el nivel 1, se borran los niveles 2 y 3 automáticamente.

---

## 9. Optimizaciones de Rendimiento

*Fuente: `archia_optimizaciones_v2.pdf`*

### 9.1. Diagnóstico — Tiempos de Respuesta

| Tipo de *prompt* | Latencia observada |
|-------------------|-------------------:|
| *Prompts* simples (ASRs, *greetings*) | ~80 s |
| *Prompts* complejos (estilos basados en ASRs) | ~120 s |
| *Prompts* multi-intent (ASR + estilos + tácticas + diagramas en uno solo) | 300–360 s |

### 9.2. Bottlenecks Identificados (por prioridad)

| Prioridad | *Bottleneck* | Impacto estimado | Descripción | Dificultad | Estado |
|:--:|---|---|---|:--:|:--:|
| **P1** | LLM calls secuenciales | 60–70 % de la latencia total | *Calls* paralelizables — 16–30 s evitables | Media | Pendiente |
| **P2** | Sin *streaming* al cliente | Latencia percibida | `graph.invoke()` bloquea hasta `END`; sin *feedback* al usuario | Baja | Pendiente |
| **P3** | RAG queries secuenciales | 15–20 % de la latencia | 4–6 búsquedas vectoriales en *loop*, ~200–800 ms cada una | Baja | **Implementado ✓** |
| **P4** | Sin caché de ningún tipo | 40–60 % cómputo redundante | Todo se re-ejecuta íntegro para el mismo *prompt* o QA | Media | **Implementado ✓** |
| **P5** | Graphviz bloqueante | 5–15 s en diagramas | `subprocess.run()` síncrono en *handler* async — bloquea *event loop* | Baja | **Implementado ✓** |
| **P6** | Un solo *worker* uvicorn | Bloqueo total de concurrencia | Segunda petición queda en cola 60–80 s durante respuesta en curso | Muy baja | Pendiente |
| **P7** | State/prompt sin límite | Empeora progresivamente | `memory_text` crece sin truncar; latencia aumenta en sesiones largas | Baja | Pendiente |

### 9.3. Diagnóstico Detallado de los Bottlenecks Atacados

#### P3 — RAG: 12–16 búsquedas en serie

**Código original (`tactics/common.py`):**
```python
# 4+ queries en loop bloqueante
for q in queries:
    for d in _retriever.invoke(q):
        seen.add(d.metadata["source"])
        snippets.append(d.page_content)
```

El mismo patrón existía en `styles/common.py`. En el *prompt* de TravelHub (escalabilidad) se disparaban 12–16 búsquedas vectoriales en total entre ambos nodos.

**Por qué era un problema:**
1. Las *queries* son completamente independientes — sin dependencia de datos.
2. ChromaDB soporta concurrencia — el *lock* es a nivel de lectura, no exclusivo.
3. Costo lineal acumulativo (TravelHub: 4 *queries* × 2 nodos × 400 ms ≈ 3.2 s solo en RAG).
4. Nada en el código justificaba el *loop* secuencial — era simplemente la forma más directa de escribirlo.

#### P4 — Sin caché: todo se re-ejecuta íntegro en cada turno

| Operación re-ejecutada | Costo |
|------------------------|------:|
| Clasificación del mensaje (`temperature=0.0`, determinística) | ~2–4 s |
| Resolución del QA (`resolve_quality_attribute()`, determinística) | ~1–2 s |
| Búsquedas RAG (estilos) — índices estáticos en sesión | ~0.8–1.5 s |
| Búsquedas RAG (tácticas) — mismo caso | ~0.8–1.5 s |

**Oportunidad adicional — *Prompt Caching* del proveedor:**
- OpenAI aplica caché automático sobre el prefijo del *prompt* cuando supera 1024 *tokens*.
- **Problema en ArchIA:** los *prompts* de `styles` y `tactics` inyectaban contexto variable (ASR, *snippets* RAG, historial) en el medio del *string* — no al final. Esto destruía el prefijo cacheado en cada llamada.
- **Solución:** separar instrucciones estáticas (`SystemMessage`) del contenido dinámico (`HumanMessage`). El prefijo del sistema queda fijo y el proveedor lo cachea automáticamente.

#### P5 — Graphviz: proceso externo bloqueante en servidor async

**Código original (`diagram_render.py`):**
```python
def render_svg(dot_string, engine="dot"):
    proc = subprocess.run(
        [engine, "-Tsvg"],
        input=dot_string.encode(),
        capture_output=True,
    )
    return proc.stdout  # bloquea 5–15 s
```

**Tres problemas independientes:**
1. **Bloquea el *event loop* de FastAPI:** `subprocess.run()` es sincrónico. Llamado desde un *endpoint* `async def`, detiene el *loop* de `asyncio` durante toda la ejecución del proceso. Ninguna otra corrutina puede avanzar.
2. **Sin caché:** Graphviz es determinístico — mismo string DOT + mismo motor → exactamente el mismo SVG. No cachear significa invocar el binario externo cada vez.
3. **Diagramas complejos amplifican el costo:** para TravelHub (nivel 2–3, 15+ proveedores PMS) el *render* puede tardar 5–15 s.

### 9.4. Implementación de Soluciones

#### Commit 1 — Solución P3: Paralelización RAG con `ThreadPoolExecutor`

**Por qué `ThreadPoolExecutor` y no `asyncio.gather`:** `_retriever.invoke()` de LangChain es **síncrono**. `asyncio.gather` solo paraleliza corrutinas async — usarlo con una función síncrona no ofrece concurrencia real. `ThreadPoolExecutor` corre cada invocación en un hilo separado del OS, logrando verdadero paralelismo I/O para las *queries* al *vectorstore*.

```python
# DESPUÉS — ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=len(queries)) as executor:
    futures = {
        executor.submit(_retriever.invoke, q): q
        for q in queries
    }
    for f in as_completed(futures):
        for d in f.result():
            ...
```

**Resultado:** todas las *queries* en paralelo ≈ 400–500 ms por nodo. Reducción estimada: −60–70 % del tiempo RAG.

#### Commit 2 — Solución P4a: Caché LRU

**Principio de diseño:** solo se cachean funciones determinísticas (`temperature=0`) cuyos *inputs* son *strings* serializables y cuyos *outputs* no dependen de estado externo mutable.

| Archivo | Función | `maxsize` | Justificación de seguridad |
|---------|---------|----------:|---------------------------|
| `classifier.py` | `_classify_cached(msg, qa_opts_str) → tuple` | 256 | `temperature=0`; lista de QAs serializada en clave invalida automáticamente al cambiar opciones. |
| `classifier.py` | `_resolve_qa_cached(msg) → str` | 128 | Solo depende del texto del mensaje; sin estado externo. |
| `styles/common.py` | `_fetch_styles_rag(qa, resolved_index, k=6) → str` | 64 | Índices RAG estáticos en sesión; clave `(qa, resolved_index, k)` captura todos los factores. |
| `tactics/common.py` | `_fetch_tactics_rag(qa, resolved_index, k=6) → tuple` | 64 | Mismo razonamiento que estilos. Retorna `(book_snippets, src_meta)` — ambos deterministas. |

#### Commit 3 — Solución P4b: Separación System / Human Message (provider caching)

```python
# DESPUÉS — System / Human separados
_STYLES_SYSTEM = (
    "You are a software architect"
    " applying ADD 3.0..."
    # instrucciones estáticas → candidato a cacheo
)
human = f"{directive}\n{qa}\n{rag_snippets}"
result = llm.invoke([
    SystemMessage(_STYLES_SYSTEM),
    HumanMessage(human),
])
```

**Beneficios:** `SystemMessage` idéntico entre llamadas → el proveedor lo cachea. Además, `_STYLES_SYSTEM` como constante mejora la mantenibilidad.

#### Commit 4 — Solución P5: `render_svg_async` + `_SVG_CACHE`

```python
async def render_svg_async(dot_string: str, engine: str = "dot") -> bytes:
    key = hash((dot_string, engine))
    if key in _SVG_CACHE:
        return _SVG_CACHE[key]
    proc = await asyncio.create_subprocess_exec(
        engine, "-Tsvg",
        stdin=asyncio.subprocess.PIPE,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    stdout, _ = await asyncio.wait_for(
        proc.communicate(input=dot_bytes),
        timeout=30,
    )
    _SVG_CACHE[key] = stdout
    return stdout
```

**Por qué `asyncio.create_subprocess_exec()`:** retorna inmediatamente una corrutina. El *event loop* puede seguir procesando otras *requests* mientras espera que `dot` termine. El *endpoint* `diagram_export` se migró de `def` a `async def` para usar `await`.

**Por qué `_SVG_CACHE` es seguro sin TTL:** Graphviz es puramente funcional — `dot_string + engine → SVG` sin estado externo. La clave `hash((dot_string, engine))` cubre todos los factores que determinan el *output*.

**Deuda asumida:** `_SVG_CACHE` es *in-memory* por *worker*. No persiste entre reinicios. Con múltiples *workers* (Gunicorn) cada proceso calienta su propio caché de forma independiente.

### 9.5. Resumen del Impacto de las Optimizaciones

| Solución | Problema atacado | Archivos clave | Reducción estimada |
|----------|------------------|----------------|--------------------|
| RAG paralelo (`ThreadPoolExecutor`) | P3 — búsquedas vectoriales secuenciales | `styles/common.py`, `tactics/common.py` | −60–70 % tiempo RAG por nodo |
| Caché LRU (4 funciones) | P4 — re-ejecución íntegra en cada turno | `classifier.py`, `styles/common.py`, `tactics/common.py` | −40–60 % cómputo en *queries* repetidas |
| `System` / `Human` message separation | P4b — *prompt caching* del proveedor LLM | `styles/common.py`, `tactics/common.py` | Habilita prefijo cacheable en API |
| `render_svg_async` + `_SVG_CACHE` | P5 — Graphviz bloqueante sin caché | `diagram_render.py`, `main.py` | −5–15 s *renders* repetidos · *event loop* libre |

---

## 10. Design Ledger — Trazabilidad de Decisiones

*Fuente: `archia_presentation_v3.pdf`*

### 10.1. Contexto: el problema que había que resolver primero

**Antes:** `arch_flow` era un *blob* JSON plano en SQLite. Cada decisión sobreescribía la anterior. Sin trazabilidad entre ASR, estilo y tácticas.

**Después:** **Design Ledger** — árbol de decisiones inmutables. Cada decisión referencia a la que la originó. Cadena trazable y verificable.

### 10.2. La Cadena de Decisiones

```
ASR (Quality Attribute from ledger)
    ↓ Bound to active ASR
Estilo
    ↓ traces_to asr
Táctica
    ↓ Parents: style_ref + tactic_ref
Diagrama
```

> La coherencia no depende de que el LLM recuerde el hilo — está forzada estructuralmente.

### 10.3. Problema 1: La IA no recordaba el hilo (Fases 1, 2 y 4 de ADD 3.0)

**El síntoma:**
- El arquitecto definía un ASR de latencia y pedía tácticas.
- El sistema proponía soluciones para disponibilidad o escalabilidad.
- Las decisiones no tenían ninguna relación entre sí.

**La raíz del problema:**
- Toda la memoria de una sesión vivía en `arch_flow`, un *blob* JSON plano en SQLite.
- Cada nueva decisión sobreescribía la anterior.
- No había vínculo estructural entre el ASR, el estilo que lo satisfacía y las tácticas derivadas.
- El `quality_attribute` se calculaba por heurística en cada turno, no desde el ASR aprobado.

### 10.4. La Solución en Tres Capas

**1. Design Ledger:**
- Módulo `ledger/` con `types`, `validate`, `store`, `render`.
- *Endpoints* `/dossier`, `/ledger`, `/phase` — el *ledger* es consultable.

**2. State Hydration:**
- `context_loader` recarga el *ledger* al inicio de cada turno.
- `_mirror_legacy` traduce el *ledger* activo a los *scalars* que los nodos necesitan.

**3. Dossier Binding:**
- `ACTIVE ASR BINDING` inyectado en *prompts* de estilo y tácticas.
- `traces_to_asr` obligatorio en cada táctica — la validación rechaza si está vacío.

### 10.5. Resultado para el Arquitecto

- **Cadena ASR → Estilo → Táctica → Diagrama trazable:** la coherencia no depende de que el LLM recuerde el hilo — está forzada estructuralmente. Cada decisión persiste con referencia explícita a la que la originó.
- **Continuidad entre turnos garantizada:** el `quality_attribute` del turno actual viene del ASR que el arquitecto aprobó, no de una heurística. Si el *ledger* está vacío, los *scalars* existentes no se tocan — sin regresión en sesiones nuevas.
- ***Dossier* consultable:** `GET /sessions/{id}/dossier` devuelve el estado completo del diseño en Markdown en cualquier momento.

---

## 11. ASR Write-back y Supersession

*Fuente: `archia_presentation_v3.pdf`*

### 11.1. Problema 2: La IA entraba en loop con los ASRs (Fase 3 de ADD 3.0)

**El síntoma:**
- El arquitecto pedía refinar el ASR y el sistema volvía a proponer exactamente lo mismo.
- El loop podía repetirse múltiples veces en la misma sesión sin progresar.

**La raíz del problema:**
- No había memoria de qué ASRs se habían generado y reemplazado.
- Cada vez que el arquitecto pedía uno nuevo, el nodo arrancaba desde cero.
- El ASR anterior simplemente se sobreescribía — sin rastro de que había existido.

### 11.2. El Mecanismo de Supersession

- `append_decision(kind="asr")` se llama después de cada generación.
- ASR anterior → `status: superseded` con referencia al nuevo.
- El historial es **inmutable y monotónico**.

### 11.3. El Prompt — `PRIOR ASR HISTORY`

- Inyectado en el *prompt* antes de llamar al LLM.
- Máx. 1500 caracteres — sin inflar *tokens*.
- Ausente en sesiones nuevas — el *prompt* es bit-idéntico al anterior.

### 11.4. Resultado para el Arquitecto

- **No repite opciones descartadas:** el arquitecto rechaza un ASR y el sistema no lo vuelve a proponer. La IA recibe visibilidad explícita del historial en cada generación nueva.
- **Garantía de funcionalidad base:** los *scalars* `current_asr` y `quality_attribute` se escriben incondicionalmente. Incluso si `append_decision` falla, la herramienta sigue funcionando.
- **Historial consultable:** `GET /sessions/{id}/ledger` devuelve todas las decisiones — activas, *supersededas* y rechazadas.

---

## 12. Demostración Integral — Caso TravelHub

*Fuentes: `archia_presentation_v3.pdf` y `ArchIA_Demo_Presentacion_2.pdf`*

### 12.1. Contexto del Sistema TravelHub

Plataforma de reservas hoteleras en 6 países de LATAM (Colombia, Perú, Ecuador, México, Chile, Argentina).

| Métrica | Valor |
|---------|-------|
| Reservas/mes | ~18.000 |
| Ingresos anuales | $51.6M |
| Hoteles asociados | 1.200 |
| Viajeros activos | 450.000 |
| Sistema construido hace | 5 años |
| Base de datos | Centralizada en un único servidor |

**Procesos críticos con problemas de rendimiento:**

| Proceso | Tiempo actual |
|---------|--------------:|
| Búsqueda de hospedaje | 3–5 s |
| Detalle de propiedad | 1.2–1.5 s |
| Historial de reservas | 6–8 s |
| Crear reserva | 2–3 s |
| Procesar pago | 4–6 s |

**Impacto en negocio:**
- 25–30 % de usuarios abandonan antes de completar la reserva.
- 85 % de CPU en temporada alta · 35–40 % de rechazos.
- Disponibilidad 98.5 % *uptime*, una sola región AWS, RTO > 4 horas.
- Escalabilidad: 150 TPM → colapso a 400+ TPM. Meta: ≤800 ms (p95) y soporte de 800 TPM en pico.

### 12.2. Flujo Demostrativo (10 *prompts*)

#### Prompt 1 — Saludo y Contexto Inicial
*"Hola, soy arquitecto de software y voy a trabajar en el rediseño de TravelHub..."*

| Etapa | Acción |
|-------|--------|
| Classifier | Detecta `intent='greeting'`. Index Resolver clasifica contra `indices.json` pero no encuentra QA específico → `resolved_index='general'`. Sin filtro RAG. |
| Supervisor | No hay ASR ni diagrama pendiente. Decide activar Investigator para registrar el contexto de dominio TravelHub en el *state* (`add_context`). |
| Investigator | Guard: `intent='greeting'` → no ejecuta RAG. Registra contexto del dominio (6 países LATAM, 3 QAs problemáticos). |
| Unifier | Modo *output*: Greeting. Genera bienvenida personalizada. Sugerencias: *Generar ASR de latencia*, *Explorar estilos*, *Crear diagrama*. |

#### Prompt 2 — ASR de Latencia
*"Genera un ASR de latencia para el módulo de búsqueda de hospedaje de TravelHub..."*

| Etapa | Acción |
|-------|--------|
| Classifier | `intent='asr'`. Index Resolver → `resolved_index='latencia'`. *Keywords match*: `'milisegundos'`, `'p95'`, `'respuesta'`. `force_rag=true`. |
| Supervisor | `nextNode='asr'`. Refina `localQuestion` con contexto TravelHub (800 ms p95, 600 usuarios concurrentes). |
| ASR Node | `get_indexed_retriever(QA='latencia', content_type='asr', k=6)`. ChromaDB filtra: 57 *chunks* disponibles. Recupera 6 ejemplos de QAS del libro. LLM genera las 6 partes canónicas. |
| Unifier | Presenta ASR formateado con fuentes RAG (página, libro). Almacena en `current_asr` y `last_asr`. `arch_stage='ASR'`. |

#### Prompt 3 — ASR de Escalabilidad
*"Crea un ASR de escalabilidad para TravelHub. El sistema hoy procesa 150 TPM..."*

- `resolved_index='escalabilidad'`. ChromaDB filtra → 46 *chunks*.
- Genera ASR con métricas: 150→800 TPM, autoescalado horizontal, *sharding* geográfico.
- Reemplaza `current_asr`. Fuentes: *Software Architecture in Practice*.

#### Prompt 4 — ASR de Disponibilidad
*"Necesito un ASR de disponibilidad para TravelHub. Actualmente tienen 98.5 % uptime..."*

- `resolved_index='disponibilidad'`. *Keywords*: `'uptime'`, `'RTO'`, `'RPO'`, `'redundancia'`, `'región AWS'`.
- Genera ASR: *Response Measure* = 99.95 % *uptime* (máx. 21.6 min/mes), RTO ≤ 15 min, RPO ≤ 5 min, *failover* automático, *health checks* cada 10 s.

#### Prompt 5 — Estilos Arquitectónicos
*"Basándote en el ASR de escalabilidad... ¿qué estilos arquitectónicos recomiendas?"*

| Etapa | Acción |
|-------|--------|
| Classifier | `intent='style'`, `resolved_index='escalabilidad'`. Detecta referencia al ASR previo. |
| Supervisor | Router traduce `style + escalabilidad → style_escalabilidad` (nodo dinámico vía `make_style_qa_node`). Inyecta `current_asr` como contexto. |
| Style Escalabilidad | `get_indexed_retriever(QA='escalabilidad', content_type='estilos', k=6)`. 7 *chunks* disponibles. *Queries* RAG: *"scalability architecture style"*, *"Bass Clements Kazman styles"*. LLM genera JSON: 2 candidatos + `best_style`. |
| Unifier | Presenta comparación de estilos (ej. Microservicios vs. Event-Driven) con impacto por QA. Almacena `selected_style`. |

#### Prompt 6 — Tácticas de Disponibilidad
*"¿Qué tácticas de disponibilidad recomiendas para TravelHub?"*

- Router activa `tactics_disponibilidad` (nodo dinámico).
- Restricción especial: `_FAULT_DETECTION_CATALOG` con 9 tácticas predefinidas (*Ping/Echo*, *Heartbeat*, *Monitor*, etc.).
- Antes del *refactor* de *keywords*: solo 3 *chunks*. Después: 12+ *chunks* de tácticas reales del libro.
- Genera 3 tácticas en JSON con reparación multi-paso si falla.

#### Prompt 7 — Multi-Intent (ASR + Estilos + Tácticas)
*"Para el módulo de pagos desacoplado... necesito tres cosas en una sola respuesta..."*

| Etapa | Acción |
|-------|--------|
| Supervisor | Detecta 3 *intents*. Cola: `requested_nodes=[asr, style_disponibilidad, tactics_disponibilidad]`. Prioridad: ASR primero. |
| ASR Node | RAG `disp/asr` → ASR de pagos: fallo en Stripe no afecta plataforma, *circuit breaker*, *retry* con *backoff*. |
| Style Disponibilidad | RAG `disp/estilos` → recomienda Event-Driven + Microservicios para desacoplar pagos. Comunicación asíncrona vía eventos. |
| Tactics Disponibilidad | RAG `disp/tácticas` → 3 tácticas del catálogo Fault Detection. |
| Unifier | Consolida ASR + Estilos + Tácticas en modo *Composite* con todas las fuentes RAG. |

#### Prompt 8 — Diagrama Nivel 1 (Overview)
*"Genera un diagrama de arquitectura de alto nivel (overview) para TravelHub..."*

- `intent='diagram'`. Detecta `'alto nivel'`/`'overview'` → `diagram_level=1`.
- Los diagramas **no** usan RAG directamente; el contexto viene del *state* acumulado: `current_asr` + `selected_style` + `tactics_struct`.
- `_resolve_diagram_level()` → nivel 1 (OVERVIEW, 5–15 nodos). *System prompt:* `DOT_SYSTEM_OVERVIEW`.
- Pipeline: `_sanitize_dot()` → `parse_dot_to_model()` → `DiagramModel` (IR). *Rendering* paralelo: `render_svg_b64()` + `render_dot()` + `render_dot_drawio()`.
- Almacena DOT en `diagram_history[1]` para expansión futura.
- *Endpoint:* `/diagram/export?format=svg|dot|drawio`.

#### Prompt 9 — Diagrama Nivel 2 (Expansión Pagos)
*"Expande el diagrama anterior, profundizando en el Servicio de Pagos desacoplado..."*

- Detecta `'expande'`/`'profundizando'` → modo expansión. `diagram_level=2` (MEDIUM, ~30 nodos).
- Carga `diagram_history[1]` existente. Inyecta el DOT del nivel 1 como base. Usa `DOT_SYSTEM_EXPAND`.
- Genera ~30 nodos: orquestador de pagos, adaptadores (Stripe, MercadoPago, OpenPay, PayPal), tokenización PCI-DSS, detector de fraude, reconciliación. Conexiones asíncronas vía eventos.
- *Multi-formato:* SVG base64 + DOT limpio + draw.io XML. Almacena en `diagram_history[2]`.
- **Regla de invalidación:** si se regenera nivel 1, se borran niveles 2 y 3 automáticamente.

#### Prompt 10 — Evaluación de ASR Débil
*"Evalúa este ASR: 'El sistema responde rápido cuando hay muchos usuarios'..."*

| Etapa | Acción |
|-------|--------|
| Classifier | `intent='evaluator'`. `resolved_index='latencia'` (inferido de *"responde rápido"*). |
| Evaluator | `_pick_asr_to_evaluate()` extrae ASR del mensaje. `_book_snippets_for_eval()` usa *retriever* **sin filtros** de QA (evaluación requiere contexto amplio). |
| Verdict | `'Weak'`. **Gaps:** *Response Measure* vago (*"menos de 1 segundo"* sin percentil), *Environment* genérico (*"operación normal"* sin carga), sin métricas p95/p99. **Quality:** no medible, no reproducible. |
| Unifier | Presenta Verdict, Gaps, Quality, Risks & Tactics. Genera *Rewrite* con métricas concretas (p95 ≤800 ms, 600 concurrentes). Cita *snippets* del libro con número de página. |

### 12.3. Demo del Design Ledger (Presentación v3)

Flujo demostrativo del *Design Ledger* sobre TravelHub: contexto → ASR → estilo → tácticas → diagrama → traza LangSmith.

#### ASR Generado en la Demo

**Caso:** Servicio de Búsqueda bajo Carga Pico.

| Parte canónica | Contenido |
|----------------|-----------|
| Source | Usuario final que inicia una búsqueda |
| Stimulus | Solicitud de búsqueda de hospedaje |
| Environment | *Flash sale* · 5.000 búsquedas/s · 20.000 usuarios concurrentes |
| Artifact | Servicio de búsqueda de TravelHub |
| Response | Procesar la solicitud y devolver resultados completos y ordenados |
| Response Measure | p95 < 1000 ms durante 5 minutos de carga sostenida |

#### Estilos Candidatos Generados

**Estilo 1: Microservicios + Caché Distribuida**
- *Ventajas:* escalado horizontal, reducción de carga sobre el *backend*, mejora de latencia en consultas frecuentes, mayor aislamiento de fallos.
- *Limitaciones:* mayor complejidad operativa, posibles problemas de coherencia de caché, no resuelve por sí solo búsquedas complejas.

**Estilo 2: Clúster de Búsqueda Distribuido** ✓ *Recomendado*
- *Ventajas:* distribuye 5.000 búsquedas/s entre nodos, lecturas desde réplicas disponibles, reduce latencia p95, tolerante a picos.
- *Limitaciones:* mayor complejidad operativa, rebalanceo y monitoreo continuos, costos de infraestructura más altos.

**Justificación de la recomendación:** el ASR está centrado en el rendimiento del servicio de búsqueda bajo carga pico. Los microservicios y la caché ayudan, pero no son suficientes si el motor de búsqueda no puede procesar consultas en paralelo con baja variabilidad.

#### Tácticas Recomendadas (compatibles con el estilo)

1. **Particionado de Índices (*Sharding*)** — divide el índice de búsqueda en varias particiones distribuidas. Reduce carga por nodo, procesamiento en paralelo, evita *hotspots*. *Trade-off:* aumenta complejidad de rebalanceo.
2. **Replicación de Lectura** — atender consultas desde réplicas disponibles. Mejora *throughput*, reduce latencia, evita saturar un único nodo, mayor disponibilidad en picos. *Trade-off:* consistencia eventual.
3. **Caché de Resultados y Reducción de *Fan-out*** — cachea búsquedas frecuentes y reduce nodos consultados. Disminuye trabajo por consulta, beneficio especial en *flash sales* (alto *hit-rate*). *Trade-off:* datos desactualizados si TTL incorrecto.

#### Tácticas Excluidas (incompatibles con baja latencia)

| Excluida | Razón |
|----------|-------|
| Consistencia fuerte síncrona | Obliga a sincronizar múltiples réplicas antes de responder. Aumenta latencia directamente. |
| Transacciones distribuidas globales | Añaden bloqueo y coordinación entre nodos. Degradan el tiempo de respuesta. |
| Monolito de búsqueda vertical | No permite escalar horizontalmente. No soporta 5.000 búsquedas/s. |

---

## 13. Trabajo Pendiente y Próximos Pasos

### 13.1. De la presentación de optimizaciones (`archia_optimizaciones_v2.pdf`)

**Optimizaciones de rendimiento aún pendientes:**
- **P1 — LLM calls secuenciales paralelizables** (60–70 % de la latencia total).
- **P2 — Sin *streaming* al cliente** (mejora de latencia percibida).
- **P6 — Un solo *worker* uvicorn** (concurrencia bloqueada).
- **P7 — State/prompt sin límite** (`memory_text` crece sin truncar).

### 13.2. De la presentación de Design Ledger (`archia_presentation_v3.pdf`)

**Estado actual vs. lo que falta:**

| Estado actual | Lo que falta |
|---------------|--------------|
| El arquitecto dice *"dame un ASR de latencia"* y ArchIA ejecuta. | ADD 3.0 empieza **antes**. ¿Cuánto conoce ArchIA del sistema antes de tomar su primera decisión? |

**Próximas dos semanas:** *flujo de intake guiado* en ArchIA — definir qué información mínima necesita un arquitecto antes de tomar su primera decisión de diseño.

**Líneas abiertas para mejorar:**

- **Metodología ADD 3.0 forzada:** el agente debe priorizar el orden de la metodología y guiar al usuario a seguir los pasos secuencialmente.
- **Sistema de proyectos:** poder incluir la documentación técnica de cada proyecto. Un arquitecto de software usa cientos de documentos constantemente, y estos incluyen contexto importante. El agente debe poder tener fácil acceso a estos.
- **Tiempos de respuesta:** se mejoraron, pero aún no están en el nivel deseado. El problema principal radica en que las llamadas a LLM toman mucho tiempo y no hay control sobre eso. Sin embargo, se deben optimizar todas las demás aristas posibles.

### 13.3. Pendientes de la Demo (`ArchIA_Demo_Presentacion_2.pdf`)

La presentación cierra con dos secciones explícitamente marcadas como **a completar por el equipo:**
- Conclusiones.
- Mejoras Futuras.

---

## 14. Referencias Citadas en las Presentaciones

- LangChain. (s. f.). *LangGraph*. https://langchain.com/LangGraph
- LangChain. (s. f.). *LangGraph overview*. Docs by LangChain. https://docs.langchain.com/oss/python/langgraph/overview
- Tiangolo, S. (s. f.). *FastAPI*. https://fastapi.tiangolo.com/
- OpenAI. (s. f.). *OpenAI brand guidelines*. https://openai.com/brand/
- Chroma. (s. f.). *Chroma: The AI-native embedding database*. https://www.trychroma.com/
- LobeHub. (s. f.). *LobeHub icons*. https://lobehub.com/icons/
- Vite. (s. f.). *Vite: Next generation frontend tooling*. https://vite.dev/
- Tencent Cloud. (2025, 26 de septiembre). *How do conversational robots handle multi-intent input?* Tencent Cloud Techpedia. https://www.tencentcloud.com/techpedia/127525
- Santhosh, R. (2024, 6 de septiembre). *Building a multilingual multi-agent chat application using LangGraph — Part I*. Medium. https://medium.com/data-science/building-a-multilingual-multi-agent-chat-application-using-langgraph-i-262d40df6b4f
- Bass, L., Clements, P., & Kazman, R. *Software Architecture in Practice* — referencia base para ASRs, estilos y tácticas (citada repetidamente en *snippets* RAG).
- *Evaluating Software Architecture* — segunda fuente principal del corpus RAG.

---

## Apéndice A — Trazabilidad de Avances por Presentación

| # | Presentación | Tema principal | Hitos cubiertos en este resumen |
|---|--------------|----------------|----------------------------------|
| 1 | `ARCHIA.pdf` | Reunión de Apertura | §2 — Estado Inicial |
| 2 | `ARCHIA_reunión_2.pdf` | Reunión de Seguimiento | §3 (Refactor), §4 (Multi-Intent + Lenguaje), §5 (Mermaid → Graphviz) |
| 3 | `ArchIA_RAG_Indexado.pdf` | Reestructuración del RAG | §6 (RAG indexado), §7 (Refactor de nodos por QA) |
| 4 | `ArchIA_Sistema_Diagramas____Repaired.pdf` | RAG + Refactor + Diagramas | §6, §7, §8 (Sistema de diagramas con niveles), comparaciones antes/después |
| 5 | `archia_optimizaciones_v2.pdf` | Optimizaciones de Rendimiento | §9 (paralelización, caché, async) |
| 6 | `archia_presentation_v3.pdf` | Design Ledger + Demo TravelHub | §10 (Ledger), §11 (Supersession), §12 (Demo TravelHub), §13 (Próximos pasos) |
| 7 | `ArchIA_Demo_Presentacion_2.pdf` | Demo integral con 10 prompts | §12 (10 prompts en detalle, *traza interna* de cada uno) |
