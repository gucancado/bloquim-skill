# Bloquim MCP — Catálogo de tools

Referência completa das **35 tools** do MCP `bloquim`. Carregue sob demanda quando
precisar do schema exato de uma tool. Para a postura geral e workflows, veja `../SKILL.md`.

Convenção de leitura: **req** = obrigatório, **opt** = opcional. Tipos `uuid` são
strings UUID. Datas sempre `YYYY-MM-DD`.

---

## Auth (1)

| Tool | Propósito | Params |
|---|---|---|
| `whoami` | Valida a autenticação e retorna o usuário Bloquim atual. | (nenhum) |

## Tarefas — leitura (6)

| Tool | Propósito | Params |
|---|---|---|
| `list_my_tasks` | Lista as tarefas do usuário (standalone + workspaces). | `scope` (today\|week\|all, opt, **def today**), `status` (array, opt; aceita `overdue`), `workspaceId` (uuid, opt) |
| `list_workspace_tasks` | Lista tarefas de um workspace. | `workspaceId` (uuid, req), `assignedTo` (array de uuid e/ou `unassigned`, opt), `status` (array, opt; aceita `overdue`), `scope` (today\|week\|all, opt, **def all**) |
| `get_task` | Detalhes completos de uma tarefa (roteia standalone/workspace automaticamente). | `taskId` (uuid, req) |
| `search_tasks` | Busca tarefas por texto em título+descrição. **Máx 20 resultados.** | `query` (string ≥2, req), `workspaceId` (uuid, opt), `status` (array, opt) |
| `list_task_comments` | Lista comentários de uma tarefa. | `taskId` (uuid, req) |
| `get_task_activity` | Histórico de atividades da tarefa. | `taskId` (uuid, req) |

## Tarefas — criar & editar (5)

| Tool | Propósito | Params |
|---|---|---|
| `create_task` | Cria tarefa (standalone, workspace ou plano). Sai `pending` por default em todos os modos. ⚠️ `dueDate`/`startAt` sem `scheduleMode` são descartados (default `sem_prazo`). Em modo plano (`planId`), `priority`/`scheduleMode`/datas/`assignee` são rejeitados. | `title` (string, req), `description` (markdown, opt), `workspaceId` (uuid, opt), `planId` (uuid, opt), `assignee` ({userId}\|{email}\|null, opt), `scheduleMode` (ate\|entre\|em\|sem_prazo\|urgente, opt), `startAt` (YYYY-MM-DD, opt), `dueDate` (YYYY-MM-DD, opt), `priority` (low\|medium\|high\|critical, opt), `status` (pending\|in_progress\|completed\|blocked\|draft, opt) |
| `update_task` | Edita campos da tarefa. `description`/`startAt`/`dueDate` aceitam `null` para limpar. | `taskId` (uuid, req), `title` (opt), `description` (markdown\|null, opt), `priority` (opt), `scheduleMode` (opt), `startAt` (YYYY-MM-DD\|null, opt), `dueDate` (YYYY-MM-DD\|null, opt) |
| `set_task_status` | Muda o status. `overdue` é derivado (não settável). | `taskId` (uuid, req), `status` (pending\|in_progress\|completed\|blocked\|draft, req) |
| `set_task_assignee` | Reatribui a tarefa. **Só workspace tasks.** | `taskId` (uuid, req), `assignee` ({userId}\|{email}\|null, req) |
| `set_task_schedule` | Ajusta a modalidade de prazo + datas. | `taskId` (uuid, req), `mode` (ate\|entre\|em\|sem_prazo\|urgente, req), `startAt` (YYYY-MM-DD\|null, opt), `dueDate` (YYYY-MM-DD\|null, opt) |

## Tarefas — deletar & mover (2)

| Tool | Propósito | Params |
|---|---|---|
| `delete_task` | Apaga a tarefa (cascateia subtasks, comentários, anexos). | `taskId` (uuid, req), `confirm` (true literal, req) |
| `move_task_to_workspace` | Move standalone → workspace. | `taskId` (uuid, req), `targetWorkspaceId` (uuid, req), `assignee` ({userId}\|{email}\|null, opt) |

## Comentários (1)

| Tool | Propósito | Params |
|---|---|---|
| `add_task_comment` | Adiciona comentário à tarefa. | `taskId` (uuid, req), `content` (string ≥1, req) |

## Checklist / subtasks (5)

| Tool | Propósito | Params |
|---|---|---|
| `list_task_checklist` | Lista itens do checklist (total/completed/pending + items[]). | `taskId` (uuid, req) |
| `add_checklist_items` | Adiciona 1..50 itens (atômico). | `taskId` (uuid, req), `items` (array[{text, completed?, order?}], 1..50, req) |
| `update_checklist_item` | Edita um item. | `taskId` (uuid, req), `subtaskId` (uuid, req), `text` (opt), `completed` (bool, opt), `order` (int, opt) |
| `delete_checklist_item` | Remove um item. | `taskId` (uuid, req), `subtaskId` (uuid, req) |
| `clear_task_checklist` | Apaga TODOS os itens. | `taskId` (uuid, req), `confirm` (true literal, req) |

## Anexos (3)

| Tool | Propósito | Params |
|---|---|---|
| `add_task_attachment` | Anexa arquivo (≤10MB). Forneça `contentBase64` **XOR** `sourceUrl`. | `taskId` (uuid, req), `filename` (string 1..255, req), `mimeType` (string, req), `contentBase64` (base64 ≤10MB, opt), `sourceUrl` (url http(s), opt), `kind` (standard\|deliverable, opt, def standard) |
| `list_task_attachments` | Lista anexos da tarefa. | `taskId` (uuid, req) |
| `remove_task_attachment` | Remove anexo (soft-delete). | `taskId` (uuid, req), `attachmentId` (uuid, req) |

## Planos / mapas de ação (8)

| Tool | Propósito | Params |
|---|---|---|
| `list_plans` | Lista planos do workspace. | `workspaceId` (uuid, req), `includeHidden` (bool, opt) |
| `get_plan` | Detalhes do plano (tasks + dependências). | `workspaceId` (uuid, req), `planId` (uuid, req) |
| `create_plan` | Cria plano vazio. | `workspaceId` (uuid, req), `name` (string ≥1, req) |
| `update_plan` | Renomeia o plano. | `workspaceId` (uuid, req), `planId` (uuid, req), `name` (string ≥1, req) |
| `delete_plan` | Apaga o plano (cascateia tasks vinculadas). | `workspaceId` (uuid, req), `planId` (uuid, req), `confirm` (true literal, req) |
| `search_plans` | Busca planos por nome (ilike). **Máx 50.** Cross-workspace se `workspaceId` omitido. | `query` (string ≥1, req), `workspaceId` (uuid, opt) |
| `attach_task_to_plan` | Anexa task existente (sem plano) a um plano. | `workspaceId` (uuid, req), `planId` (uuid, req), `taskId` (uuid, req) |
| `detach_task_from_plan` | Remove task do plano (volta standalone-workspace). | `workspaceId` (uuid, req), `planId` (uuid, req), `taskId` (uuid, req) |

## Workspaces (4)

| Tool | Propósito | Params |
|---|---|---|
| `list_workspaces` | Lista workspaces do usuário. | `includeHidden` (bool, opt) |
| `list_workspace_members` | Lista membros (userId/name/email/role/classes). | `workspaceId` (uuid, req) |
| `create_workspace` | Cria workspace (caller vira admin). | `name` (string ≥1, req) |
| `add_workspace_member` | Adiciona membro por email (role fixo `editor`). | `workspaceId` (uuid, req), `email` (email, req) |

> **Não há `delete_workspace` no MCP** — criar workspace é one-way; apagar só pela UI do Bloquim.

**Total: 35 tools** — Auth 1, Tasks-read 6, Tasks-write 5, Delete/move 2, Comments 1, Checklist 5, Attachments 3, Plans 8, Workspaces 4.

---

## Roteamento automático standalone ↔ workspace

Tools que operam por `taskId` (get/update/status/assignee/schedule/comentários/atividade/checklist/anexos/delete)
resolvem a rota sozinhas via `resolveTaskRoute(taskId)` → `{taskId, workspaceId}`. Você **não** precisa
saber se a tarefa é standalone (Minhas Tarefas) ou de workspace — passe só o `taskId`.

## Convenções

### priority
`low` · `medium` (default) · `high` · `critical`.
Inferência: "urgente/pra já" → `critical`; "importante/prioritário" → `high`; default → `medium`.

### scheduleMode (modalidade de prazo) + datas
Datas em `YYYY-MM-DD` (campo DATE, não timestamp).
⚠️ **Datas exigem `scheduleMode`**: em `create_task`, passar `dueDate`/`startAt` sem `scheduleMode` faz as datas serem **silenciosamente descartadas** (default `sem_prazo`).

| Modo | Semântica | Datas exigidas |
|---|---|---|
| `ate` | prazo final | só `dueDate` |
| `entre` | janela | `startAt` **e** `dueDate` |
| `em` | dia pontual | `startAt` = `dueDate` (ambos) |
| `sem_prazo` | sem prazo | nenhuma |
| `urgente` | atenção imediata; **pinada no topo de toda lista**, ignora filtro de scope | nenhuma |

Inferência: prazo único → `ate`; dia pontual → `em`; janela → `entre`; sem data → `sem_prazo`;
"urgente" sem data → `urgente`.

### status
Settáveis: `pending` · `in_progress` · `completed` · `blocked` · `draft`.
**`overdue` é derivado** (calculado quando `dueDate < hoje`) — não settável, mas **filtrável** no `status` de `list_my_tasks`/`list_workspace_tasks`.
O backend cria toda tarefa como `draft`; a tool já faz o PATCH para `pending` (ou para o `status` passado) em **todos os modos**, inclusive cards de plano. Em modo plano o que é ignorado são `priority`/`scheduleMode`/datas/`assignee` (rejeitados na chamada — ajuste via `update_task`/`set_task_*` depois).
⚠️ Cards de plano saem com `createdBy: null`. `delete_task` neles só é rejeitado se o caller **não** for criador nem admin do workspace — admin tem bypass e o delete passa. Pra limpar um plano inteiro prefira `delete_plan` (cascade); pra preservar tarefas, `detach_task_from_plan`.
⚠️ **Bug de backend conhecido:** `set_task_schedule` (ou `update_task` com `scheduleMode`+datas) num **card de plano** retorna **HTTP 500** e pode correlacionar com perda do card/plano. Em card de plano, hoje só ajuste `priority`/`title`/`description` via `update_task`. Pra agendar, crie a task standalone-de-workspace com o prazo e use `attach_task_to_plan`.

### tipos de tarefa
- **Standalone** (Minhas Tarefas): omita `workspaceId`. Assignee = caller fixo (não customizável; `set_task_assignee` não se aplica).
- **Workspace**: passe `workspaceId`. Assignee customizável.
- **Plan-attached**: passe `planId`. Nasce `draft`, priority `medium`, scheduleMode `sem_prazo`, assignee = caller — edite depois via `update_task`/`set_task_*`.

### assignee (só workspace)
- `{userId: "<uuid>"}` — quando se sabe o ID.
- `{email: "<email>"}` — resolve entre membros do workspace (case-insensitive). 404 se não for membro.
- `null` — desatribuir.
- omitir — default = caller.

### operações destrutivas
`delete_task`, `clear_task_checklist`, `delete_plan` exigem `confirm: true` literal.

### limites
`search_tasks` máx 20 (refine a query se truncar) · `search_plans` máx 50 · `add_checklist_items` 1..50 ·
anexo ≤10MB, `contentBase64` **XOR** `sourceUrl`.
