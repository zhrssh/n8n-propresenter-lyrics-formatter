# Prompts

The prompts are the portable part of this project. The n8n workflows are one way to run them; the pipeline they describe — split songs, strip artifacts, lay out slides — has nothing to do with n8n and can be reimplemented on any platform or in plain application code.

Each file below contains the system prompt and user message **verbatim** as they appear in the workflow JSON, with n8n's leading `=` expression marker stripped.

## v2 pipeline (current)

Three sequential LLM calls per song, each with a narrow job:

| | Prompt | Job |
|---|---|---|
| 1 | [Extract Lyrics](v2-1-extract-lyrics.md) | Find song boundaries; split concatenated songs; merge `Part 1` / `Part 2` continuations |
| 2 | [Artifacts Cleaner](v2-2-artifacts-cleaner.md) | Remove chords, page numbers, credits, repeat instructions; rejoin OCR-split words |
| 3 | [ProPresenter Formatter](v2-3-propresenter-formatter.md) | Normalize section headers; split into slides |

Stage 1 runs once over the whole input and returns an array; stages 2 and 3 run once per song.

## v1 pipeline (legacy)

Two calls, no song-splitting stage — v1 gets one song per item from the form upload.

| | Prompt | Job |
|---|---|---|
| 1 | [Lyrics Extractor](v1-1-lyrics-extractor.md) | Remove chords and artifacts (more detailed than v2's cleaner) |
| 2 | [ProPresenter Formatter](v1-2-propresenter-formatter.md) | Split into slides (looser rules than v2 — allows 3 lines per slide) |

## Structured output

Every v2 stage is constrained by the same JSON schema, attached in n8n as a Structured Output Parser node. If you port these prompts, reproduce this with whatever structured-output or JSON-mode facility your platform offers — the prompts describe the shape in prose too, but the schema is what actually keeps the output parseable:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "title": { "type": "string" },
      "lyrics": { "type": "string" }
    },
    "required": ["title", "lyrics"],
    "additionalProperties": false
  }
}
```

v1 has no output parser — its agents return plain text, which is why v1 can't name files after songs.

## Notes for porting

- **Model** — everything here was written against `gemini-3.1-flash-lite`. The prompts are ordinary instruction-following text with no provider-specific syntax, but rule adherence varies by model; re-check the "do not invent section headers" and duplicate-block rules after switching.
- **Why three stages instead of one** — a single prompt covering splitting, cleaning, and layout tended to drop rules under load: format correctly but miss a chord, or clean well but merge two songs. Narrow prompts plus a schema hold up better. The cost is three calls per song.
- **Keep these in sync** — if you edit a prompt in the n8n UI, re-export the workflow JSON and update the matching file here. The JSON is the running copy; these files are the readable one.
