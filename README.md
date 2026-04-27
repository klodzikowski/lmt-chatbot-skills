# LMT Chatbot — Skills edition

A reference fork of [`lmt-chatbot`](https://github.com/klodzikowski/lmt-chatbot). Adds three layers on top of the bare Class 19 chatbot: persistent memory with a Local↔Supabase backend toggle, three preset skills plus a custom textarea, and a minimal RAG retrieval flow. Single static HTML file, no build step.

## 1. Try it

[klodzikowski.github.io/lmt-chatbot-skills](https://klodzikowski.github.io/lmt-chatbot-skills/). Paste your OpenAI key into Settings, then explore the four collapsible drawers.

## 2. What's added

- **Memory.** Tick *Remember conversation between sessions* in the Memory drawer. Pick a backend: **Local** writes the chat history JSON to `localStorage`; **Supabase** writes the same shape to a hosted Postgres `chat_memory` table. Same JSON either way; the storage backend is interchangeable.
- **Skills.** Three preset skills with checkboxes—**Top-mark thesis** (sharp MA thesis advisor, Socratic questioning), **Tight summariser** (bullets only, never prose), **Stretch a deadline** (email structure for asking a professor for an extension)—plus a **Your skill (markdown)** textarea for your own. Every ticked skill is concatenated into the system prompt on every reply.
- **RAG.** Paste a long document, click **Index document**, and ~500-character chunks are embedded via OpenAI's `text-embedding-3-small`. Index is **append-mode**: every Index click adds to the existing chunks (paste a second source, index again, both coexist). **Upload .txt / .md** for file input. **Clear index** wipes. Top-3 most similar chunks are prepended to the system prompt before each chat call.
- **JSON proof of injection.** Drawer footer → **Simple JSON** or **Detailed JSON**. The downloaded file includes `system_prompt_assembled`—the actual string sent to the model with all active skills concatenated—plus `skills_active`, `custom_skill`, `memory_backend`, and `rag_chunks_indexed`. Open the file to see exactly what the model saw.

## 3. The argument: skills before RAG

Modern context windows fit hundreds of pages of markdown. Reach for a skill before reaching for RAG. Skills are deterministic, version-controllable, free of retrieval-failure modes. RAG is the heavy hammer for cases where skills genuinely won't fit—huge corpora, content changing faster than you can edit a file, retrieval that varies per user.

**Skills win when someone owns the knowledge. RAG wins when the knowledge already exists and nobody's curating it.**

Same pattern as Anthropic's Skills, OpenAI's Custom GPT knowledge files, Cursor's Rules. Different branding, identical shape.

## 4. Source map

`index.html` is one file. Key functions in the `<script>` block:

- `PRESET_SKILLS` — the three preset skill definitions, each `{id, name, blurb, content}`.
- `getActiveSkillContent()` — concatenates ticked presets + the custom textarea into one markdown blob.
- `buildAugmentedSystemPrompt(lastUserMessage)` — assembles `[user prompt] + [active skills] + [retrieved RAG context]` with `---` separators. Called before every chat completion.
- `embed(text)` — POSTs `/v1/embeddings`, returns a 1,536-dim vector.
- `cosineSim(a, b)` — pure JS, no dependencies.
- `retrieveContext(query, k=3)` — embeds the query, cosine-sorts the index, returns the top-k chunk texts.
- `saveHistoryToStorage()` — dispatches save to `localStorage` or Supabase based on the Memory backend.
- `saveHistoryToSupabase()` / `hydrateHistoryFromSupabase()` — wipe-and-rewrite save and ordered hydrate against the `chat_memory` table.

Storage keys are namespaced `lmt-chatbot-skills-*` to avoid colliding with the canonical `lmt-chatbot` on the same `klodzikowski.github.io` origin.

## 5. Supabase setup (if you want the cross-device backend)

Sign up at [supabase.com](https://supabase.com) via GitHub. New project, any region, save the database password somewhere. In the SQL Editor, run:

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

Project Settings → API → copy the Project URL and the `anon public` key. Paste both into the app's Memory drawer (after switching backend to Supabase). The `anon` key is safe to put in client code—access is gated by Row Level Security policies, not the key. We disabled RLS for the class demo; add a policy back in production.

## 6. Reset behaviour

- **Clear chat** — empties the chat off the screen. Storage stays untouched (both `localStorage` and Supabase). Reload re-hydrates.
- **Reset all** — wipes the chat, the API key, the system prompt, the sliders, the active skills, the RAG index, the `localStorage` history, AND the Supabase rows for `user_id = 'demo'`. Clean slate.
