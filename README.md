# LMT Chatbot—Skills edition

A reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot). Adds three layers on top of the bare Class 19 chatbot: persistent memory with a Local↔Supabase backend toggle, three preset skills plus a custom textarea, and a minimal RAG retrieval flow. Single static HTML file, no build step.

## 1. Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/). Paste your OpenAI key into Settings, then explore the four collapsible drawers.

## 2. The four drawers

- **Settings.** API key, model, system prompt, sampling, max tokens. The Class 19 controls, tucked behind a collapsible.
- **Memory.** "Remember conversation between sessions" toggle plus a Local↔Supabase backend radio. **Local** writes the chat history JSON to `localStorage`; **Supabase** writes the same shape to a hosted Postgres `chat_memory` table. Same JSON either way; the storage backend is interchangeable.
- **Skills.** Three preset checkboxes—**Top-mark thesis**, **Tight summariser**, **Stretch a deadline**—plus a **Your skill (markdown)** textarea. Every ticked skill is concatenated into the system prompt on every reply. Visible in **Simple JSON → `system_prompt_assembled`**.
- **RAG.** Two preset documents (Noam Chomsky's life and theory; the fictional Jabłoński-Żukowski Conjecture) plus a paste textarea. **Index document** chunks the textarea at ~500 chars and embeds each via OpenAI's `text-embedding-3-small`. Append-mode; **Clear index** wipes. Top-3 most similar chunks land in the system prompt before each chat call. Every retrieval-augmented reply shows a purple **`+N RAG chunks`** chip on its meta line; **Detailed JSON → `retrieved_context`** has the actual passages per turn.

## 3. Skills before RAG

Modern context windows fit hundreds of pages of markdown. Reach for a skill before reaching for RAG. Skills are deterministic, version-controllable, free of retrieval-failure modes. RAG is the heavy hammer for cases where skills genuinely won't fit—huge corpora, content changing faster than you can edit a file, retrieval that varies per user.

**Skills win when someone owns the knowledge. RAG wins when the knowledge already exists and nobody's curating it.**

Same pattern as Anthropic's Skills, OpenAI's Custom GPT knowledge files, Cursor's Rules. Different branding, identical shape.

## 4. Supabase setup (for cross-device memory)

Free tier; ~60 seconds. Sign up at [supabase.com](https://supabase.com) via GitHub; new project, any region.

**SQL Editor (~30 sec)**—paste, then **Run**:

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

**Or Table Editor UI**—new table `chat_memory`; untick **Enable Row Level Security**; keep the pre-filled `id` (int8 identity) and `created_at` (timestamptz default `now()`); click **Add column** four times for `user_id` (text default `'demo'`), `role` (text), `content` (text), `payload` (jsonb). Save.

Project Settings → API → copy the **Project URL** and the **`anon` public** key. Paste both into the app's Memory drawer after switching the backend to Supabase. The `anon` key is safe in client code—access is gated by Row Level Security policies, not key secrecy. We disabled RLS for the class demo; in production you'd write a policy and keep RLS on.

## 5. Reset behaviour

- **Clear chat**—empties the conversation off the screen. Storage stays untouched (both `localStorage` and Supabase). Reload re-hydrates from whichever backend is active.
- **Reset all**—wipes the chat, the API key, the system prompt, the sliders, the active skills, the RAG index, the `localStorage` history, AND the Supabase rows for `user_id = 'demo'`. Clean slate.

## 6. Source map

`index.html` is one file. Key entries in the `<script>` block:

- `PRESET_SKILLS`, `PRESET_DOCS`—preset definitions, each `{id, name, blurb, content}`.
- `getActiveSkillContent()`—concatenates ticked presets plus the custom textarea.
- `buildAugmentedSystemPrompt(lastUserMessage)`—assembles `[user prompt] + [active skills] + [retrieved RAG context]` with `---` separators. Returns `{prompt, retrievedContext, retrievedChunkCount}` so the send flow can attach retrieval to the assistant turn.
- `embed(text)`, `cosineSim(a, b)`, `retrieveContext(query, k=3)`—the RAG primitives.
- `saveHistoryToStorage()`, `saveHistoryToSupabase()`, `hydrateHistoryFromSupabase()`—the memory dispatch.

Storage keys are namespaced `lmt-chatbot-skills-*` to avoid colliding with the canonical `lmt-chatbot` on the same `klodzikowski.github.io` origin.

## 7. Class context

Class 20 of the 2 MA LMT *Artificial Intelligence* course at Adam Mickiewicz University, 28 April 2026.
