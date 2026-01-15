# User Stories - LTI ATS (FutureHire)

## Información del Proyecto

| Campo | Valor |
|-------|-------|
| **Proyecto** | LTI ATS (FutureHire) |
| **Autor** | VSL |
| **Fecha** | Enero 2025 |
| **Versión** | 1.0 |

---

## Índice de User Stories

| ID | Título | Actor | Prioridad |
|----|--------|-------|-----------|
| US-001 | Gestión visual del pipeline de candidatos | Reclutador | Alta |
| US-002 | Feedback colaborativo en tiempo real | Reclutador/Manager | Alta |
| US-003 | Análisis de CV con IA (Copiloto) | Manager/Reclutador | Alta |
| US-005 | Portal de seguimiento para candidatos | Candidato | Media |

---

## User Story US-001: Gestión Visual del Pipeline de Candidatos

### Descripción

**Como** Reclutador,  
**Quiero** visualizar y gestionar los candidatos en un tablero Kanban con drag & drop,  
**Para** tener una visión clara del estado de cada candidato y moverlos eficientemente entre etapas del proceso de selección.

### Criterios de Aceptación

- [ ] El sistema muestra un tablero Kanban con columnas correspondientes a cada etapa del proceso (ej: Recibido, Revisión, Entrevista, Oferta, Contratado, Descartado)
- [ ] Cada tarjeta de candidato muestra: nombre, foto (si disponible), vacante a la que postula y tiempo en la etapa actual
- [ ] El usuario puede arrastrar y soltar una tarjeta de candidato de una columna a otra
- [ ] Al mover un candidato, el sistema actualiza automáticamente el campo `etapa_actual_id` en la tabla POSTULACION
- [ ] El sistema envía una notificación al Manager de Contratación cuando un candidato cambia de etapa
- [ ] El tablero se actualiza en tiempo real para todos los usuarios que lo estén visualizando (WebSocket)
- [ ] El sistema registra un log de auditoría con: usuario, fecha/hora, etapa origen y etapa destino
- [ ] El usuario puede filtrar el tablero por vacante específica
- [ ] El sistema permite personalizar las etapas del pipeline por vacante

### Notas Técnicas

- **Frontend:** Implementar con librería de drag & drop (react-beautiful-dnd o similar)
- **Backend:** Endpoint PUT `/api/postulaciones/{id}/etapa` para actualizar etapa
- **WebSocket:** Usar el Servicio de Colaboración para broadcast de cambios en tiempo real
- **Base de datos:** Actualizar tabla POSTULACION (campo `etapa_actual_id`) y crear tabla de auditoría
- **Notificaciones:** Integrar con Broker de Mensajes (Kafka) para notificar al Manager

### Prioridad

**Alta** - Es la funcionalidad core del ATS. Sin el pipeline visual, el sistema no tiene valor diferencial.

### Estimación

*Por definir en planificación*

### Dependencias

- Modelo de datos base implementado (tablas: POSTULACION, ETAPA, VACANTE, CANDIDATO, USUARIO)
- Servicio de Colaboración (WebSocket) configurado
- Sistema de autenticación y roles funcionando

### Mockup de Referencia

```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│  RECIBIDO   │  REVISIÓN   │ ENTREVISTA  │   OFERTA    │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ ┌─────────┐ │ ┌─────────┐ │             │             │
│ │ Juan P. │ │ │ María L.│ │             │             │
│ │ Dev Sr. │ │ │ Dev Jr. │ │             │             │
│ │ 2 días  │ │ │ 5 días  │ │             │             │
│ └─────────┘ │ └─────────┘ │             │             │
│ ┌─────────┐ │             │             │             │
│ │ Ana R.  │ │             │             │             │
│ │ QA Lead │ │             │             │             │
│ │ 1 día   │ │             │             │             │
│ └─────────┘ │             │             │             │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

---

## User Story US-002: Feedback Colaborativo en Tiempo Real

### Descripción

**Como** Reclutador o Manager de Contratación,  
**Quiero** añadir comentarios y evaluaciones en la ficha de un candidato y ver en tiempo real los comentarios de otros colaboradores,  
**Para** tomar decisiones de contratación informadas y coordinadas con mi equipo sin necesidad de reuniones síncronas.

### Criterios de Aceptación

- [ ] La ficha del candidato incluye una sección de "Feedback del equipo" visible para Reclutadores y Managers
- [ ] El usuario puede escribir un comentario de texto libre (máximo 2000 caracteres)
- [ ] El usuario puede asignar una puntuación numérica (1-5 estrellas) junto con el comentario
- [ ] Los comentarios aparecen en tiempo real para otros usuarios que estén viendo la misma ficha (sin recargar página)
- [ ] Cada comentario muestra: autor, fecha/hora, contenido y puntuación
- [ ] El sistema indica visualmente cuando otro usuario está escribiendo un comentario ("María está escribiendo...")
- [ ] El usuario puede marcar un comentario como "colaborativo" para diferenciarlo de notas privadas
- [ ] El sistema calcula y muestra la puntuación promedio del candidato basada en todos los feedbacks
- [ ] Los comentarios se ordenan cronológicamente (más reciente primero)
- [ ] El usuario puede editar o eliminar sus propios comentarios dentro de las primeras 24 horas

### Notas Técnicas

- **Frontend:** Componente de chat/comentarios con actualización vía WebSocket
- **Backend:** Endpoints CRUD para `/api/postulaciones/{id}/feedback`
- **WebSocket:** Eventos `feedback_created`, `feedback_updated`, `user_typing`
- **Base de datos:** Tabla FEEDBACK con campos: id, postulacion_id, usuario_id, contenido, puntuacion, colaborativo (boolean), created_at, updated_at
- **Cache:** Usar Redis para gestionar estado de "usuario escribiendo" y reducir latencia

### Prioridad

**Alta** - Es el diferenciador principal de LTI ("Collaboration-First"). Sin esta funcionalidad, el sistema es un ATS tradicional.

### Estimación

*Por definir en planificación*

### Dependencias

- US-001 (Pipeline visual) parcialmente implementado (ficha de candidato accesible)
- Servicio de Colaboración (WebSocket) configurado
- Sistema de autenticación funcionando

### Ejemplo de Interacción

```
┌──────────────────────────────────────────────────────┐
│ FEEDBACK DEL EQUIPO                    Promedio: ⭐4.2│
├──────────────────────────────────────────────────────┤
│ 👤 Carlos (Manager) - Hace 5 min           ⭐⭐⭐⭐⭐ │
│ "Excelente experiencia en microservicios.           │
│  Recomiendo avanzar a entrevista técnica."          │
├──────────────────────────────────────────────────────┤
│ 👤 Laura (Recruiter) - Hace 2 horas        ⭐⭐⭐⭐  │
│ "Buena comunicación en el primer contacto.          │
│  Expectativa salarial dentro del rango."            │
├──────────────────────────────────────────────────────┤
│ ✍️ María está escribiendo...                         │
├──────────────────────────────────────────────────────┤
│ [Escribe tu comentario...]              ⭐⭐⭐⭐⭐    │
│                                         [Enviar]    │
└──────────────────────────────────────────────────────┘
```

---

## User Story US-003: Análisis de CV con IA (Copiloto)

### Descripción

**Como** Manager de Contratación o Reclutador,  
**Quiero** solicitar un análisis automático del CV de un candidato mediante IA que genere un resumen de habilidades, experiencia relevante y una puntuación de ajuste,  
**Para** evaluar candidatos de forma objetiva y rápida, reduciendo el tiempo de screening manual.

### Criterios de Aceptación

- [ ] La ficha del candidato incluye un botón "Analizar con IA" visible cuando el candidato tiene CV cargado
- [ ] Al hacer clic, el sistema muestra un indicador de "Analizando..." mientras procesa
- [ ] El análisis genera y muestra:
  - Resumen ejecutivo del perfil (máximo 300 palabras)
  - Lista de habilidades clave extraídas del CV
  - Experiencia relevante para la vacante específica
  - Puntuación de ajuste (score 0-100%) con justificación
- [ ] El sistema detecta y alerta sobre posibles sesgos en el CV (opcional, si aplica)
- [ ] El análisis se almacena en la tabla ANALISIS_IA y se asocia a la postulación
- [ ] El usuario puede regenerar el análisis si el CV se actualiza
- [ ] El tiempo de respuesta del análisis no supera los 30 segundos
- [ ] El análisis considera los requisitos específicos de la vacante (descripción, skills requeridos)
- [ ] El usuario puede ver el historial de análisis previos si existen

### Notas Técnicas

- **Frontend:** Componente de resultados de IA con estados (loading, success, error)
- **Backend:** Endpoint POST `/api/postulaciones/{id}/analizar` que invoca al Servicio de IA
- **Servicio de IA (Copiloto):**
  - API REST de entrada (FastAPI)
  - Manager de Modelos (MLFlow) para versionado
  - Motor de Análisis (TensorFlow/NLP) para procesamiento
- **Modelo ML:** Usar embeddings para comparar CV vs descripción de vacante
- **Base de datos:** Tabla ANALISIS_IA con campos: id, postulacion_id, resumen, score_ajuste, habilidades_json, alertas_sesgo, created_at
- **Almacenamiento:** Modelos en AWS S3/GCP Storage

### Prioridad

**Alta** - Es la propuesta de valor diferencial de IA del producto ("Smart Screening").

### Estimación

*Por definir en planificación*

### Dependencias

- CV del candidato cargado en el sistema (campo `cv_url` en tabla CANDIDATO)
- Servicio de IA (Copiloto) desplegado y accesible
- Modelo de ML entrenado y versionado en el Manager de Modelos
- Vacante con descripción y requisitos definidos

### Diagrama de Flujo del Análisis

```
┌─────────┐     POST /analyze      ┌─────────────┐
│ Frontend│────────────────────────▶│ API REST    │
└─────────┘                        │ (FastAPI)   │
     ▲                             └──────┬──────┘
     │                                    │
     │ JSON Response                      ▼
     │                             ┌─────────────┐
     │                             │ Manager de  │
     │                             │ Modelos     │
     │                             └──────┬──────┘
     │                                    │
     │                                    ▼
     │                             ┌─────────────┐
     │                             │ Motor de    │
     └─────────────────────────────│ Análisis NLP│
                                   └──────┬──────┘
                                          │
                                          ▼
                                   ┌─────────────┐
                                   │ Base Datos  │
                                   │ (Guardar)   │
                                   └─────────────┘
```

### Ejemplo de Output del Análisis

```
┌──────────────────────────────────────────────────────┐
│ 🤖 ANÁLISIS IA                      Score: 87%  🟢   │
├──────────────────────────────────────────────────────┤
│ RESUMEN EJECUTIVO                                    │
│ Profesional con 5 años de experiencia en desarrollo │
│ backend con Java y Python. Destaca su trabajo en    │
│ arquitecturas de microservicios y su certificación  │
│ en AWS. Buen fit cultural para startups.            │
├──────────────────────────────────────────────────────┤
│ HABILIDADES CLAVE                                    │
│ ✓ Java (5 años)  ✓ Python (3 años)  ✓ AWS          │
│ ✓ Docker         ✓ Kubernetes       ✓ PostgreSQL   │
├──────────────────────────────────────────────────────┤
│ EXPERIENCIA RELEVANTE PARA ESTA VACANTE             │
│ • Lideró migración a microservicios (2 años)        │
│ • Implementó CI/CD en equipo de 8 personas          │
├──────────────────────────────────────────────────────┤
│ ⚠️ ALERTAS: Ninguna detectada                        │
└──────────────────────────────────────────────────────┘
```

---

## User Story US-005: Portal de Seguimiento para Candidatos

### Descripción

**Como** Candidato,  
**Quiero** acceder a un portal personalizado donde pueda ver el estado actual de mi postulación y los próximos pasos del proceso,  
**Para** mantenerme informado sin necesidad de contactar a Recursos Humanos y tener una experiencia de candidatura transparente.

### Criterios de Aceptación

- [ ] El candidato puede acceder al portal con credenciales (email + contraseña o link mágico)
- [ ] El portal muestra todas las vacantes a las que el candidato ha postulado
- [ ] Para cada postulación se visualiza:
  - Título de la vacante y empresa
  - Etapa actual del proceso (ej: "Entrevista en curso")
  - Fecha de última actualización
  - Indicador visual de progreso (barra o steps)
- [ ] Si hay una entrevista programada, el sistema muestra: fecha, hora y enlace de videollamada
- [ ] El candidato recibe notificaciones automáticas (email/SMS según preferencia) cuando su estado cambia
- [ ] El portal es responsive y funciona correctamente en dispositivos móviles
- [ ] El candidato puede actualizar sus datos de contacto y preferencias de notificación
- [ ] El sistema muestra mensajes personalizados según la etapa (ej: "¡Felicidades! Has avanzado a la fase de entrevistas")
- [ ] El tiempo de carga del portal no supera los 3 segundos

### Notas Técnicas

- **Frontend:** SPA con React, diseño mobile-first
- **Backend:** Endpoints GET `/api/candidato/postulaciones` y GET `/api/candidato/entrevistas`
- **Autenticación:** JWT con refresh token, opción de magic link por email
- **Notificaciones:** 
  - Servicio de Notificaciones integrado con SendGrid (email) y Twilio (SMS)
  - Eventos disparados desde el Broker de Mensajes cuando cambia `etapa_actual_id`
- **Base de datos:** Consultas a tablas CANDIDATO, POSTULACION, VACANTE, ETAPA
- **Cache:** Redis para datos de sesión y reducir queries frecuentes

### Prioridad

**Media** - Importante para la experiencia del candidato y diferenciación, pero no bloquea el uso core del ATS por parte de reclutadores.

### Estimación

*Por definir en planificación*

### Dependencias

- Modelo de datos base implementado
- Sistema de autenticación para candidatos (separado del de usuarios internos)
- Servicio de Notificaciones configurado
- Bot de Automatización para gestión de entrevistas (para mostrar datos de citas)

### Mockup del Portal

```
┌──────────────────────────────────────────────────────┐
│ 🏠 Portal LTI                      👤 Juan Pérez    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  MIS POSTULACIONES                                   │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ 💼 Senior Backend Developer - TechCorp         │ │
│  │                                                │ │
│  │ Estado: ✅ Entrevista en curso                 │ │
│  │ Actualizado: Hace 2 días                       │ │
│  │                                                │ │
│  │ ○────────●────────○────────○                   │ │
│  │ Recibido  Entrevista  Oferta  Contratado      │ │
│  │                                                │ │
│  │ 📅 PRÓXIMA ENTREVISTA                          │ │
│  │ 📆 Lunes 20 Ene, 10:00 AM                      │ │
│  │ 🔗 [Unirse a Google Meet]                      │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ 💼 QA Engineer - StartupXYZ                    │ │
│  │                                                │ │
│  │ Estado: 📋 En revisión                         │ │
│  │ Actualizado: Hace 5 días                       │ │
│  │                                                │ │
│  │ ●────────○────────○────────○                   │ │
│  │ Recibido  Entrevista  Oferta  Contratado      │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Resumen de User Stories

| ID | Título | Actor | Prioridad | Caso de Uso |
|----|--------|-------|-----------|-------------|
| US-001 | Gestión visual del pipeline | Reclutador | Alta | CU1 - Gestión Colaborativa |
| US-002 | Feedback colaborativo en tiempo real | Reclutador/Manager | Alta | CU1 - Gestión Colaborativa |
| US-003 | Análisis de CV con IA | Manager/Reclutador | Alta | CU2 - Copiloto IA |
| US-005 | Portal de seguimiento | Candidato | Media | CU3 - Experiencia Candidato |

---

## Backlog de Producto Priorizado

### Metodología de Priorización

Se evaluaron **3 enfoques diferentes** para priorizar el backlog:

| Enfoque | Descripción | Resultado |
|---------|-------------|-----------|
| **Prompt Simple** | Lista directa sin contexto | Orden básico sin justificación |
| **MoSCoW + Contexto** | Clasificación con contexto de negocio | Mejor equilibrio negocio/técnica |
| **RICE Cuantitativo** | Scoring numérico (Reach×Impact×Confidence/Effort) | Cuantitativo pero ignora dependencias |

**Conclusión:** El enfoque **MoSCoW con contexto de negocio** fue el más efectivo porque:
1. Alinea prioridades con la propuesta de valor del producto
2. Distingue claramente el MVP de iteraciones futuras
3. Considera las dependencias técnicas entre User Stories

### Clasificación MoSCoW

#### 🔴 MUST HAVE (Imprescindible para MVP)

| ID | User Story | Justificación |
|----|------------|---------------|
| US-001 | Pipeline Visual de Candidatos | Core funcional del ATS. Sin esto no hay producto. Base técnica para otras features (WebSocket). |
| US-002 | Feedback Colaborativo en Tiempo Real | Diferenciador #1 de LTI ("Collaboration-First"). Sin colaboración, es un ATS genérico más. |

#### 🟡 SHOULD HAVE (Importante, segunda iteración)

| ID | User Story | Justificación |
|----|------------|---------------|
| US-003 | Análisis de CV con IA | Alto valor diferencial pero requiere infraestructura ML compleja. Puede lanzarse post-MVP. |
| US-005 | Portal de Seguimiento para Candidatos | Mejora experiencia del candidato y NPS. No bloquea operación de reclutadores. |

### Matriz RICE (Análisis Cuantitativo)

| User Story | Reach | Impact | Confidence | Effort | RICE Score |
|------------|-------|--------|------------|--------|------------|
| US-001 Pipeline Visual | 80 | 3.0 | 100% | 4 sem | **60.0** |
| US-002 Feedback Colaborativo | 80 | 2.0 | 80% | 3 sem | **42.7** |
| US-003 Análisis IA | 80 | 2.0 | 50% | 6 sem | **13.3** |
| US-005 Portal Candidatos | 500 | 1.0 | 80% | 4 sem | **100.0** |

> **Nota:** RICE favorece US-005 por el alto reach (500 candidatos vs 80 usuarios internos), pero las dependencias técnicas requieren implementar primero US-001 y US-002.

### 📋 Backlog Final Ordenado

| Prioridad | ID | Título | Categoría | Sprint | Dependencias |
|-----------|-----|--------|-----------|--------|--------------|
| 🥇 1 | US-001 | Pipeline Visual de Candidatos | Must Have | 1-2 | Modelo de datos, Auth |
| 🥈 2 | US-002 | Feedback Colaborativo en Tiempo Real | Must Have | 3 | US-001, WebSocket |
| 🥉 3 | US-003 | Análisis de CV con IA (Copiloto) | Should Have | 4-6 | Servicio IA, US-001 |
| 4 | US-005 | Portal de Seguimiento para Candidatos | Should Have | 7-8 | Notificaciones, Auth candidatos |

### Roadmap Visual

```
SPRINT 1-2          SPRINT 3           SPRINT 4-6         SPRINT 7-8
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   US-001    │    │   US-002    │    │   US-003    │    │   US-005    │
│  Pipeline   │───▶│  Feedback   │───▶│  IA/Copiloto│    │   Portal    │
│   Visual    │    │ Colaborativo│    │             │    │  Candidato  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
   MUST HAVE          MUST HAVE         SHOULD HAVE       SHOULD HAVE
                                              │                 │
                                              └────────┬────────┘
                                                       │
                                              Pueden desarrollarse
                                                en paralelo
```

---

## Tickets de Trabajo - US-005: Portal de Seguimiento para Candidatos

### Resumen del Desglose

La User Story US-005 se descompone en **10 tickets técnicos** organizados por capa:

| Capa | Tickets | IDs |
|------|---------|-----|
| Backend - API | 2 tickets | TK-001, TK-002 |
| Backend - Autenticación | 1 ticket | TK-003 |
| Backend - Notificaciones | 2 tickets | TK-004, TK-005 |
| Frontend | 4 tickets | TK-006, TK-007, TK-008, TK-009 |
| Testing & Performance | 1 ticket | TK-010 |

### Diagrama de Dependencias

```
TK-003 (Auth) ─────────────────────────────────┐
      │                                        │
      ▼                                        ▼
TK-001 (API Postulaciones) ──────────▶ TK-006 (Login UI)
      │                                        │
      ▼                                        ▼
TK-002 (API Entrevistas) ────────────▶ TK-007 (Dashboard UI)
                                               │
TK-004 (Notif Email) ──┐                       ▼
                       ├─────────────▶ TK-008 (Detalle UI)
TK-005 (Notif SMS) ────┘                       │
                                               ▼
                                       TK-009 (Perfil UI)
                                               │
                                               ▼
                                       TK-010 (Testing & Perf)
```

---

### TK-001: API REST - Endpoints de Postulaciones del Candidato

#### Descripción
Implementar los endpoints backend que permiten a un candidato autenticado consultar sus postulaciones y el estado de cada una.

#### Tareas Técnicas
- [ ] Crear endpoint `GET /api/candidato/postulaciones` - Lista todas las postulaciones del candidato
- [ ] Crear endpoint `GET /api/candidato/postulaciones/{id}` - Detalle de una postulación específica
- [ ] Implementar query con JOINs a tablas: POSTULACION, VACANTE, ETAPA
- [ ] Añadir campo calculado `dias_en_etapa` (fecha actual - fecha última actualización)
- [ ] Incluir información de la vacante: título, empresa, descripción
- [ ] Incluir información de la etapa: nombre, orden en el pipeline
- [ ] Implementar paginación (limit/offset) para lista de postulaciones
- [ ] Añadir filtro opcional por estado (activa/finalizada)

#### Especificación de API

```json
// GET /api/candidato/postulaciones
// Response 200:
{
  "data": [
    {
      "id": 123,
      "vacante": {
        "id": 456,
        "titulo": "Senior Backend Developer",
        "empresa": "TechCorp",
        "ubicacion": "Madrid"
      },
      "etapa_actual": {
        "id": 3,
        "nombre": "Entrevista",
        "orden": 3
      },
      "fecha_postulacion": "2025-01-10T10:30:00Z",
      "fecha_actualizacion": "2025-01-15T14:00:00Z",
      "dias_en_etapa": 2
    }
  ],
  "pagination": {
    "total": 5,
    "limit": 10,
    "offset": 0
  }
}
```

#### Criterios de Aceptación del Ticket
- [ ] Endpoints responden en menos de 200ms
- [ ] Solo devuelve postulaciones del candidato autenticado (validación JWT)
- [ ] Manejo correcto de errores (401, 404, 500)
- [ ] Tests unitarios con cobertura >80%
- [ ] Documentación OpenAPI/Swagger actualizada

#### Dependencias
- TK-003 (Sistema de autenticación) debe estar completado

#### Estimación
**12 horas**

---

### TK-002: API REST - Endpoints de Entrevistas Programadas

#### Descripción
Implementar endpoint que devuelve las entrevistas programadas para el candidato, incluyendo fecha, hora y enlace de videollamada.

#### Tareas Técnicas
- [ ] Crear endpoint `GET /api/candidato/entrevistas` - Lista entrevistas pendientes
- [ ] Crear endpoint `GET /api/candidato/entrevistas/{id}` - Detalle de entrevista
- [ ] Crear/extender tabla ENTREVISTA si no existe (id, postulacion_id, fecha_hora, enlace_videollamada, tipo, notas)
- [ ] Implementar query con JOIN a POSTULACION y VACANTE
- [ ] Filtrar solo entrevistas futuras por defecto
- [ ] Añadir parámetro `?incluir_pasadas=true` para histórico
- [ ] Ordenar por fecha_hora ascendente

#### Especificación de API

```json
// GET /api/candidato/entrevistas
// Response 200:
{
  "data": [
    {
      "id": 789,
      "postulacion_id": 123,
      "vacante_titulo": "Senior Backend Developer",
      "tipo": "Entrevista Técnica",
      "fecha_hora": "2025-01-20T10:00:00Z",
      "duracion_minutos": 60,
      "enlace_videollamada": "https://meet.google.com/abc-defg-hij",
      "notas": "Preparar caso práctico de arquitectura"
    }
  ]
}
```

#### Criterios de Aceptación del Ticket
- [ ] Endpoint responde en menos de 150ms
- [ ] Solo devuelve entrevistas de postulaciones del candidato autenticado
- [ ] Formato de fecha ISO 8601 con timezone
- [ ] Tests unitarios implementados
- [ ] Documentación Swagger actualizada

#### Dependencias
- TK-003 (Autenticación)
- TK-001 (API Postulaciones) - para consistencia de estructura

#### Estimación
**7 horas**

---

### TK-003: Sistema de Autenticación para Candidatos

#### Descripción
Implementar sistema de autenticación independiente para candidatos con soporte para login tradicional (email/password) y magic link.

#### Tareas Técnicas
- [ ] Crear endpoint `POST /api/candidato/auth/registro` - Registro de nuevo candidato
- [ ] Crear endpoint `POST /api/candidato/auth/login` - Login con email/password
- [ ] Crear endpoint `POST /api/candidato/auth/magic-link` - Solicitar magic link por email
- [ ] Crear endpoint `GET /api/candidato/auth/magic-link/{token}` - Validar magic link
- [ ] Crear endpoint `POST /api/candidato/auth/refresh` - Renovar access token
- [ ] Crear endpoint `POST /api/candidato/auth/logout` - Invalidar refresh token
- [ ] Implementar generación de JWT (access token: 15min, refresh token: 7 días)
- [ ] Crear tabla CANDIDATO_AUTH (candidato_id, password_hash, magic_token, magic_token_expiry)
- [ ] Implementar middleware de validación JWT para rutas `/api/candidato/*`
- [ ] Hashear passwords con bcrypt (cost factor 12)
- [ ] Magic links expiran en 15 minutos y son de un solo uso

#### Flujo de Magic Link

```
┌──────────┐     POST /magic-link      ┌──────────┐     SendGrid      ┌──────────┐
│ Candidato│──────────────────────────▶│  Backend │──────────────────▶│  Email   │
└──────────┘                           └──────────┘                   └──────────┘
     │                                       │
     │    Click en link del email            │
     │◀──────────────────────────────────────┘
     │
     │         GET /magic-link/{token}
     │──────────────────────────────────────▶│
     │                                       │
     │◀──────────────────────────────────────│
     │         { access_token, refresh_token }
```

#### Criterios de Aceptación del Ticket
- [ ] Registro valida email único y formato correcto
- [ ] Password requiere mínimo 8 caracteres, 1 mayúscula, 1 número
- [ ] Magic link funciona correctamente y expira después de uso
- [ ] JWT incluye claims: candidato_id, email, exp, iat
- [ ] Refresh token se almacena de forma segura (httpOnly cookie o DB)
- [ ] Tests de seguridad: no expone información sensible en errores
- [ ] Rate limiting en endpoints de auth (máx 5 intentos/minuto)

#### Dependencias
- Ninguna (es base para otros tickets)

#### Estimación
**19 horas**

---

### TK-004: Servicio de Notificaciones - Email (SendGrid)

#### Descripción
Implementar servicio de envío de emails transaccionales usando SendGrid para notificar a candidatos sobre cambios en sus postulaciones.

#### Tareas Técnicas
- [ ] Configurar SDK de SendGrid en el proyecto
- [ ] Crear servicio `NotificationEmailService` con métodos por tipo de notificación
- [ ] Diseñar templates de email en SendGrid:
  - `candidato_bienvenida` - Email de registro exitoso
  - `candidato_etapa_cambio` - Cambio de etapa en postulación
  - `candidato_entrevista_programada` - Nueva entrevista agendada
  - `candidato_entrevista_recordatorio` - Recordatorio 24h antes
  - `candidato_magic_link` - Link de acceso sin password
- [ ] Implementar cola de mensajes (Redis/Bull) para envío asíncrono
- [ ] Crear tabla NOTIFICACION_LOG (id, candidato_id, tipo, canal, estado, enviado_at, error)
- [ ] Implementar retry logic (3 intentos con backoff exponencial)
- [ ] Añadir tracking de apertura y clicks (opcional)

#### Templates de Email (Variables)

```
Template: candidato_etapa_cambio
Variables:
  - {{nombre_candidato}}
  - {{titulo_vacante}}
  - {{empresa}}
  - {{etapa_anterior}}
  - {{etapa_nueva}}
  - {{mensaje_personalizado}}
  - {{link_portal}}
```

#### Criterios de Aceptación del Ticket
- [ ] Emails se envían en menos de 5 segundos tras el trigger
- [ ] Templates son responsive (mobile-friendly)
- [ ] Incluyen enlace de "darse de baja" de notificaciones
- [ ] Logs registran todos los envíos (éxito y error)
- [ ] Configuración de SendGrid via variables de entorno
- [ ] Tests con mock de SendGrid API

#### Dependencias
- Cuenta de SendGrid configurada (API Key)
- TK-003 para obtener datos del candidato

#### Estimación
**14 horas**

---

### TK-005: Servicio de Notificaciones - SMS (Twilio)

#### Descripción
Implementar servicio de envío de SMS usando Twilio como canal alternativo de notificación para candidatos.

#### Tareas Técnicas
- [ ] Configurar SDK de Twilio en el proyecto
- [ ] Crear servicio `NotificationSMSService`
- [ ] Definir templates de SMS (máximo 160 caracteres):
  - `sms_etapa_cambio` - "LTI: Tu postulación a {vacante} avanzó a {etapa}. Ver más: {link}"
  - `sms_entrevista_recordatorio` - "LTI: Recordatorio entrevista mañana {hora}. Link: {enlace}"
- [ ] Añadir campo `telefono` y `preferencia_notificacion` a tabla CANDIDATO
- [ ] Implementar validación de formato de teléfono (E.164)
- [ ] Integrar con cola de mensajes existente (Redis/Bull)
- [ ] Registrar en NOTIFICACION_LOG con canal='SMS'
- [ ] Implementar opt-out vía respuesta "STOP"

#### Criterios de Aceptación del Ticket
- [ ] SMS se envía solo si candidato tiene `preferencia_notificacion` incluye 'sms'
- [ ] Teléfono validado en formato internacional E.164
- [ ] Respeta horarios (no enviar entre 22:00 y 08:00 hora local)
- [ ] Logs completos de envíos
- [ ] Manejo de errores de Twilio (número inválido, sin fondos, etc.)
- [ ] Tests con mock de Twilio API

#### Dependencias
- Cuenta de Twilio configurada
- TK-004 (comparten infraestructura de colas)

#### Estimación
**10 horas**

---

### TK-006: Frontend - Página de Login y Registro de Candidatos

#### Descripción
Implementar las pantallas de autenticación del portal de candidatos: login, registro y solicitud de magic link.

#### Tareas Técnicas
- [ ] Crear página `/login` con formulario email/password
- [ ] Crear página `/registro` con formulario de nuevo candidato
- [ ] Crear página `/magic-link` para solicitar acceso sin password
- [ ] Crear página `/magic-link/verificar/{token}` para validar token
- [ ] Implementar validaciones en frontend (email, password strength)
- [ ] Mostrar indicadores de carga durante peticiones
- [ ] Manejar errores de API con mensajes user-friendly
- [ ] Implementar "Recordar sesión" (refresh token en localStorage seguro)
- [ ] Añadir enlace "¿Olvidaste tu contraseña?" → magic link
- [ ] Diseño responsive mobile-first (breakpoints: 320px, 768px, 1024px)

#### Wireframe de Login

```
┌─────────────────────────────────────┐
│         🏢 Portal LTI               │
├─────────────────────────────────────┤
│                                     │
│   Accede a tu cuenta                │
│                                     │
│   ┌─────────────────────────────┐   │
│   │ 📧 Email                    │   │
│   └─────────────────────────────┘   │
│   ┌─────────────────────────────┐   │
│   │ 🔒 Contraseña               │   │
│   └─────────────────────────────┘   │
│                                     │
│   [     Iniciar Sesión      ]       │
│                                     │
│   ─────────── o ───────────         │
│                                     │
│   [ 🔗 Enviarme un Magic Link ]     │
│                                     │
│   ¿No tienes cuenta? Regístrate     │
│                                     │
└─────────────────────────────────────┘
```

#### Criterios de Aceptación del Ticket
- [ ] Formularios validan en tiempo real (feedback visual)
- [ ] Botones deshabilitados durante carga (evitar doble submit)
- [ ] Errores de API se muestran claramente (ej: "Email no registrado")
- [ ] Funciona en Chrome, Firefox, Safari, Edge (últimas 2 versiones)
- [ ] Lighthouse score >90 en mobile
- [ ] Tests de componentes con React Testing Library

#### Dependencias
- TK-003 (API de autenticación disponible)
- Sistema de diseño/componentes base del proyecto

#### Estimación
**17 horas**

---

### TK-007: Frontend - Dashboard de Postulaciones

#### Descripción
Implementar la pantalla principal del portal donde el candidato ve todas sus postulaciones con su estado actual.

#### Tareas Técnicas
- [ ] Crear página `/dashboard` como página principal post-login
- [ ] Implementar componente `PostulacionCard` con:
  - Logo/nombre de empresa
  - Título de vacante
  - Etapa actual con indicador visual (badge de color)
  - Barra de progreso del pipeline
  - Fecha de última actualización
  - Botón "Ver detalle"
- [ ] Implementar lista de postulaciones con scroll infinito o paginación
- [ ] Añadir estados vacíos ("No tienes postulaciones activas")
- [ ] Implementar skeleton loading mientras carga
- [ ] Añadir filtro por estado (Todas / Activas / Finalizadas)
- [ ] Ordenar por fecha de actualización (más reciente primero)
- [ ] Diseño responsive con grid adaptativo

#### Componente PostulacionCard

```
┌────────────────────────────────────────────────┐
│ 🏢 TechCorp                                    │
│                                                │
│ Senior Backend Developer                       │
│                                                │
│ Estado: [🟢 Entrevista en curso]              │
│                                                │
│ ●━━━━━━━━●━━━━━━━━○━━━━━━━━○                  │
│ Recibido  Entrevista  Oferta  Contratado      │
│                                                │
│ 📅 Actualizado hace 2 días     [Ver detalle →]│
└────────────────────────────────────────────────┘
```

#### Criterios de Aceptación del Ticket
- [ ] Página carga en menos de 2 segundos
- [ ] Barra de progreso refleja correctamente la etapa actual
- [ ] Colores de badge consistentes: Verde (avanzando), Amarillo (espera), Gris (finalizado)
- [ ] Empty state con CTA si no hay postulaciones
- [ ] Responsive: 1 columna en mobile, 2 columnas en tablet, 3 en desktop
- [ ] Tests de componentes implementados

#### Dependencias
- TK-001 (API de postulaciones)
- TK-006 (Autenticación funcionando)

#### Estimación
**14 horas**

---

### TK-008: Frontend - Detalle de Postulación y Entrevistas

#### Descripción
Implementar la página de detalle donde el candidato ve información completa de una postulación específica, incluyendo próximas entrevistas.

#### Tareas Técnicas
- [ ] Crear página `/postulacion/{id}` con detalle completo
- [ ] Mostrar información de la vacante:
  - Título, empresa, ubicación
  - Descripción del puesto
  - Requisitos (si disponibles)
- [ ] Implementar timeline visual del proceso:
  - Etapas completadas (✓)
  - Etapa actual (●)
  - Etapas pendientes (○)
  - Fechas de cada transición
- [ ] Sección "Próximas Entrevistas" con:
  - Fecha y hora formateada (ej: "Lunes 20 Ene, 10:00 AM")
  - Tipo de entrevista
  - Botón prominente "Unirse a videollamada"
  - Opción "Añadir a calendario" (Google Calendar, Outlook, ICS)
- [ ] Mostrar mensaje personalizado según etapa
- [ ] Botón "Volver al dashboard"
- [ ] Manejar estado de postulación no encontrada (404)

#### Layout del Detalle

```
┌─────────────────────────────────────────────────────┐
│ ← Volver                                            │
├─────────────────────────────────────────────────────┤
│                                                     │
│ 🏢 TechCorp                                         │
│ Senior Backend Developer                            │
│ 📍 Madrid, España                                   │
│                                                     │
│ ─────────────────────────────────────────────────── │
│                                                     │
│ TU PROGRESO                                         │
│                                                     │
│ ✓ Recibido ─── ✓ Revisión ─── ● Entrevista ─── ○   │
│   10 Ene         12 Ene         15 Ene             │
│                                                     │
│ 💬 "¡Felicidades! Has avanzado a la fase de        │
│     entrevistas. Prepárate para conocer al equipo." │
│                                                     │
│ ─────────────────────────────────────────────────── │
│                                                     │
│ 📅 PRÓXIMA ENTREVISTA                               │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Entrevista Técnica                              │ │
│ │ 📆 Lunes 20 Enero, 10:00 AM (GMT+1)            │ │
│ │ ⏱️ Duración: 60 minutos                         │ │
│ │                                                 │ │
│ │ [🎥 Unirse a Google Meet]  [📅 Añadir a cal.]  │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### Criterios de Aceptación del Ticket
- [ ] Timeline muestra correctamente el progreso
- [ ] Enlace de videollamada abre en nueva pestaña
- [ ] "Añadir a calendario" genera archivo ICS descargable
- [ ] Fechas formateadas según locale del navegador
- [ ] Página 404 si postulación no existe o no pertenece al candidato
- [ ] Tests de integración con API mockeada

#### Dependencias
- TK-001, TK-002 (APIs de postulaciones y entrevistas)
- TK-007 (Navegación desde dashboard)

#### Estimación
**17 horas**

---

### TK-009: Frontend - Página de Perfil y Preferencias

#### Descripción
Implementar la página donde el candidato puede actualizar sus datos de contacto y configurar preferencias de notificación.

#### Tareas Técnicas
- [ ] Crear página `/perfil` accesible desde header/menú
- [ ] Sección "Datos Personales":
  - Nombre completo (editable)
  - Email (solo lectura, mostrar verificado ✓)
  - Teléfono (editable, con validación)
- [ ] Sección "Preferencias de Notificación":
  - Toggle: Notificaciones por email (on/off)
  - Toggle: Notificaciones por SMS (on/off)
  - Checkbox: Tipos de notificación (cambio estado, recordatorios, ofertas)
- [ ] Sección "Seguridad":
  - Botón "Cambiar contraseña" → modal con formulario
  - Botón "Cerrar sesión en todos los dispositivos"
- [ ] Implementar endpoint `PUT /api/candidato/perfil` 
- [ ] Implementar endpoint `PUT /api/candidato/preferencias`
- [ ] Feedback visual al guardar (toast de confirmación)
- [ ] Validación de teléfono formato E.164

#### Layout del Perfil

```
┌─────────────────────────────────────────────────────┐
│ ⚙️ Mi Perfil                                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│ DATOS PERSONALES                                    │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Nombre:    [Juan Pérez García        ]          │ │
│ │ Email:     juan.perez@email.com ✓               │ │
│ │ Teléfono:  [+34 612 345 678          ]          │ │
│ └─────────────────────────────────────────────────┘ │
│                                         [Guardar]   │
│                                                     │
│ ─────────────────────────────────────────────────── │
│                                                     │
│ PREFERENCIAS DE NOTIFICACIÓN                        │
│ ┌─────────────────────────────────────────────────┐ │
│ │ 📧 Email                              [====●]   │ │
│ │ 📱 SMS                                [●====]   │ │
│ │                                                 │ │
│ │ Notificarme sobre:                              │ │
│ │ ☑️ Cambios en mis postulaciones                 │ │
│ │ ☑️ Recordatorios de entrevistas                 │ │
│ │ ☐ Nuevas vacantes similares                     │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ ─────────────────────────────────────────────────── │
│                                                     │
│ SEGURIDAD                                           │
│ [🔒 Cambiar contraseña]  [🚪 Cerrar todas sesiones]│
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### Criterios de Aceptación del Ticket
- [ ] Cambios se guardan correctamente en backend
- [ ] Toast de confirmación "Cambios guardados" al éxito
- [ ] Validación de teléfono con mensaje de error claro
- [ ] Toggle de SMS deshabilitado si no hay teléfono registrado
- [ ] Cambiar contraseña requiere contraseña actual
- [ ] Tests de formularios y validaciones

#### Dependencias
- TK-003 (Autenticación)
- TK-004, TK-005 (Preferencias afectan envío de notificaciones)

#### Estimación
**12 horas**

---

### TK-010: Testing E2E y Optimización de Rendimiento

#### Descripción
Implementar pruebas end-to-end del flujo completo del portal y optimizaciones para cumplir el requisito de carga <3 segundos.

#### Tareas Técnicas

**Testing E2E:**
- [ ] Configurar Cypress o Playwright para E2E
- [ ] Test: Flujo de registro completo
- [ ] Test: Login con email/password
- [ ] Test: Login con magic link
- [ ] Test: Ver dashboard de postulaciones
- [ ] Test: Ver detalle de postulación
- [ ] Test: Actualizar perfil y preferencias
- [ ] Test: Recibir notificación al cambiar estado (mock)
- [ ] Test: Responsive en viewport mobile (375px)

**Optimización de Rendimiento:**
- [ ] Configurar Redis para cache de sesiones JWT
- [ ] Implementar cache de datos de postulaciones (TTL: 5 min)
- [ ] Lazy loading de componentes pesados (React.lazy)
- [ ] Optimizar imágenes (WebP, lazy loading)
- [ ] Implementar compresión gzip en respuestas API
- [ ] Code splitting por rutas
- [ ] Medir y optimizar Core Web Vitals (LCP, FID, CLS)

**Monitoreo:**
- [ ] Añadir métricas de tiempo de respuesta en API
- [ ] Configurar alertas si latencia >500ms
- [ ] Dashboard de métricas (opcional: Grafana/DataDog)

#### Criterios de Aceptación del Ticket
- [ ] Todos los tests E2E pasan en CI/CD
- [ ] Tiempo de carga inicial (LCP) <2.5 segundos
- [ ] Time to Interactive (TTI) <3 segundos
- [ ] API responses p95 <300ms
- [ ] Lighthouse Performance score >90
- [ ] Tests E2E cubren flujos críticos (login, ver postulaciones)

#### Dependencias
- Todos los tickets anteriores completados (TK-001 a TK-009)

#### Estimación
**19 horas**

---

### Resumen de Tickets

| ID | Título | Tipo | Dependencias | Sprint Sugerido |
|----|--------|------|--------------|-----------------|
| TK-001 | API Postulaciones | Backend | TK-003 | Sprint 1 |
| TK-002 | API Entrevistas | Backend | TK-003 | Sprint 1 |
| TK-003 | Autenticación Candidatos | Backend | - | Sprint 1 |
| TK-004 | Notificaciones Email | Backend | TK-003 | Sprint 2 |
| TK-005 | Notificaciones SMS | Backend | TK-004 | Sprint 2 |
| TK-006 | UI Login/Registro | Frontend | TK-003 | Sprint 1 |
| TK-007 | UI Dashboard | Frontend | TK-001, TK-006 | Sprint 2 |
| TK-008 | UI Detalle Postulación | Frontend | TK-001, TK-002 | Sprint 2 |
| TK-009 | UI Perfil/Preferencias | Frontend | TK-003 | Sprint 2 |
| TK-010 | Testing & Performance | QA/DevOps | Todos | Sprint 3 |

---

## Estimaciones de Esfuerzo

### Metodología de Estimación

Se utilizó **estimación en horas** considerando:
- Desarrollador de nivel medio-senior
- Incluye: desarrollo, code review, testing unitario y documentación
- No incluye: reuniones, despliegue a producción, bugs post-release
- Factor de contingencia: +20% sobre estimación base

### Tabla de Estimaciones

| ID | Ticket | Estimación Base | Contingencia (+20%) | Total |
|----|--------|-----------------|---------------------|-------|
| TK-001 | API Postulaciones | 10h | 2h | **12h** |
| TK-002 | API Entrevistas | 6h | 1h | **7h** |
| TK-003 | Autenticación Candidatos | 16h | 3h | **19h** |
| TK-004 | Notificaciones Email | 12h | 2h | **14h** |
| TK-005 | Notificaciones SMS | 8h | 2h | **10h** |
| TK-006 | UI Login/Registro | 14h | 3h | **17h** |
| TK-007 | UI Dashboard | 12h | 2h | **14h** |
| TK-008 | UI Detalle Postulación | 14h | 3h | **17h** |
| TK-009 | UI Perfil/Preferencias | 10h | 2h | **12h** |
| TK-010 | Testing & Performance | 16h | 3h | **19h** |
| | | | **TOTAL** | **141h** |

### Justificación de Estimaciones

#### TK-001: API Postulaciones (12h)
| Tarea | Horas |
|-------|-------|
| Diseño de endpoints y estructura de respuesta | 1h |
| Implementación GET /postulaciones (lista) | 3h |
| Implementación GET /postulaciones/{id} (detalle) | 2h |
| Queries SQL con JOINs optimizados | 2h |
| Tests unitarios | 2h |
| Documentación Swagger | 1h |
| Code review y ajustes | 1h |

#### TK-002: API Entrevistas (7h)
| Tarea | Horas |
|-------|-------|
| Diseño de endpoint y modelo de datos | 1h |
| Implementación GET /entrevistas | 2h |
| Filtros y ordenamiento | 1h |
| Tests unitarios | 2h |
| Documentación | 1h |

#### TK-003: Autenticación Candidatos (19h)
| Tarea | Horas |
|-------|-------|
| Diseño del sistema de auth y tabla BD | 2h |
| Endpoint registro con validaciones | 2h |
| Endpoint login + generación JWT | 3h |
| Sistema de magic link (generación + validación) | 4h |
| Refresh token y logout | 2h |
| Middleware de validación JWT | 2h |
| Tests de seguridad | 3h |
| Documentación | 1h |

#### TK-004: Notificaciones Email (14h)
| Tarea | Horas |
|-------|-------|
| Configuración SendGrid SDK | 1h |
| Diseño de templates (4 templates) | 4h |
| Servicio NotificationEmailService | 3h |
| Cola de mensajes (Redis/Bull) | 3h |
| Tabla de logs y retry logic | 2h |
| Tests con mocks | 1h |

#### TK-005: Notificaciones SMS (10h)
| Tarea | Horas |
|-------|-------|
| Configuración Twilio SDK | 1h |
| Templates SMS (2 templates) | 1h |
| Servicio NotificationSMSService | 2h |
| Validación E.164 y horarios | 2h |
| Integración con cola existente | 2h |
| Opt-out handling | 1h |
| Tests | 1h |

#### TK-006: UI Login/Registro (17h)
| Tarea | Horas |
|-------|-------|
| Setup de rutas y navegación | 1h |
| Página de Login | 4h |
| Página de Registro | 4h |
| Flujo Magic Link (2 páginas) | 3h |
| Validaciones frontend | 2h |
| Manejo de estados y errores | 2h |
| Tests de componentes | 1h |

#### TK-007: UI Dashboard (14h)
| Tarea | Horas |
|-------|-------|
| Componente PostulacionCard | 4h |
| Lista con paginación/scroll infinito | 3h |
| Filtros y ordenamiento | 2h |
| Estados vacíos y loading | 2h |
| Responsive design | 2h |
| Tests | 1h |

#### TK-008: UI Detalle Postulación (17h)
| Tarea | Horas |
|-------|-------|
| Layout de página de detalle | 3h |
| Componente Timeline de progreso | 4h |
| Sección de entrevistas | 3h |
| Generador de archivo ICS | 2h |
| Mensajes personalizados por etapa | 2h |
| Manejo de errores (404) | 1h |
| Responsive y tests | 2h |

#### TK-009: UI Perfil/Preferencias (12h)
| Tarea | Horas |
|-------|-------|
| Formulario datos personales | 3h |
| Toggles de preferencias | 2h |
| Modal cambiar contraseña | 2h |
| Endpoints PUT perfil/preferencias | 2h |
| Validaciones y feedback | 2h |
| Tests | 1h |

#### TK-010: Testing & Performance (19h)
| Tarea | Horas |
|-------|-------|
| Setup Cypress/Playwright | 2h |
| Tests E2E (8 flujos críticos) | 8h |
| Configuración cache Redis | 2h |
| Optimizaciones frontend (lazy load, code split) | 3h |
| Optimización de imágenes | 1h |
| Métricas y monitoreo | 2h |
| Documentación de resultados | 1h |

### Distribución por Rol

| Rol | Tickets | Horas Totales |
|-----|---------|---------------|
| **Backend Developer** | TK-001, TK-002, TK-003, TK-004, TK-005 | 62h |
| **Frontend Developer** | TK-006, TK-007, TK-008, TK-009 | 60h |
| **QA / DevOps** | TK-010 | 19h |
| | **TOTAL** | **141h** |

### Planificación Sugerida (2 desarrolladores)

Asumiendo 1 backend + 1 frontend trabajando en paralelo:

```
SPRINT 1 (40h/persona = 1 semana)
├── Backend: TK-003 (19h) + TK-001 (12h) + TK-002 (7h) = 38h ✓
└── Frontend: TK-006 (17h) + inicio TK-007 = 40h ✓

SPRINT 2 (40h/persona = 1 semana)  
├── Backend: TK-004 (14h) + TK-005 (10h) + soporte = 24h ✓
└── Frontend: TK-007 (14h) + TK-008 (17h) + TK-009 (12h) = 43h ⚠️

SPRINT 3 (20h = 0.5 semana)
└── QA/Ambos: TK-010 (19h) = 19h ✓

TOTAL ESTIMADO: 2.5 semanas con 2 desarrolladores
```

### Riesgos Identificados

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Integración SendGrid/Twilio con problemas de configuración | +4-8h | Tener cuentas configuradas antes de Sprint 2 |
| Complejidad del sistema de magic link | +4h | Usar librería probada (ej: next-auth) |
| Performance <3s difícil de alcanzar | +8h | Priorizar cache desde el inicio |
| Tests E2E flaky | +4h | Invertir en fixtures estables |

---

*Documento generado como parte del ejercicio AI4Devs - Módulo de User Stories*
