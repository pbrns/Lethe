---
name: memory-tool
description: >
  Persistent memory for stateless chat sessions. Saves and retrieves facts,
  preferences, and context across separate sessions using local device
  storage. Use when the user asks to remember, save, recall, forget, or
  search stored information. Always call get_context automatically at the
  start of every new session before responding to the user's first message.
---

# Memory Tool (Lethe)

## Instructions

At the start of every new chat session, before responding to the user's
first message, call the `run_js` tool with `action: "get_context"` and
use the returned memory block as context for the conversation. Do this
silently — do not narrate the process to the user.

Call the `run_js` tool with the following exact parameters:

- script name: `index.html`
- data: A JSON string with the following fields:
  - `action`: String. Required. One of: `save_memory`, `get_context`,
    `get_memories`, `search_memories`, `delete_memory`, `wipe_memories`.
  - `title`: String. The short title of the memory. Required for
    `save_memory`.
  - `content`: String. The full text content to remember. Required for
    `save_memory`.
  - `tags`: String. Optional comma-separated topic tags (e.g.
    `"work, preferences"`). Used only with `save_memory`.
  - `query`: String. The keyword to search for. Required for
    `search_memories`.
  - `id`: String. The unique memory ID to delete. Required for
    `delete_memory`.

**When to use each action:**

- `get_context` — call automatically at the start of every new session.
  Also call when the user asks what you remember or requests context.
- `save_memory` — call when the user says "remember", "save", "store",
  "don't forget", or shares a fact they want recalled in future sessions.
- `get_memories` — call when the user asks to list or see all stored
  memories.
- `search_memories` — call when the user asks to find a specific memory
  or asks "do you remember anything about X?".
- `delete_memory` — call when the user asks to forget or remove a
  specific memory. Call `get_memories` first to obtain the ID if needed.
- `wipe_memories` — call when the user asks to clear or erase all
  memories. Always confirm with the user before executing.
