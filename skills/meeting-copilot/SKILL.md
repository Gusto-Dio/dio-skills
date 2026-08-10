---
name: meeting-copilot
description: Use when the user pastes a link to a Notion meeting-note page (Notion's live AI transcript) and wants a personalized, live Portuguese interpretation of the meeting written into their personal Dio-lab hub. Keeps updating roughly every 60-90s until the user says the meeting is over.
requires_mcp: [notiongusto]
allowed-tools: [mcp__claude_ai_Notion_Gusto__notion-fetch, mcp__claude_ai_Notion_Gusto__notion-create-pages, mcp__claude_ai_Notion_Gusto__notion-update-page]
---

# /meeting-copilot — Live Portuguese meeting interpreter

Monitora a transcrição de uma reunião no Notion (gerada em inglês) e escreve, em
português, uma interpretação personalizada — temas discutidos, pontos de atenção,
próximos passos e sugestões — numa página de reunião, atualizando a cada ~60-90s
enquanto a reunião estiver rolando.

## Config

| Key | Value |
|---|---|
| Dio-lab hub page ID | `3b5ad673-c6c2-805a-9a21-dcc1be92e9a8` |
| Meetings container title | `Meetings` |

## Section template

Todo conteúdo da página de reunião (na criação e em cada ciclo do loop) segue este
template:

```
**Reunião original (inglês):** [<TÍTULO>](<SOURCE_URL>)
**Status:** <STATUS_LINE>

## Temas discutidos
- <bullet por assunto discutido>

## Pontos de atenção
- <o que foi reforçado / importa observar, mesmo que não dirigido ao usuário>

## Próximos passos
- <ações concretas mencionadas>

## Take-aways & sugestões
- <interpretação personalizada: quando algo cruzar com o perfil/contexto de
  carreira do usuário (já disponível na memória da sessão), explicar por que
  vale atenção>
```

`<STATUS_LINE>` é `🟢 Ao vivo — atualizado automaticamente a cada ~1 min` durante
o loop, e `✅ Finalizado às <hora>` depois do encerramento.
