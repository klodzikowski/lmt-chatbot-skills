# LMT Chatbot #2 (Memory, skills, knowledge)

Homework: 

Optional video: As we continue evolving our chatbots towards agents, context management will become increasingly important for increasing the functionality of your system and maintaining its reliability. See a quick overview of one of the main challenges, context rotting. We dealt with it in a very rudimentary way today (by saving verbatim payloads) and there are better ways of dealing with it that you will encounter later (e.g. summarising chats into an actual 'memory' of key facts, just like apps such as ChatGPT do).  

link

A reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot) for Class 20 of the 2 MA LMT *Artificial Intelligence* course at Adam Mickiewicz University. Single static HTML file, no build step. Adds three layers on top of the Class 19 baseline:

- **Memory.** Chat history persisted across sessions. Two backends: browser-local (`localStorage`) or hosted Supabase Postgres.
- **Skills.** Structured custom instructions or domain know-how injected into the system prompt. Three presets plus a custom markdown textarea.
- **RAG.** Two preset documents plus paste-your-own. Chunks, embeds, and retrieves the top-3 most similar passages before each reply.

**At a glance:**

| Component | OpenAI calls | Contribution to the system prompt |
| --- | --- | --- |
| **Memory** | None—`localStorage` (browser) or Supabase Postgres (cloud) | None directly; rehydrated chat history flows in via `messages` |
| **Skills** | None—string concatenation only | Ticked markdown blocks, joined with `---` separators |
| **RAG** | `text-embedding-3-small`—vectorises each chunk at index time, then the user query at chat time | Top-3 chunks (cosine-ranked) under a `# Retrieved context` header |
| **Orchestrator** (`buildAugmentedSystemPrompt`) | None | Joins the three contributions with `---` and ships the assembled prompt to `/v1/chat/completions` |

## Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/). Paste your OpenAI key into Settings, then explore the four drawers: Settings, Memory, Skills, RAG.

## Skills before RAG

Modern context windows fit hundreds of pages of markdown. Reach for a skill before reaching for RAG. Skills are deterministic, version-controllable, free of retrieval-failure modes. RAG is the heavy hammer for cases where skills genuinely won't fit—huge corpora, content changing faster than you can edit a file, retrieval that varies per user.

**Skills win when one curator owns the knowledge. RAG wins when the knowledge already exists across many documents and nobody is curating it.**

As of 2026 the focus has shifted toward **skills**—Anthropic's Skills, OpenAI's Custom GPT knowledge files, Cursor's Rules. Different branding, identical shape: a markdown blob in the system prompt. A couple of years ago (2023–24) the default reflex was RAG-first. Both still ship in production.

## Memory

Tick **Remember conversation between sessions** in the Memory drawer. **Local** writes the chat history JSON to `localStorage` under `lmt-chatbot-skills-history`. **Supabase** writes the same shape to a hosted Postgres `chat_memory` table; reload re-hydrates from whichever backend is active. Same JSON either way—the storage backend is interchangeable.

### Supabase setup

Free tier; ~60 seconds. Sign up at [supabase.com](https://supabase.com) via GitHub; new project, any region.

**SQL Editor** (~30 sec)—paste, then **Run**:

```sql
create table chat_memory (
  id          bigserial primary key,
  user_id     text default 'demo',
  role        text,
  content     text,
  payload     jsonb,
  created_at  timestamptz default now()
);
alter table chat_memory disable row level security;
```

**Or Table Editor (UI)**—new table `chat_memory`; untick **Enable Row Level Security**; keep the pre-filled `id` (int8 identity) and `created_at` (timestamptz default `now()`); click **Add column** four times for `user_id` (text default `'demo'`), `role` (text), `content` (text), `payload` (jsonb). Save.

Project Settings → API → copy the **Project URL** and the **`anon` public** key. Paste both into the Memory drawer after switching the backend to Supabase.

The `anon` key is public by design—safe in client code. Access is gated by Row Level Security policies on the table; we disabled RLS for the class demo. In production you'd write a policy and keep RLS on.

## Skills

Three preset checkboxes—**Top-mark thesis** (sharp MA thesis advisor; Socratic), **Tight summariser** (one-line headline plus 3–7 bullets, never prose), **Stretch a deadline** (extension-request email structure)—plus a **Your skill (markdown)** textarea. Every ticked skill is concatenated into the system prompt with `---` separators on every reply. Ticked state persists in `localStorage` per preset.

Drawer footer → **Simple JSON** → field `system_prompt_assembled` shows the actual string the model saw, with all active skills concatenated.

## RAG

Two preset documents (**Noam Chomsky—life and theory**; the fictional **Jabłoński-Żukowski Conjecture**) plus a paste textarea. **Index document** chunks the textarea at ~500 chars and embeds each via OpenAI's `text-embedding-3-small`. Append-mode—each click adds chunks to the existing index. **Clear index** wipes.

Top-3 most similar chunks land in the system prompt before each chat call. Every retrieval-augmented reply shows a purple **`+N RAG chunks`** chip on its meta line.

Drawer footer → **Detailed JSON** → assistant turn → `retrieved_context` (the actual injected passages) and `retrieved_chunk_count`. Per-turn, because retrieval varies by query.

## Reset behaviour

- **Clear chat**—empties the screen only. Storage stays untouched (both `localStorage` and Supabase). Reload re-hydrates from whichever backend is active.
- **Reset all**—wipes everything: chat, API key, system prompt, sliders, active skills, RAG index, `localStorage` history, AND the Supabase rows for `user_id = 'demo'`.

## Source map

`index.html` is one file. Key entries in the `<script>` block:

- `PRESET_SKILLS`, `PRESET_DOCS`—preset definitions, each `{id, name, blurb, content}`.
- `getActiveSkillContent()`—concatenates ticked presets plus the custom textarea.
- `buildAugmentedSystemPrompt(lastUserMessage)`—assembles `[user prompt] + [active skills] + [retrieved RAG context]` with `---` separators. Returns `{prompt, retrievedContext, retrievedChunkCount}` so the send flow can attach retrieval to the assistant turn.
- `embed(text)`, `cosineSim(a, b)`, `retrieveContext(query, k=3)`—the RAG primitives.
- `saveHistoryToStorage()`, `saveHistoryToSupabase()`, `hydrateHistoryFromSupabase()`—the memory dispatch.

Storage keys are namespaced `lmt-chatbot-skills-*` to avoid colliding with the canonical `lmt-chatbot` on the same `klodzikowski.github.io` origin.

## Use as homework reference

Class 20 homework: point your AI coding assistant at this repo and ask it to add memory, skills, and RAG to your own Class 19 fork. Something like:

> *Add the persistent memory, skills, and RAG from `klodzikowski/lmt-chatbot-skills` to my chatbot, but keep my existing design.*
