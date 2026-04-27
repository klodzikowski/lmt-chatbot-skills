# LMT Chatbot — Skills Edition

The Class 20 reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot). Same single static HTML file, plus three extensions that turn the bare chatbot into something with memory, lightweight knowledge, and (optional) heavy retrieval.

## Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/)

Paste your OpenAI API key into the drawer to start.

## What's added on top of the Class 19 template

1. **Persistent memory.** A "Remember conversation between sessions" toggle in the drawer. When on, the chat survives page reloads and tab close (`localStorage`).
2. **Summarise button** in the drawer footer. Compresses the running conversation into a markdown summary and appends it as an assistant message. Handy when chats get long, or to demo the "conversation as data" idea.
3. **Markdown skill file.** A textarea where you paste a markdown skill — domain knowledge, persona, style guide, a chapter of your thesis, whatever. It's prepended to the system prompt on every API call. **Skills are the lightweight alternative to RAG**: modern context windows easily fit hundreds of pages of markdown, and the result is deterministic, version-controllable, with zero retrieval-failure modes.
4. **Minimal RAG.** A second textarea takes a long document, an "Index document" button chunks it into ~500-character pieces and embeds each via OpenAI's `text-embedding-3-small`. Before each chat call, the top-3 most similar chunks are retrieved by cosine similarity and prepended to the system prompt.

The pedagogical headline of the fork: **try a markdown skill before you reach for RAG.** RAG is the heavy hammer for cases where skills genuinely won't fit (huge corpora, frequently changing content). Most "I want my bot to know about X" requests are better served by a markdown file in the prompt.

## How it works

`index.html` is one file, all source visible. Look for these functions in the `<script>` block:

- `summariseConversation()` — calls the chat API with a "summarise as markdown" system prompt, renders the result.
- `embed(text)` — POSTs to `/v1/embeddings`, returns a 1,536-dimensional vector.
- `cosineSim(a, b)` — pure-JS arithmetic, no dependencies.
- `indexDocument()` — chunks the RAG textarea, embeds each chunk, stores `{text, embedding}` per chunk in `ragIndex` (and persists to `localStorage` if it fits).
- `retrieveContext(query, k)` — embeds the query, sorts by cosine similarity, returns the top-k chunk texts joined by separators.
- `buildAugmentedSystemPrompt(lastUserMessage)` — assembles the final system prompt as `[user system prompt] + [skill markdown] + [retrieved context]`. Called before each chat completion.

Storage keys are namespaced `lmt-chatbot-skills-*` so this fork doesn't collide with the canonical `lmt-chatbot` if a student runs both on the same `klodzikowski.github.io` origin.

## Class 20 (Memory and retrieval) — context

This repo is the in-class reference for Class 20 of the 2 MA LMT _Artificial Intelligence_ course. Class 20 extends students' own Class 19 forks of `lmt-chatbot` with the same four capabilities. The instructor demos on this repo; students replicate on their own forks for homework.

Class arc: **memory → JSON → markdown skills → RAG**. The argument: each layer is a different intervention with a different cost. Skills first, RAG only when skills run out.

## Licence

MIT. Take it, fork it, ship your own.

## Lineage

- Forked from [`klodzikowski/lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot) (Apache 2.0). Relicensed MIT here.
- Companion consumer demo: [`klodzikowski/dad_jokes`](https://github.com/klodzikowski/dad_jokes).
- All three repos share the topic tag `lmt-ai-course`.
