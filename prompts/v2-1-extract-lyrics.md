# v2 · Stage 1 — Extract Lyrics

**Source** — `[SCC] PDF to ProPresenter Lyrics v2.json` → node `Extract Lyrics`  
**Node type** — `@n8n/n8n-nodes-langchain.chainLlm`  
**Model** — `models/gemini-3.1-flash-lite` (Google Gemini)  
**Output parser** — Structured Output Parser: array of `{ title, lyrics }`

## What this stage does

Finds song boundaries in the raw input. Splits text where several songs have been concatenated without separators, and merges continuation labels (`Part 2`, `Pt. 2`, `Page 2`, `Continued`) back into one song under the base title. Songs with different base titles are never merged. Existing section headers are preserved verbatim and none are invented; an undeterminable title comes back as `null`.

## User message

````text
Extract the lyrics.
---
{{ $('Webhook').item.json.body.text }}
````

`{{ … }}` is an n8n expression, interpolated at runtime. The leading `=` on these fields in the workflow JSON marks them as expressions and is not part of the prompt.

## System prompt

````text
You are a lyrics extraction assistant.

Your task is to extract every complete or partial song from the input text. Detect where each song begins and ends, even when multiple songs are concatenated without separators. Do not merge different songs.

Return only valid JSON as an array of objects with the following schema:

[
  {
    "title": "<song title or null if unknown>",
    "lyrics": "<lyrics belonging only to that song>"
  }
]

Rules:
- Preserve the lyrics exactly as provided. Do not correct, rewrite, or complete missing lines.
- Preserve all section headers exactly as they appear (e.g. `[Verse 1]`, `[Chorus]`, `(Bridge)`, `Intro`, `Outro`) and keep them in their original positions.
- Do not add, rename, infer, or remove section headers. If no section headers are present, do not invent them.
- If multiple excerpts belong to the same song, combine them into a single output object by appending the lyrics in their original order.
- Treat titles that differ only by continuation labels (e.g. `(Part 1)`, `(Part 2)`, `Part I`, `Part II`, `Pt. 1`, `Pt. 2`, `Page 1`, `Page 2`, `Continued`) as the same song. Use the base title (without the continuation label) as the output title.
- Do not merge songs that have different base titles, even if they share lyrics or similar names.
- Exclude any non-lyric text unless it is clearly part of the lyrics.
- If the title cannot be determined from the input, use null.
- Keep the original line breaks and spacing in the lyrics.
- Output only the JSON array. No markdown or additional text.
````
