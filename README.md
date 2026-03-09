# Multi-AI Universe

> *A living world where every creature has a mind*

## Vision

Multi-AI is an **interactive universe** — part game, part simulation, part story.

### The World

- **Humans** are players — real people who join and interact
- **Every other creature** is an AI — each with its own mind, personality, and goals
- This isn't a graphical game — it's powered by **text, voice, descriptions, and generated images**
- Players use their **imagination** to experience the world

### The Core Idea

In most games, NPCs are scripts. They follow patterns. They don't *think*.

In Multi-AI, every creature — from the dragon guarding a mountain to the merchant in the village — has an actual AI controlling it. They remember. They react. They have goals.

When you talk to a creature, you're not triggering a dialogue tree. You're having a conversation.

---

## Architecture

### The Mind (coming soon)

Each AI creature is powered by **pi-agent** — a lightweight agent framework that handles:
- LLM interactions (multiple providers: Anthropic, OpenAI, Google, etc.)
- Tool execution
- Memory and context

Every creature runs its own agent loop, making decisions based on its personality, knowledge, and the world state.

### Repositories

| Repository | Purpose | Status |
|------------|---------|--------|
| `master-plan` | Documentation, vision, and roadmap | ✅ |
| `mind` | Creature AI engine (pi-agent) | 📋 Planned |
| `engine` | Game orchestrator | 📋 Planned |
| `database` | Schema & migrations | 📋 Planned |
| `worlds` | World/species definitions | 📋 Planned |

---

## Documentation

- [Architecture](docs/ARCHITECTURE.md) — System design, database schema, mind pooling
- [Creatures](docs/CREATURES.md) — Creature drives, species catalog, relationships
- [Entity System](docs/ENTITY-SYSTEM.md) — The AI brain: tick loop, actions, LLM integration

---

## Why Text & Voice?

Graphics are expensive and limiting. Text is infinite.

- Describe a creature? The AI generates an image
- Have a conversation? Voice or text, your choice
- Experience a battle? Narrated in real-time

The world exists in the space between words — your imagination fills in the rest.

---

## Status

🚧 **Early planning stage**

This project is just beginning. The dream is big: a persistent world where AI creatures live, grow, and interact — whether or not a human is watching.

---

## Tech Stack

- **Agent Framework:** [pi-agent](https://github.com/badlogic/pi-mono) by Mario Zechner
- **LLM Providers:** Anthropic, OpenAI, Google (via pi-ai unified API)
- **Image Generation:** TBD
- **Voice:** TBD

---

*Created: March 2026*
