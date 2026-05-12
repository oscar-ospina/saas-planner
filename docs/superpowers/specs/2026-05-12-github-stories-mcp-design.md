# Gestión de user stories en GitHub con MCP desde Claude Code

**Fecha:** 2026-05-12
**Autor:** brainstorming session con Claude Code
**Estado:** Aprobado para planificación

## Contexto y objetivo

Configurar un sistema de gestión de user stories que permita trabajar end-to-end desde Claude Code, usando solo herramientas gratuitas. El sistema debe servir a un único desarrollador inicialmente y escalar a un equipo pequeño (2-5 personas) sin migración.

**Decisiones previas (brainstorm):**

- Plataforma: GitHub (Issues + Projects v2) — el usuario ya vive en GitHub
- Jerarquía: 2 niveles (Epic → Story) usando sub-issues nativos
- Uso de MCP: bidireccional (crear stories durante brainstorm + leer/actualizar al implementar)

## Arquitectura

```
Claude Code ←→ GitHub MCP ←→ GitHub API ←→ Repo + Project v2
```

**Componentes:**

1. **Repo GitHub** — código + plantillas de issues
2. **GitHub Project v2** — board con vistas (Backlog, Sprint, Roadmap) y campos custom
3. **Issue templates** — `epic.yml` y `story.yml` con AC en formato Given/When/Then
4. **GitHub MCP server oficial** — instalado en Claude Code, autenticado con PAT fine-grained
5. **Convención de labels** — `epic`, `story`, `bug`, `priority:*`, `status:*`

**Por qué GitHub vs alternativas (Linear, Notion):** evita duplicar mundos (código vs stories), tier gratis sin límite de issues, escala sin migración cuando llegue el equipo.

## Setup del repo

- **Nombre:** a definir antes de implementación (placeholder: `<repo-name>`)
- **Visibilidad:** privado al inicio
- **Inicialización:** README, `.gitignore`, LICENSE, carpeta `.github/ISSUE_TEMPLATE/`

## Setup del Project v2

- **Vista default:** Board (Kanban)
- **Vistas adicionales:**
  - Backlog (tabla, filtra `status:ready`, ordena por priority)
  - Sprint actual (board, agrupa por status)
  - Roadmap (timeline, opcional)
- **Campos custom:**
  - `Priority` — single-select: high / med / low
  - `Story Points` — number, escala Fibonacci (1/2/3/5/8)
  - `Sprint` — iteration field, ciclos de 2 semanas (default; ajustable)
  - `Type` — single-select: epic / story / bug / spike
- **Sub-issues:** activados (feature nativa GitHub 2024+)

## Plantillas de issues

### `.github/ISSUE_TEMPLATE/epic.yml`

```yaml
name: Epic
description: Iniciativa grande que agrupa varias stories
title: "[Epic] "
labels: ["epic"]
body:
  - type: textarea
    id: goal
    attributes:
      label: Objetivo
      description: Qué problema resuelve, para quién, por qué importa
    validations: { required: true }
  - type: textarea
    id: success
    attributes:
      label: Criterios de éxito
      description: Cómo sabemos que la épica está completa (métricas o entregables)
    validations: { required: true }
  - type: textarea
    id: stories
    attributes:
      label: Stories planeadas
      description: Lista inicial (se convertirán en sub-issues)
```

### `.github/ISSUE_TEMPLATE/story.yml`

```yaml
name: User Story
description: Historia accionable, implementable en 1 sprint
title: "[Story] "
labels: ["story"]
body:
  - type: textarea
    id: story
    attributes:
      label: Historia
      description: "Como [rol], quiero [acción], para [beneficio]"
    validations: { required: true }
  - type: textarea
    id: ac
    attributes:
      label: Criterios de aceptación
      description: |
        Formato Given/When/Then:
        - Given [contexto], When [acción], Then [resultado esperado]
    validations: { required: true }
  - type: textarea
    id: notes
    attributes:
      label: Notas técnicas
      description: Constraints, dependencias, decisiones
  - type: input
    id: epic
    attributes:
      label: Epic padre (opcional)
      description: "#número o URL del epic"
```

**Por qué Given/When/Then:** estructurado, testeable, Claude puede generar tests directamente desde los AC.

## Setup del MCP

```bash
# 1. Crear PAT fine-grained en github.com/settings/personal-access-tokens
#    Permisos requeridos:
#      - Issues (Read & Write)
#      - Pull requests (Read & Write)
#      - Contents (Read & Write)
#      - Projects (Read & Write)
#      - Metadata (Read)

# 2. Agregar MCP a Claude Code
claude mcp add github -- npx -y @modelcontextprotocol/server-github
# Configurar variable: GITHUB_PERSONAL_ACCESS_TOKEN=<tu-pat>
```

## Workflows desde Claude Code

### A) Crear story durante brainstorm

```
Usuario: "Vamos a brainstorm una feature de login con Google"
Claude:  [conversación brainstorming]
         [propone story con AC]
         [crea issue via MCP usando template story.yml]
         → devuelve link al issue
```

### B) Implementar story asignada

```
Usuario: "Trabajemos en #42"
Claude:  [lee issue #42 via MCP]
         [lee AC y notas técnicas]
         [implementa]
         [actualiza status a in-progress al empezar]
         [abre PR linkeado al issue con "Closes #42"]
```

### C) Planificación de Epic

```
Usuario: "Necesito una épica para sistema de notificaciones"
Claude:  [crea epic via MCP]
         [propone 4-6 stories sub-issues]
         [tras OK del usuario, crea las sub-issues]
         [agrega al Project en columna Backlog]
```

**Convención de commits/PRs:** `feat: <título story> (#42)`. El linkeo `Closes #N` cierra el issue al merge automáticamente.

## Validación post-setup

Test manual en orden, sin intervención fuera de lo descrito:

1. **MCP responde:** pedir a Claude `lista mis repos`. Debe devolver lista → MCP+PAT OK
2. **Templates cargan:** ir a `github.com/<user>/<repo>/issues/new/choose` → ver "Epic" y "User Story"
3. **Crear epic vía MCP:** Claude crea epic de prueba. Verificar label correcto y campos del Project en UI
4. **Crear story con sub-issue:** Claude crea story bajo el epic. Verificar relación parent/child en UI
5. **Cerrar via PR:** branch + commit `Closes #N` + PR + merge. Issue se cierra solo y se mueve a "Done"
6. **Limpieza:** borrar issues de prueba

**Criterio de éxito:** los 6 pasos completan sin error.

## Riesgos conocidos

- **PAT expira** (fine-grained máx 1 año) → poner recordatorio en calendario para rotación
- **Rate limit GitHub API** — 5000 req/h con PAT. Suficiente para uso solo; revisar si llega equipo
- **Sub-issues feature relativamente nueva** — si la UI/API cambia, ajustar templates y workflow

## Fuera de alcance (YAGNI)

Lo siguiente queda explícitamente fuera y se considerará en futuros specs si surge necesidad real:

- Automatización de CI/CD (GitHub Actions más allá de las default)
- Integración con herramientas externas (Slack, Discord, email)
- Dashboards de métricas/velocity
- Multi-repo / monorepo
- Roles y permisos finos en el Project (escalable luego al invitar equipo)

## Decisiones pendientes para fase de planificación

- Nombre real del repo
- Duración de sprint (1 vs 2 semanas)
- Story points vs t-shirt sizes (XS/S/M/L)
- Visibilidad final del repo (privado fijo, o flip a público en algún criterio)
