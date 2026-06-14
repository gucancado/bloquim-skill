# Casos de uso — Bloquim via MCP

Guia prático pra tirar proveito do conector `bloquim` no Claude. Fale natural — você
não precisa decorar nenhuma tool. Cada caso traz uma **frase-gatilho** pra colar e
testar. `⚠️` = cria/altera dado real (a captura é *preview-then-confirm*: nada é criado
sem sua confirmação; ainda assim, prefira um workspace de teste e limpe depois).

---

## 1. Captura proativa — deixe o agente te pegar

Solte o compromisso no meio da conversa; ele oferece registrar.

```
preciso revisar os criativos do cliente amanhã de manhã
```
Espera: oferta de 1 linha (título/prioridade/prazo inferidos) → você confirma → cria.
Recusou? Ele cala. Sem insistência.

## 2. Reunião → tarefas (caso âncora) ⚠️

```
/extrair_tarefas_de_reuniao
```
Depois cole a transcrição/ata (Fireflies, Meet, notas). Espera: tabela de action items
com responsável + prazo inferidos → confirmação em lote → cria N tarefas.

## 3. Ritual de revisão

```
/revisao_minhas_tarefas
```
```
o que está atrasado?
```
```
me mostra minha semana
```

## 4. Plano de ação (mapa/projeto com etapas) ⚠️

```
monta um plano "Teste lançamento" no meu workspace de teste com 3 etapas: brief, criativos, subir campanha
```
Cards nascem `pending`. Ajuste prazo/prioridade direto no card depois. Limpeza:
```
apaga o plano "Teste lançamento"
```

## 5. Delegação em workspace (time) ⚠️

```
no workspace X, cria a tarefa "editar vídeo" e atribui pro email ana@exemplo.com
```
```
o que está pendente e sem responsável no workspace X?
```

## 6. Entregável + comentário ⚠️

```
anexa esse link como entregável na tarefa "editar vídeo" e comenta que é a versão final: https://exemplo.com/arquivo.pdf
```

## 7. Busca / triagem

```
acha minhas tarefas que falam de relatório
```
```
tem algum plano de black friday?
```

## 8. Manutenção / organização ⚠️

```
marca a tarefa "editar vídeo" como urgente
```
```
move a tarefa "editar vídeo" pro workspace Y
```

---

## Convenções que ajudam (e evitam erro)

- **Prazo sempre com a modalidade certa:** "até sexta" → prazo final; "dia 20" → dia pontual;
  "entre seg e qua" → janela. Data sem modalidade é descartada.
  ```
  anota: enviar boleto entre segunda e quarta que vem, prioridade alta
  ```
  ```
  lembra de ligar pro contador dia 25
  ```
- **Sem workspace** = vai pra **Minhas Tarefas** (standalone).
- **`list_my_tasks` mostra só "hoje"** por default — peça "tudo" pra ver o resto.

## Onboarding de um novo usuário (roteiro)

1. **Conectar o conector** — `https://mcp.bloquim.beeads.com.br/mcp` + login (ver README).
2. **Instalar a skill** `bloquim` → o agente já sabe operar tudo.
3. **Primeiro contato:** rodar `/revisao_minhas_tarefas` e jogar uma ata em `/extrair_tarefas_de_reuniao`.
4. **Mensagem-chave:** "não decore tool nenhuma — fale natural; a captura proativa e os 3 prompts fazem o resto."
