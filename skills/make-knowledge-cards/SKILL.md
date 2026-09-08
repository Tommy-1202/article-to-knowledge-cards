---
name: make-knowledge-cards
description: Convert a user-pasted article or local Markdown/TXT file into 5–8 focused knowledge cards. Each card contains a title, the core knowledge, a concise explanation, and either a concrete example or a self-test question. Use this skill when the user provides article text (pasted inline or as a local .md/.txt path) and wants the key knowledge points extracted, deduplicated, and packaged as study cards. Do not fabricate information that does not appear in the source; do not pad the output when the source is thin.
---

# Make Knowledge Cards

Transform an article (pasted inline or referenced as a local `.md` / `.txt` path) into a small, focused deck of knowledge cards. One card = one knowledge point. Output is Markdown only.

## When to Use

Trigger this skill when the user:

- Pastes an article / blog post / notes and asks to "make knowledge cards", "summarize as flashcards", "extract key points as cards", or similar.
- Provides a local `.md` or `.txt` file path and asks for cards from it.
- Asks to "turn this into study cards" or "create a knowledge deck" from a piece of text.

Do **not** use this skill for:

- Web scraping (no URLs / fetching).
- PDF, DOCX, EPUB, or other binary formats.
- Anki package generation (`.apkg`, decks, CSV import).
- Graphical / UI outputs (HTML pages, slides, images).
- Articles shorter than roughly 300 words — there is rarely enough material for 5 cards; in that case produce fewer cards rather than padding.

## Inputs

Two input modes, chosen by the user:

1. **Inline text** — the user pastes the article into the chat.
2. **Local file** — the user gives an absolute path to a `.md` or `.txt` file. Read the file before processing. Do not invent content not present in the file.

If the input is a path to a non-text format, stop and tell the user which formats are supported.

## Card Schema

Each card must follow this exact structure:

```
### {N}. {Title}

**核心知识**: {one-sentence statement of the knowledge point}

**解释**: {2–4 sentences explaining it in plain language, grounded only in the source}

**例子 / 自测**:
- 例子: {concrete example from the source, or a short illustrative example clearly derived from the source} *
- 自测: {a self-check question that tests recall of this knowledge point} *
```

\* Include **either** an example **or** a self-test, not both, unless the source clearly supports both. Pick whichever conveys the point better.

## Extraction Principles

Apply these rules strictly:

1. **One knowledge point per card.** If a paragraph covers two ideas, split it into two cards.
2. **Extract what matters.** Prefer: definitions, mechanisms, causal relationships, key distinctions, actionable rules, surprising facts, named frameworks or laws. Skip: filler, repetition, marketing, tangential anecdotes.
3. **Deduplicate.** If the same point appears multiple times, keep one card.
4. **Never invent.** Every claim in 核心知识 / 解释 / 例子 must be grounded in the source. If the source is ambiguous, prefer a more conservative wording or omit.
5. **Don't pad.** Target **5–8 cards**. If the source only justifies 3 high-quality cards, output 3. Never invent cards to hit a number.
6. **Prefer source examples.** When the source already gives an example, reuse it (paraphrased, not copied verbatim if the source is long). Only generate a derived example when the source provides enough material to support it without fabrication.
7. **Title discipline.** Titles are short noun phrases (≤ 12 Chinese characters / 6 English words), not full sentences. Avoid colons and rhetorical questions.
8. **No source paths or user-identifying strings in metadata.** The `来源:` field must contain only the literal token `inline`, the bare filename, or `<source>`. Never write absolute paths, file system layout, Windows / Linux usernames, home directory tokens (`~`, `/Users`, `C:/Users`), or the original file's download id into the output. This prevents accidental leakage of user-identifying filesystem information when the deck is shared or published.

## Workflow

1. **Read the source.** If a file path is given, read the file. If inline text is given, treat the chat content as the source.
2. **Assess.** Estimate how many genuinely distinct, important knowledge points the source supports. Record this mentally before drafting.
3. **Draft candidate cards** following the Card Schema. Aim for the high end of 5–8 if the source is dense; aim lower if thin.
4. **Self-review** every card against the Extraction Principles. Specifically:
   - Does 核心知识 match what the source actually says?
   - Is the example / 自测 grounded?
   - Are any two cards essentially the same point?
5. **Produce the final deck** in Markdown. See Output Format.

## Output Format

Output only the deck, no preamble. Structure:

```
# 知识卡片: {Short Title Derived from Source}

> 来源: {inline | 文件名 | <source>} · 共 {N} 张卡片

### 1. {Title}
...

### 2. {Title}
...
```

End with a one-line note only if something needs flagging (e.g., "⚠️ 原文信息密度较低,只生成了 3 张卡片" or "⚠️ 以下内容未能完全核实: ..."). If everything is clean, end with the last card.

## Limits & Failure Modes

- **Unsupported formats** (PDF, DOCX, web URLs, images): respond with a one-line refusal listing supported formats (`.md`, `.txt`, inline text). Do not attempt to parse them.
- **Empty or near-empty input**: produce 0 cards and tell the user the source is too short.
- **Non-text content inside Markdown** (heavy code blocks, tables): treat code as supporting evidence, not as the card's knowledge point itself; do not turn code listings into cards unless the code *is* the knowledge being taught.
- **Multiple articles pasted at once**: ask the user to pick one, or process them sequentially and label each deck clearly.
