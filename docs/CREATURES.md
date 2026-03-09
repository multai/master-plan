# Creatures

> *Every creature has a mind, a drive, and a place in the universe*

## Core Concepts

### Drives & Objectives

Each creature has its own **drive** — what motivates them, what they want from life:

| Drive Type | Example |
|------------|---------|
| **Family** | A family man protecting and providing for his loved ones |
| **Wealth** | A bounty hunter chasing rewards across the galaxy |
| **Freedom** | A fugitive running from the law, surviving day by day |
| **Power** | A warlord seeking dominion over territories |
| **Knowledge** | A scholar uncovering ancient secrets |
| **Survival** | A scavenger living on the edge |

Drives shape behavior. A family-driven creature will prioritize safety; a bounty hunter will take risks for the right price.

### Identity & Connection

Creatures form bonds based on:

1. **Species** — Creatures of the same species recognize each other. Shared biology, culture, and instincts.
2. **Planet of Origin** — Creatures from the same world share history, environment, and often language.
3. **Faction/Allegiance** — Beyond biology, creatures may align with groups, guilds, or causes.

Just like humans on Earth would identify with other humans, creatures from Jupiter would have their own shared identity.

---

## Species Catalog

*(To be populated)*

Each species entry should include:

```yaml
species:
  name: "Kelthari"
  homeworld: "Keltar Prime"
  physical_description: "..."
  culture: "..."
  common_drives: ["family", "territory"]
  lifespan: "200 cycles"
  language: "Kelthari Common"
  notable_traits: ["telepathic bonds", "pack mentality"]
```

---

## Creature Database Schema

Every creature in the universe is stored with:

| Field | Description |
|-------|-------------|
| `id` | Unique identifier |
| `name` | Creature's name |
| `species_id` | Reference to species catalog |
| `homeworld` | Planet of birth |
| `current_location` | Where they are now |
| `drive` | Primary motivation |
| `personality` | AI personality traits |
| `backstory` | Personal history |
| `relationships` | Connections to other creatures |
| `inventory` | What they carry |
| `status` | Alive, dead, missing, etc. |
| `memory` | What they remember (AI context) |

---

## Population

The universe will be **fully populated** — every creature exists in the database before any player arrives.

When a player travels through the world, they encounter creatures that already exist, with their own histories and ongoing lives.

Creatures don't spawn when needed. They *live*.
