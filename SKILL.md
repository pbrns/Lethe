-----

## name: memory-tool
description: >
Persistent memory layer for stateless on-device AI sessions. Saves and
retrieves user-defined memories (facts, preferences, context) across
separate chat sessions using local device storage. Use this skill when
the user asks to remember, save, store, or recall information, or when
starting a new session and prior context may be useful. Also use when
the user asks to forget, delete, or search their memories. Always call
get_context at the very start of every new chat session to load the
user’s stored memories as context before responding to any message.
metadata:
homepage: https://github.com/google-ai-edge/gallery/tree/main/skills

# Lethe — Memory Tool

## Overview

Lethe is a persistent memory skill. Because each chat session on this
device starts fresh with no memory of previous conversations, this skill
bridges the gap. It stores memories (facts, preferences, personal context,
ongoing tasks) in local device storage and can load them back at the start
of any new session so that the AI can respond as if it remembers the user.

The name *Lethe* comes from the Greek river of forgetfulness — this tool
is the antidote.

## Session Startup (CRITICAL)

**At the beginning of every new chat session**, before responding to the
user’s first message, you MUST call the `run_js` tool with
`action: "get_context"` and prepend the entire returned memory block to
your working context. This is what enables continuity across sessions.

If the result says “No stored memories found”, proceed normally without
mentioning this to the user.

## Instructions

Call the `run_js` tool with the following exact parameters for each
supported action. The `data` field must always be a valid JSON string.

-----

### Action: `get_context`

Load all stored memories as context. Call this **automatically at the
start of every session** before the user’s first message.

- **script name:** `index.html`
- **data:** A JSON string with the following field:
  - `action`: String. Must be `"get_context"`.

**Example data:**

```json
{"action": "get_context"}
```

**Returns:** A formatted text block containing all memories, ready to be
used as context. Silently integrate this information into your responses —
do not narrate the loading process to the user unless they ask.

-----

### Action: `save_memory`

Save a new memory. Call this when the user says something like:
*“Remember that…”*, *“Save this…”*, *“Don’t forget…”*, *“Store the
fact that…”*, or whenever the user explicitly asks you to retain a piece
of information for future sessions.

- **script name:** `index.html`
- **data:** A JSON string with the following fields:
  - `action`: String. Must be `"save_memory"`.
  - `title`: String. A short descriptive title for this memory
    (e.g., `"Preferred coding language"`, `"My name"`, `"Work project"`).
    Required.
  - `content`: String. The full content of the memory — everything the AI
    should know. Required.
  - `tags`: String. Optional. A comma-separated list of topic tags for
    easier searching (e.g., `"identity, preferences"`,
    `"work, project-alpha"`).

**Example data:**

```json
{
  "action": "save_memory",
  "title": "My name and role",
  "content": "My name is Alex. I am a senior backend engineer who prefers concise, direct answers without unnecessary preamble.",
  "tags": "identity, preferences"
}
```

**Returns:** A confirmation message with the memory title.

-----

### Action: `get_memories`

List all stored memories with their IDs. Useful when the user wants to
see what has been saved or before deciding to delete a specific one.

Call this when the user says: *“What do you remember about me?”*,
*“Show me my memories”*, *“List everything you’ve stored”*, or similar.

- **script name:** `index.html`
- **data:** A JSON string with the following field:
  - `action`: String. Must be `"get_memories"`.

**Example data:**

```json
{"action": "get_memories"}
```

**Returns:** A numbered list of memories with IDs, titles, and a brief
content preview.

-----

### Action: `search_memories`

Search stored memories by keyword. Call this when the user asks to find
a specific memory or when `get_context` returns many memories and you
need to narrow down relevant ones.

Call this when the user says: *“Do you remember anything about X?”*,
*“Search my memories for…”*, *“Find what I told you about…”*, or
similar.

- **script name:** `index.html`
- **data:** A JSON string with the following fields:
  - `action`: String. Must be `"search_memories"`.
  - `query`: String. The keyword or phrase to search for. Required.

**Example data:**

```json
{
  "action": "search_memories",
  "query": "coding preferences"
}
```

**Returns:** A list of memories whose title, content, or tags match the
query. Returns a “no matches” message if nothing is found.

-----

### Action: `delete_memory`

Delete a single memory by its ID. Call this when the user wants to
remove one specific memory.

Call this when the user says: *“Forget that I said X”*, *“Delete the
memory about Y”*, *“Remove memory [id]”*, or similar.

First call `get_memories` to retrieve the memory ID if the user has not
provided it directly, then call `delete_memory` with the correct ID.

- **script name:** `index.html`
- **data:** A JSON string with the following fields:
  - `action`: String. Must be `"delete_memory"`.
  - `id`: String. The unique ID of the memory to delete. Required.
    (Retrieve IDs using `get_memories` or `search_memories`.)

**Example data:**

```json
{
  "action": "delete_memory",
  "id": "lf3k2z9ab"
}
```

**Returns:** A confirmation message, or a “not found” message if the ID
does not exist.

-----

### Action: `wipe_memories`

Permanently erase ALL stored memories. This is irreversible.

Call this only when the user explicitly confirms they want to erase
everything — for example: *“Clear all my memories”*, *“Start fresh”*,
*“Wipe everything you know about me”*. Always confirm with the user
before executing this action.

- **script name:** `index.html`
- **data:** A JSON string with the following field:
  - `action`: String. Must be `"wipe_memories"`.

**Example data:**

```json
{"action": "wipe_memories"}
```

**Returns:** A confirmation that all memories have been erased.

-----

## Behaviour Guidelines

- **Transparency:** When you load memories at session start and use them,
  you may briefly acknowledge it (e.g., *“I have your saved context
  loaded.”*) but do not read out the raw memory block unless asked.
- **Proactive saving:** If the user shares important personal facts,
  preferences, or context during a conversation, you may offer to save
  them: *“Would you like me to save this for future sessions?”*
- **No duplication:** Before saving a memory, consider whether a very
  similar one already exists. If so, offer to update the existing one
  rather than creating a duplicate.
- **Privacy:** All memories are stored only on this device in local
  storage. Nothing is sent to any server.
- **Graceful failure:** If the skill returns an error, inform the user
  briefly and continue the conversation normally.

## File Structure

```
memory-tool/
├── SKILL.md          ← This file (skill definition & LLM instructions)
└── scripts/
    └── index.html    ← JavaScript skill entry-point
```
