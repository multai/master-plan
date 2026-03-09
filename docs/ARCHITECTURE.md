# System Architecture

> *The infrastructure behind the living universe*

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      MULTI-AI UNIVERSE                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐            │
│  │ Player 1 │     │ Player 2 │     │ Player N │            │
│  └────┬─────┘     └────┬─────┘     └────┬─────┘            │
│       │                │                │                   │
│       └────────────────┼────────────────┘                   │
│                        ▼                                    │
│              ┌─────────────────┐                            │
│              │   GAME ENGINE   │                            │
│              │  (Orchestrator) │                            │
│              └────────┬────────┘                            │
│                       │                                     │
│       ┌───────────────┼───────────────┐                     │
│       ▼               ▼               ▼                     │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│  │  MIND   │    │  MIND   │    │  MIND   │                 │
│  │(pi-agent)│   │(pi-agent)│   │(pi-agent)│                │
│  │         │    │         │    │         │                 │
│  │Creature1│    │Creature2│    │CreatureN│                 │
│  └────┬────┘    └────┬────┘    └────┬────┘                 │
│       │              │              │                       │
│       └──────────────┼──────────────┘                       │
│                      ▼                                      │
│            ┌───────────────────┐                            │
│            │     DATABASE      │                            │
│            │                   │                            │
│            │ • Species Catalog │                            │
│            │ • Creatures       │                            │
│            │ • Worlds          │                            │
│            │ • Events/History  │                            │
│            └───────────────────┘                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Components

### 1. Game Engine (Orchestrator)

The central coordinator that:
- Manages player sessions
- Routes interactions to the right creature minds
- Handles world state and physics
- Generates narrative descriptions and images
- Manages time progression

### 2. Mind (Creature AI)

Each creature runs its own **pi-agent** instance:

```
┌─────────────────────────────────────┐
│              MIND                   │
├─────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  │
│  │ Personality │  │   Memory     │  │
│  │   (prompt)  │  │  (context)   │  │
│  └─────────────┘  └──────────────┘  │
│                                     │
│  ┌─────────────┐  ┌──────────────┐  │
│  │    Drive    │  │   Species    │  │
│  │ (objective) │  │   (traits)   │  │
│  └─────────────┘  └──────────────┘  │
│                                     │
│  ┌─────────────────────────────────┐│
│  │         pi-agent-core           ││
│  │  (LLM interaction & tools)      ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

**Key features:**
- Personality defined by species + individual backstory
- Drive shapes decision-making
- Memory persists between interactions
- Tools for world interaction (move, speak, trade, fight, etc.)

### 3. Database

**Candidates:**
- **PostgreSQL** — Relational, proven, good for structured data
- **SQLite** — Simple, file-based, good for development
- **MongoDB** — Document-based, flexible schemas
- **Convex** — Real-time, serverless (familiar from Butley)

**Schema domains:**

#### Species Catalog
```sql
species (
  id, name, homeworld, description, 
  common_drives, traits, lifespan, culture
)
```

#### Creatures
```sql
creatures (
  id, name, species_id, homeworld, 
  current_location, drive, personality_prompt,
  backstory, status, created_at
)

creature_memory (
  creature_id, memory_key, memory_value, timestamp
)

creature_relationships (
  creature_id, related_creature_id, relationship_type, strength
)
```

#### Worlds
```sql
planets (
  id, name, system, description, environment, cultures
)

locations (
  id, planet_id, name, type, description, connections
)
```

---

## Repositories

| Repository | Purpose | Tech |
|------------|---------|------|
| `master-plan` | Documentation & vision | Markdown |
| `mind` | Creature AI engine | TypeScript, pi-agent |
| `engine` | Game orchestrator | TBD |
| `database` | Schema & migrations | SQL/ORM |
| `worlds` | World/species definitions | YAML/JSON |

---

## Open Questions

- [ ] How many creatures can run simultaneously? (resource management)
- [ ] How do creatures "live" when no player is watching? (background simulation)
- [ ] Real-time vs turn-based interactions?
- [ ] Voice integration (TTS/STT)?
- [ ] Image generation pipeline?
- [ ] Player authentication & persistence?

---

## Next Steps

1. Define database schema in detail
2. Create `mind` repository with pi-agent integration
3. Build minimal creature prototype
4. Design species catalog format
5. Create first test world with a few creatures
