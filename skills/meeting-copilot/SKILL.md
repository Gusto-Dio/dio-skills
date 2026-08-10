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

## Phase 0 — Setup (roda uma vez, quando o usuário cola o link)

1. Pegue o link/ID colado pelo usuário como `<SOURCE_ID>`.
2. Busque a página de origem:

   ```
   notion-fetch({ id: "<SOURCE_ID>", include_transcript: true })
   ```

   Extraia:
   - `<TITLE>`: o título da nota. Se o título for só uma data/hora automática
     (ex: `**<mention-date .../>**`, sem texto descritivo), use
     `Reunião — <data> <hora>` no lugar.
   - `<TRANSCRIPT>`: o texto dentro de `<transcript>`. Se vazio, trate como "sem
     conteúdo ainda" (não é erro).
   - `<SOURCE_URL>`: a URL da própria página de origem (campo `url` da resposta).

3. Garanta que existe uma página "Meetings" direto abaixo do Dio-lab:
   - `notion-fetch({ id: "3b5ad673-c6c2-805a-9a21-dcc1be92e9a8" })` e procure, no
     `<content>`, uma `<page>` cujo texto seja `Meetings`.
   - Se não existir, crie:

     ```
     notion-create-pages({
       parent: { type: "page_id", page_id: "3b5ad673-c6c2-805a-9a21-dcc1be92e9a8" },
       pages: [{ properties: { title: "Meetings" }, icon: "📅" }]
     })
     ```

   - Guarde o ID retornado como `<MEETINGS_ID>`.

4. Garanta que existe uma página com a data de hoje (formato `YYYY-MM-DD`) abaixo
   de `<MEETINGS_ID>`:
   - `notion-fetch({ id: "<MEETINGS_ID>" })`, procure uma `<page>` cujo texto seja
     a data de hoje.
   - Se não existir, crie do mesmo jeito, com `parent.page_id = <MEETINGS_ID>` e
     `properties.title` = a data de hoje.
   - Guarde o ID retornado como `<DAY_ID>`.

5. Crie a página da reunião abaixo de `<DAY_ID>`, com o Section template
   preenchido a partir do `<TRANSCRIPT>` atual e `<STATUS_LINE>` =
   `🟢 Ao vivo — atualizado automaticamente a cada ~1 min`:

   ```
   notion-create-pages({
     parent: { type: "page_id", page_id: "<DAY_ID>" },
     pages: [{
       properties: { title: "<TITLE>" },
       icon: "🗣️",
       content: "<section template preenchido>"
     }]
   })
   ```

   Guarde o ID retornado como `<MEETING_PAGE_ID>`.

6. Avise o usuário que o acompanhamento começou, com o link da página criada, e
   siga para a Phase 1.
