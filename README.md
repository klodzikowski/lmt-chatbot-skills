# LMT Chatbot — Skills edition

A reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot). Same single HTML file. Adds persistent memory, a Summarise button, a markdown skill, and minimal RAG on top of the Class 19 baseline.

## Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/). Paste your OpenAI key into the drawer.

## What's added

1. **Persistent memory.** A toggle in the drawer keeps the chat across reloads via `localStorage`. Class 20 lives the production upgrade: Supabase, summarisation every few turns, a live database panel.
2. **Summarise.** A button in the drawer footer. Compresses the running chat into a markdown summary and appends it as an assistant message.
3. **Markdown skill.** A textarea for a persona, a style guide, a syllabus chapter—any markdown. Prepended to the system prompt on every call.
4. **Minimal RAG.** A second textarea for a long document. The "Index" button chunks it into ~500-character pieces and embeds each via OpenAI's `text-embedding-3-small`. Top-3 most similar chunks land in the system prompt before each chat.

## The argument

Try a markdown skill before reaching for RAG. Modern context windows fit hundreds of pages of markdown—deterministic, version-controllable, no retrieval failures. RAG is the heavy hammer for cases where skills won't fit: huge corpora, frequently changing content. Skills win when someone owns the knowledge; RAG wins when the knowledge already exists and nobody's curating it.

Same pattern as Anthropic's Skills, OpenAI's Custom GPT knowledge files, and Cursor's Rules. Different branding, same shape.

## How it works

`index.html` is one file. Key functions in the `<script>` block:

- `summariseConversation()`: chat API call with a "summarise as markdown" system prompt.
- `embed(text)`: POSTs to `/v1/embeddings`, returns a 1,536-dimensional vector.
- `cosineSim(a, b)`: pure JS, no dependencies.
- `indexDocument()`: chunks the textarea, embeds each chunk, stores `{text, embedding}` per chunk.
- `retrieveContext(query, k)`: embeds the query, cosine-sorts, returns top-k chunks.
- `buildAugmentedSystemPrompt(lastUserMessage)`: assembles `[user prompt] + [skill] + [retrieved context]`.

Storage keys are namespaced `lmt-chatbot-skills-*` to avoid colliding with the canonical `lmt-chatbot` on the same origin.

## Class context

In-class reference for Classes 20–21 of the 2 MA LMT _Artificial Intelligence_ course at Adam Mickiewicz University. Class 20 covers memory and persistence; Class 21 covers skills and RAG. Both classes demo on this fork.

## Licence

MIT. Take it, fork it, ship your own.

## Lineage

- Forked from [`klodzikowski/lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot) (Apache 2.0). Relicensed MIT.
- Companion consumer demo: [`klodzikowski/dad_jokes`](https://github.com/klodzikowski/dad_jokes).
- Topic tag: `lmt-ai-course`.
