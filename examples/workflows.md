# Workflows — exemplos

Transcrições curtas dos fluxos-âncora. `→` indica chamada de tool. Os IDs são ilustrativos.

---

## 1. Capturar tarefa da conversa (preview-then-confirm)

> **User:** acabei de falar com o cliente, preciso mandar a proposta revisada até sexta.

**Claude:** Quer que eu registre no Bloquim: **"Enviar proposta revisada pro cliente"** (alta, até sex 20/06)?

> **User:** isso

→ `create_task` `{ title: "Enviar proposta revisada pro cliente", priority: "high", scheduleMode: "ate", dueDate: "2026-06-20" }`

**Claude:** Registrado em Minhas Tarefas — alta, vence sex 20/06.

*(Note: standalone já sai `pending` por default — não precisa passar `status`. Só cards de plano nascem `draft`.)*

---

## 2. Criar plano + popular cards

> **User:** monta um plano "Lançamento v2" no workspace Produto com: escrever changelog, gravar vídeo demo, agendar post.

→ `list_workspaces` *(acha o workspace "Produto" → `ws-111`)*
→ `create_plan` `{ workspaceId: "ws-111", name: "Lançamento v2" }` → `plan-222`
→ `create_task` `{ workspaceId: "ws-111", planId: "plan-222", title: "Escrever changelog", status: "pending" }`
→ `create_task` `{ workspaceId: "ws-111", planId: "plan-222", title: "Gravar vídeo demo", status: "pending" }`
→ `create_task` `{ workspaceId: "ws-111", planId: "plan-222", title: "Agendar post de lançamento", status: "pending" }`
→ `get_plan` `{ workspaceId: "ws-111", planId: "plan-222" }` *(confere os 3 cards)*

**Claude:** Plano "Lançamento v2" criado com 3 cards. *(Lembrete: cards de plano nascem `draft`/`medium`/`sem_prazo` — ajustei status pra `pending`; me diga prioridades/prazos se quiser.)*

---

## 3. Extrair tarefas de reunião

> **User:** /extrair_tarefas_de_reuniao  *(cola a transcrição da call)*

**Claude:** Identifiquei 3 action items — confirma antes de eu criar?

| # | Título | Responsável | Prazo | Prioridade |
|---|---|---|---|---|
| 1 | Enviar orçamento atualizado | ana@acme.com | até 18/06 | high |
| 2 | Revisar contrato | (você) | até 20/06 | medium |
| 3 | Marcar follow-up | (você) | sem prazo | medium |

> **User:** pode criar

→ `create_task` `{ workspaceId: "...", title: "Enviar orçamento atualizado", assignee: {email: "ana@acme.com"}, scheduleMode: "ate", dueDate: "2026-06-18", priority: "high", status: "pending" }`
→ `create_task` `{ title: "Revisar contrato", scheduleMode: "ate", dueDate: "2026-06-20", status: "pending" }`
→ `create_task` `{ title: "Marcar follow-up", scheduleMode: "sem_prazo", status: "pending" }`

**Claude:** 3 tarefas criadas.

---

## 4. Atualizar status / prioridade / prazo

> **User:** marca a tarefa do changelog como em andamento e sobe a prioridade.

→ `search_tasks` `{ query: "changelog" }` → `task-333`
→ `set_task_status` `{ taskId: "task-333", status: "in_progress" }`
→ `update_task` `{ taskId: "task-333", priority: "high" }`

**Claude:** "Escrever changelog" → em andamento, prioridade alta.

> **User:** na verdade ela vence dia 25.

→ `set_task_schedule` `{ taskId: "task-333", mode: "ate", dueDate: "2026-06-25" }`

---

## 5. Anexar arquivo + comentar

> **User:** anexa esse PDF na tarefa da proposta e comenta que é a versão final.

→ `search_tasks` `{ query: "proposta" }` → `task-444`
→ `add_task_attachment` `{ taskId: "task-444", filename: "proposta-final.pdf", mimeType: "application/pdf", contentBase64: "<...>", kind: "deliverable" }`
→ `add_task_comment` `{ taskId: "task-444", content: "Versão final anexada — pronta pra envio." }`

**Claude:** PDF anexado (como entregável) e comentário adicionado.

*(Alternativa: se o arquivo já está numa URL pública, use `sourceUrl` em vez de `contentBase64` — um XOR o outro.)*
