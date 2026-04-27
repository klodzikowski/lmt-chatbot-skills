# LMT Chatbot — Skills edition

A reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot). Adds memory, summarise, a markdown skill, and minimal RAG. Same single HTML file, no build step.

## 1. Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/). Paste your OpenAI key into the drawer.

## 2. What's added

- **Memory.** A toggle in the drawer keeps the chat across reloads via `localStorage`.
- **Summarise.** A drawer-footer button compresses the chat into a markdown summary.
- **Skill.** A textarea for a markdown skill: a persona, a style guide, a syllabus chapter. Prepended to the system prompt every call.
- **RAG.** A second textarea for a long document. The Index button chunks at ~500 chars and embeds each via `text-embedding-3-small`. Top-3 chunks land in the system prompt before each chat.

## 3. Skills before RAG

Modern context windows fit hundreds of pages of markdown. Try a skill before reaching for RAG. Skills are deterministic, version-controllable, no retrieval failures. RAG is the heavy hammer for huge corpora or content changing faster than you can edit a file. Skills win when someone owns the knowledge; RAG wins when nobody's curating it.

Same shape as Anthropic's Skills, OpenAI's Custom GPT knowledge files, Cursor's Rules.

## 4. Source map

`index.html` is one file. Key functions in the `<script>` block:

- `summariseConversation()`: chat call with a summarise system prompt.
- `embed(text)`: POSTs `/v1/embeddings`, returns a 1,536-dim vector.
- `cosineSim(a, b)`: pure JS, no deps.
- `indexDocument()`: chunks the textarea, embeds each, stores `{text, embedding}` per chunk.
- `retrieveContext(query, k)`: embeds the query, cosine-sorts, returns top-k.
- `buildAugmentedSystemPrompt(...)`: assembles `[user prompt] + [skill] + [retrieved context]`.

Storage keys namespaced `lmt-chatbot-skills-*` to avoid colliding with `lmt-chatbot` on the same origin.
