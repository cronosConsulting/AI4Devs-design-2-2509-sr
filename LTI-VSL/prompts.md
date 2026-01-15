# Prompts Utilizados - Ejercicio User Stories LTI

## Información del Proyecto

| Campo | Valor |
|-------|-------|
| **Proyecto** | LTI ATS (FutureHire) |
| **Autor** | VSL |
| **Fecha** | Enero 2025 |
| **Asistente utilizado** | Claude (Anthropic) |

---

## Prompt 1: Generación de User Stories

### Objetivo
Generar User Stories completas a partir de los casos de uso del PRD de LTI.

### Prompt utilizado
```
Actúa como Product Manager de LTI, un ATS colaborativo con IA.

Basándote en el PRD adjunto que incluye:
- 3 casos de uso principales (Gestión Colaborativa, Copiloto IA, Portal Candidato)
- Modelo de datos (USUARIO, VACANTE, CANDIDATO, POSTULACION, ETAPA, FEEDBACK, ANALISIS_IA)
- Arquitectura de microservicios (Core ATS, Servicio Colaboración, Servicio IA)

Genera 4 User Stories siguiendo esta plantilla para cada una:
- Descripción: Como [usuario], Quiero [acción], Para [beneficio]
- Criterios de Aceptación (8-10 criterios verificables)
- Notas Técnicas (alineadas con la arquitectura propuesta)
- Prioridad con justificación
- Dependencias técnicas
- Mockup ASCII si aplica

User Stories a generar:
1. US-001: Pipeline visual de candidatos (del CU1)
2. US-002: Feedback colaborativo en tiempo real (del CU1)
3. US-003: Análisis de CV con IA (del CU2)
4. US-005: Portal de seguimiento para candidatos (del CU3)
```

### Resultado obtenido
Se generaron 4 User Stories completas con todos los elementos solicitados. Cada una incluye entre 8-10 criterios de aceptación verificables, notas técnicas alineadas con la arquitectura (WebSocket, FastAPI, PostgreSQL, etc.) y mockups ASCII ilustrativos.

### Evaluación
- ✅ Criterios de aceptación específicos y testeables
- ✅ Notas técnicas coherentes con el diagrama C4 del PRD
- ✅ Mockups ASCII ayudan a visualizar el resultado esperado
- ✅ Dependencias bien identificadas entre User Stories
- ❌ Inicialmente generó demasiado contenido, tuve que pedir reducir a 4 US

---

## Prompt 2: Priorización del Backlog - Enfoque Simple

### Objetivo
Probar un prompt minimalista para priorizar el backlog.

### Prompt utilizado
```
Prioriza estas User Stories para un backlog de producto:
- US-001: Pipeline visual de candidatos
- US-002: Feedback colaborativo en tiempo real  
- US-003: Análisis de CV con IA
- US-005: Portal de seguimiento para candidatos
```

### Resultado obtenido
Lista ordenada simple:
1. US-001 - Pipeline visual (core del sistema)
2. US-003 - Análisis IA (valor diferencial)
3. US-002 - Feedback colaborativo (complementa US-001)
4. US-005 - Portal candidatos (nice to have)

### Evaluación
- ✅ Respuesta rápida y directa
- ❌ Sin justificación del "por qué" de cada posición
- ❌ No considera dependencias técnicas
- ❌ No hay criterios objetivos de priorización
- ❌ Orden incorrecto: US-002 debería ir antes que US-003 por ser diferenciador core

---

## Prompt 3: Priorización del Backlog - MoSCoW con Contexto

### Objetivo
Probar un prompt con contexto de negocio y metodología específica (MoSCoW).

### Prompt utilizado
```
Actúa como Product Manager de LTI, un ATS colaborativo con IA.

Contexto del producto:
- Nombre: LTI ATS (FutureHire)
- Propuesta de valor: Colaboración en tiempo real entre reclutadores y managers + Smart Screening con IA
- Segmento objetivo: Startups tecnológicas en crecimiento y PYMEs que buscan modernizar HR
- Ventajas competitivas: 
  * Collaboration-First (estilo Google Docs/Figma)
  * Copiloto IA para screening semántico
  * Experiencia transparente del candidato
- Métricas clave: Time-to-hire, Tasa de aceptación de ofertas, NPS de candidatos

User Stories a priorizar:

US-001: Gestión Visual del Pipeline
- Actor: Reclutador
- Valor: Tablero Kanban drag&drop para mover candidatos entre etapas
- Dependencias: Modelo de datos base, WebSocket

US-002: Feedback Colaborativo en Tiempo Real
- Actor: Reclutador/Manager
- Valor: Comentarios y puntuaciones visibles instantáneamente para todo el equipo
- Dependencias: US-001 parcial, WebSocket

US-003: Análisis de CV con IA (Copiloto)
- Actor: Manager/Reclutador
- Valor: Resumen automático, score de ajuste, detección de sesgos
- Dependencias: Servicio de IA desplegado, CV cargado

US-005: Portal de Seguimiento para Candidatos
- Actor: Candidato
- Valor: Ver estado de postulación, próximas entrevistas, notificaciones automáticas
- Dependencias: Sistema de notificaciones, autenticación de candidatos

Tarea: Clasifica cada User Story usando metodología MoSCoW (Must Have, Should Have, Could Have, Won't Have).
Justifica cada clasificación basándote en:
1. Alineación con la propuesta de valor
2. Impacto en métricas clave
3. Dependencias técnicas
4. Viabilidad para MVP
```

### Resultado obtenido

| Categoría | User Story | Justificación |
|-----------|------------|---------------|
| **MUST HAVE** | US-001 | Core funcional del ATS, base técnica para otras features |
| **MUST HAVE** | US-002 | Diferenciador #1 ("Collaboration-First"), sin esto es un ATS genérico |
| **SHOULD HAVE** | US-003 | Alto valor pero requiere infraestructura ML, puede ser post-MVP |
| **SHOULD HAVE** | US-005 | Mejora experiencia candidato, no bloquea operación interna |

### Evaluación
- ✅ Justificaciones alineadas con propuesta de valor del producto
- ✅ Considera dependencias técnicas entre User Stories
- ✅ Distingue claramente MVP vs iteraciones futuras
- ✅ Vincula priorización con métricas clave del negocio
- ✅ Orden final correcto y coherente
- ❌ No da un ranking fino dentro de cada categoría

---

## Prompt 4: Priorización del Backlog - RICE Cuantitativo

### Objetivo
Probar un enfoque cuantitativo con el framework RICE para obtener scores numéricos.

### Prompt utilizado
```
Como Product Manager senior, necesito priorizar el backlog de LTI usando el framework RICE con criterios específicos.

CONTEXTO DEL PRODUCTO:
- LTI es un ATS colaborativo con IA para startups y PYMEs
- Usuarios principales: Reclutadores (50), Managers (30), Candidatos (500+)
- Objetivo Q1: Lanzar MVP funcional con diferenciadores clave

FRAMEWORK RICE - Criterios de evaluación:

REACH (Alcance trimestral):
- ¿Cuántos usuarios únicos usarán esta feature por trimestre?
- Escala: número estimado de usuarios

IMPACT (Impacto en objetivo):
- ¿Cuánto contribuye al objetivo de "reducir time-to-hire y mejorar colaboración"?
- Escala: 3=Masivo, 2=Alto, 1=Medio, 0.5=Bajo, 0.25=Mínimo

CONFIDENCE (Confianza):
- ¿Qué tan seguros estamos del impacto y las estimaciones?
- Escala: 100%=Alta, 80%=Media, 50%=Baja

EFFORT (Esfuerzo):
- ¿Cuántas semanas-persona requiere implementar?
- Considera: frontend, backend, testing, despliegue

USER STORIES A EVALUAR:
[Lista completa con criterios de aceptación y notas técnicas de cada US]

TAREA:
1. Evalúa cada User Story con los 4 factores RICE
2. Calcula el score: (Reach × Impact × Confidence) / Effort
3. Genera ranking final ordenado por score
4. Incluye recomendación de implementación

Formato de salida: Tabla detallada + ranking + recomendaciones
```

### Resultado obtenido

| User Story | Reach | Impact | Confidence | Effort | RICE Score |
|------------|-------|--------|------------|--------|------------|
| US-005 Portal Candidatos | 500 | 1.0 | 80% | 4 sem | **100.0** |
| US-001 Pipeline Visual | 80 | 3.0 | 100% | 4 sem | **60.0** |
| US-002 Feedback Colaborativo | 80 | 2.0 | 80% | 3 sem | **42.7** |
| US-003 Análisis IA | 80 | 2.0 | 50% | 6 sem | **13.3** |

### Evaluación
- ✅ Criterios cuantitativos y reproducibles
- ✅ Cálculos objetivos que eliminan sesgos
- ✅ Considera volumen real de usuarios por tipo
- ✅ Incluye nivel de confianza (útil para features con incertidumbre técnica)
- ⚠️ Resultado sorprendente: US-005 tiene mayor score por alto reach (500 candidatos)
- ❌ RICE puro ignora dependencias técnicas (US-001 debe ir primero aunque tenga menor score)
- ❌ No considera que sin US-001 no puede existir US-002 ni US-003

---

## Prompt 5: Desglose de Tickets Técnicos

### Objetivo
Desglosar una User Story en tickets de trabajo técnicos como en una sesión de planificación de sprint.

### Prompt utilizado
```
Actúa como Tech Lead en una reunión de planificación de sprint.

Contexto:
- Producto: LTI ATS
- Arquitectura: Microservicios (ver diagrama C4)
- Stack: React (frontend), FastAPI/Java (backend), PostgreSQL, Redis, WebSocket, SendGrid, Twilio

User Story a desglosar: US-005 - Portal de Seguimiento para Candidatos

Descripción: Como Candidato, quiero acceder a un portal donde ver el estado de mis postulaciones y próximas entrevistas, para mantenerme informado sin contactar a HR.

Criterios de Aceptación:
[Lista de 9 criterios de aceptación]

Notas Técnicas:
[Detalles de implementación del PRD]

TAREA:
Desglosa esta User Story en tickets técnicos que incluyan:

Para cada ticket:
1. ID y título descriptivo
2. Descripción del ticket
3. Tareas técnicas específicas (checklist)
4. Especificación de API (si aplica) con ejemplo JSON
5. Criterios de aceptación del ticket
6. Dependencias con otros tickets
7. Estimación placeholder

Organiza los tickets por capa:
- Backend (APIs, Auth, Servicios)
- Frontend (Páginas, Componentes)
- QA/DevOps (Testing, Performance)

Incluye un diagrama de dependencias entre tickets.
```

### Resultado obtenido
Se generaron 10 tickets técnicos organizados por capa:
- 5 tickets Backend (TK-001 a TK-005)
- 4 tickets Frontend (TK-006 a TK-009)
- 1 ticket QA/DevOps (TK-010)

Cada ticket incluye tareas técnicas detalladas, especificaciones de API con ejemplos JSON, wireframes ASCII y criterios de aceptación específicos.

### Evaluación
- ✅ Tickets bien granulares y estimables independientemente
- ✅ Dependencias claras entre tickets con diagrama visual
- ✅ Especificaciones de API con ejemplos JSON concretos
- ✅ Wireframes ASCII ayudan a alinear expectativas con frontend
- ✅ Consideración de aspectos de seguridad (JWT, rate limiting, hashing)
- ❌ Algunos tickets podrían subdividirse más (ej: TK-003 Autenticación es muy grande)

---

## Prompt 6: Estimación en Horas

### Objetivo
Estimar el esfuerzo de los tickets técnicos en horas.

### Prompt utilizado
```
Estima en horas cada uno de los siguientes tickets técnicos para US-005.

Contexto:
- Desarrollador de nivel medio-senior
- Incluir: desarrollo, code review, testing unitario, documentación
- No incluir: reuniones, despliegue, bugs post-release
- Añadir factor de contingencia del 20%

Para cada ticket proporciona:
1. Estimación base (horas)
2. Contingencia (+20%)
3. Total
4. Desglose por tarea principal

Tickets a estimar:
[Lista de 10 tickets con sus tareas técnicas]

Además incluye:
- Distribución de horas por rol (Backend/Frontend/QA)
- Planificación sugerida asumiendo 2 desarrolladores en paralelo
- Riesgos identificados que podrían impactar las estimaciones
```

### Resultado obtenido

| Ticket | Estimación Base | Contingencia | Total |
|--------|-----------------|--------------|-------|
| TK-001 | 10h | 2h | 12h |
| TK-002 | 6h | 1h | 7h |
| TK-003 | 16h | 3h | 19h |
| TK-004 | 12h | 2h | 14h |
| TK-005 | 8h | 2h | 10h |
| TK-006 | 14h | 3h | 17h |
| TK-007 | 12h | 2h | 14h |
| TK-008 | 14h | 3h | 17h |
| TK-009 | 10h | 2h | 12h |
| TK-010 | 16h | 3h | 19h |
| **TOTAL** | | | **141h** |

Planificación: 2.5 semanas con 2 desarrolladores trabajando en paralelo.

### Evaluación
- ✅ Estimaciones realistas y desglosadas por tarea
- ✅ Factor de contingencia incluido
- ✅ Distribución por rol útil para asignación
- ✅ Planificación de sprints práctica
- ✅ Riesgos identificados con mitigaciones
- ❌ Las estimaciones pueden variar según seniority real del equipo

---

## Conclusiones sobre Efectividad de Prompts

### Mejor prompt para generar User Stories:
El prompt con **contexto completo del PRD + plantilla específica** (Prompt 1) fue el más efectivo porque:
1. Proporciona información suficiente para generar contenido alineado con la arquitectura
2. La plantilla estructura el output de forma consistente
3. Referencia al modelo de datos asegura notas técnicas coherentes

### Mejor prompt para priorizar Backlog:
El prompt **MoSCoW con contexto de negocio** (Prompt 3) fue el más efectivo porque:
1. Fuerza a pensar en términos de MVP vs iteraciones
2. Las justificaciones quedan documentadas
3. Considera tanto valor de negocio como viabilidad técnica

**RICE** (Prompt 4) fue útil como complemento cuantitativo, pero su resultado puro (US-005 primero) contradice las dependencias técnicas. La combinación MoSCoW + RICE da el mejor resultado.

### Mejor prompt para tickets técnicos:
El prompt con **rol de Tech Lead + estructura por capas** (Prompt 5) funcionó bien porque:
1. El rol contextualiza el nivel de detalle esperado
2. La estructura por capas organiza naturalmente el trabajo
3. Pedir ejemplos JSON fuerza especificaciones concretas

### Aprendizajes Generales:

1. **El contexto es clave:** Prompts sin contexto del producto generan respuestas genéricas. Incluir PRD, arquitectura y métricas mejora significativamente la calidad.

2. **Plantillas estructuran mejor:** Proporcionar la estructura exacta esperada (plantilla de US, formato de ticket) genera outputs más consistentes y utilizables.

3. **Roles especializados ayudan:** Pedir que actúe como "Product Manager" o "Tech Lead" ajusta el nivel de detalle y enfoque del output.

4. **Combinación de metodologías:** Ninguna metodología única (MoSCoW, RICE, etc.) es perfecta. Combinar varias da perspectivas complementarias.

5. **Iterar es necesario:** El primer output raramente es perfecto. Pedir ajustes específicos ("reduce a 4 US", "añade estimaciones") mejora el resultado final.

6. **Validar dependencias técnicas:** Los frameworks de priorización puros no consideran dependencias técnicas. Es necesario ajustar manualmente el orden final.

### Patrón de Prompt Recomendado:

```
[ROL]: Actúa como [Product Manager / Tech Lead / etc.]

[CONTEXTO]:
- Información del producto
- Arquitectura/Stack técnico
- Restricciones y métricas

[INPUT]:
- Datos específicos a procesar
- Documentación de referencia

[TAREA]:
- Acción específica a realizar
- Metodología a usar

[OUTPUT]:
- Formato esperado (tabla, lista, etc.)
- Elementos requeridos
- Ejemplos si aplica
```

---

*Documento generado como parte del ejercicio AI4Devs - Módulo de User Stories*
