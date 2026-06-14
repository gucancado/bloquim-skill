# bloquim-skill

Skill instalável que ensina qualquer Claude (Claude Code / Cowork / API skills) a
**operar o Bloquim de ponta a ponta** via o conector MCP `bloquim`.

O **Bloquim** (MindTask) é um gestor de tarefas, planos de ação e workspaces. Esta skill
fornece o mapa das **35 tools** do MCP, as convenções (priority, prazos, assignee,
standalone vs workspace vs plano) e os fluxos de trabalho mais comuns — otimizada para
disparar no momento certo (gestão de tarefas, action items, planos, revisões) e ficar
quieta no resto.

## Pré-requisito: conectar o MCP `bloquim`

A skill **assume que o conector MCP já está ativo**. Para conectar:

**Claude Code / Claude Desktop:**
1. Configurações → **Conectores** → *Add custom connector*.
2. Nome: `bloquim`
3. URL: `https://mcp.bloquim.beeads.com.br/mcp`
4. Salvar → **Vincular** → faça login com sua conta Bloquim (OAuth 2.0).

O server expõe as 35 tools + 3 prompts (slash commands): `capturar_tarefa`,
`extrair_tarefas_de_reuniao`, `revisao_minhas_tarefas`. Ele também injeta a política de
captura proativa de tarefas no handshake — esta skill **referencia** esse comportamento,
não o duplica.

> Modo stdio (legado, auto-hospedado) usa JWT do backend em vez de OAuth — só relevante
> se você rodar o server localmente.

## Instalar a skill

**Via skills CLI:**
```bash
npx skills add gucancado/bloquim-skill
```

**Ou manualmente** (clone para o diretório de skills do seu agente):
```bash
git clone https://github.com/gucancado/bloquim-skill.git ~/.claude/skills/bloquim
```

## Conteúdo

| Arquivo | O quê |
|---|---|
| [`SKILL.md`](SKILL.md) | Skill principal: trigger, mapa das tools, convenções, captura proativa, workflows, anti-padrões. |
| [`references/tools.md`](references/tools.md) | Catálogo completo das 35 tools — params, enums, roteamento automático. Lido sob demanda. |
| [`examples/workflows.md`](examples/workflows.md) | Transcrições de exemplo dos 5 fluxos-âncora. |

## Como funciona

A skill usa **progressive disclosure**: o `SKILL.md` é enxuto e carrega sempre; o schema
detalhado de cada tool e os exemplos ficam em arquivos lidos só quando necessários, para
não inflar o contexto.

A política de captura proativa (quando oferecer registrar uma tarefa, preview-then-confirm,
mapeamentos de inferência) vive no **servidor** `bloquim-mcp` como fonte única de verdade e
é injetada via `instructions`. Esta skill complementa com o "como operar".

## Licença

[MIT](LICENSE)
