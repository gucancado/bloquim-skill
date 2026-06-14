---
name: bloquim
description: >-
  Use ao gerenciar tarefas, compromissos, follow-ups, prazos, action items de
  reunião, planos/projetos e revisões de pendências. Dispare quando o usuário
  quiser registrar/criar/anotar algo a fazer ("preciso", "tenho que", "vou",
  "lembrar de", "depois eu"), organizar/atualizar/priorizar tarefas, extrair
  ações de uma reunião, montar um plano de ação com etapas, atribuir trabalho a
  alguém, ou revisar o que está pendente/atrasado. Também ao perceber um próximo
  passo acionável claro na conversa que valha registrar. NÃO dispare em perguntas
  conceituais puras, brainstorm exploratório, ou coisas já concluídas. Requer o
  conector MCP `bloquim` ativo (ver README para conectar).
---

# Bloquim — gestor de tarefas via MCP

O **Bloquim** (MindTask) é um gestor de tarefas, planos e workspaces. Você opera
nele pelas tools do MCP `bloquim`. Esta skill te dá o mapa das tools, as convenções
e os fluxos. **Schema detalhado de cada tool:** `references/tools.md`.
**Transcrições de exemplo:** `examples/workflows.md`.

> Antes de agir, localize: use `list_my_tasks` / `search_tasks` para achar a tarefa
> antes de editar. Nunca invente um `taskId`/`workspaceId` — busque ou liste.

## Mapa das tools (35)

| Categoria | Tools |
|---|---|
| Auth | `whoami` |
| Tarefas — ler | `list_my_tasks`, `list_workspace_tasks`, `get_task`, `search_tasks`, `list_task_comments`, `get_task_activity` |
| Tarefas — escrever | `create_task`, `update_task`, `set_task_status`, `set_task_assignee`, `set_task_schedule` |
| Tarefas — deletar/mover | `delete_task`, `move_task_to_workspace` |
| Comentários | `add_task_comment` |
| Checklist | `list_task_checklist`, `add_checklist_items`, `update_checklist_item`, `delete_checklist_item`, `clear_task_checklist` |
| Anexos | `add_task_attachment`, `list_task_attachments`, `remove_task_attachment` |
| Planos | `list_plans`, `get_plan`, `create_plan`, `update_plan`, `delete_plan`, `search_plans`, `attach_task_to_plan`, `detach_task_from_plan` |
| Workspaces | `list_workspaces`, `list_workspace_members`, `create_workspace`, `add_workspace_member` |

→ Params, enums e roteamento automático standalone↔workspace em `references/tools.md`.

## Convenções essenciais

- **priority:** `low` · `medium` (default) · `high` · `critical`. "urgente"→`critical`, "importante"→`high`.
- **scheduleMode + datas** (datas em `YYYY-MM-DD`): `ate` (só `dueDate`) · `entre` (`startAt`+`dueDate`) · `em` (dia pontual) · `sem_prazo` · `urgente` (sem datas, pinada no topo).
- **status:** `pending`/`in_progress`/`completed`/`blocked`/`draft`. `overdue` é derivado. Standalone/workspace: `create_task` já entrega `pending` por default (a tool faz o PATCH pós-criação). ⚠️ **Cards de plano** (`planId`) nascem `draft` — passe `status` no `create_task` ou chame `set_task_status` depois.
- **Onde a tarefa vive:** sem `workspaceId` → standalone (Minhas Tarefas, assignee = você). Com `workspaceId` → workspace (assignee customizável). Com `planId` → dentro de um plano.
- **assignee** (só workspace): `{userId}` | `{email}` (resolve entre membros) | `null` | omitir (= você).
- **Destrutivas exigem `confirm: true`:** `delete_task`, `clear_task_checklist`, `delete_plan`.

## Captura proativa (referência ao server)

O server `bloquim` **já injeta** a política de captura no handshake (`instructions`) —
não a reescreva aqui. Resumo operacional:

- **Preview-then-confirm.** Quando o usuário sinaliza um compromisso/próximo passo, ofereça
  em **1 linha** já com `title`/`priority`/`dueDate` inferidos. Só chame `create_task`
  **após confirmação** (exceto se ele pediu modo automático). Máx. 1 oferta; se recusar, silêncio.
- **Não** ofereça em perguntas puras, hipóteses ou coisas já feitas.
- **Atalhos (prompts MCP / slash commands)** expostos pelo server: `capturar_tarefa`,
  `extrair_tarefas_de_reuniao`, `revisao_minhas_tarefas`. Prefira-os quando couber.

## Workflows-âncora

Transcrições completas em `examples/workflows.md`. Resumo:

1. **Capturar tarefa da conversa** — detecta compromisso → 1 linha de preview → confirma → `create_task`.
2. **Criar plano + popular cards** — `create_plan` → `create_task` com `planId` (N vezes) → `get_plan` confere.
3. **Extrair tarefas de reunião** — invoque o prompt `extrair_tarefas_de_reuniao` (cole a transcrição) → tabela de action items → confirma em lote → `create_task` ×N.
4. **Atualizar status/prioridade/prazo** — `search_tasks`/`list_my_tasks` localiza → `set_task_status` / `update_task` / `set_task_schedule`.
5. **Anexar + comentar** — `add_task_attachment` (`contentBase64` XOR `sourceUrl`) → `add_task_comment`.

## Anti-padrões

- ❌ Inventar `taskId`/`workspaceId`/tool. Sempre liste/busque primeiro; o toolset é fixo (35 tools).
- ❌ Esquecer que **cards de plano** nascem `draft` (vão parecer "sumidos" de listas filtradas por status ativo) — passe `status` ou PATCHe depois.
- ❌ `set_task_assignee` em tarefa standalone (só workspace).
- ❌ Omitir `confirm: true` em delete/clear (a chamada falha).
- ❌ Esperar paginação em `search_tasks` (trunca em 20 — refine a query).
- ❌ Criar tarefa sem confirmação quando o usuário só comentou algo (respeite preview-then-confirm).
