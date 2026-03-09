# Engine Architecture

> *Two levels of intelligence: ambient life and focused narrative*

---

## The Problem

Running LLM inference for every creature on every tick is:
- **Expensive** — API costs scale with creatures
- **Slow** — Latency adds up
- **Unnecessary** — Most creatures aren't doing anything interesting

But we want a **living universe** where creatures have lives even when no one is watching.

---

## The Solution: Two-Tier Processing

### Tier 1: Global Engine (Deterministic)

Processes **ALL creatures** in the universe with simple rules. No LLM.

```
┌─────────────────────────────────────────────────────────────┐
│                      GLOBAL ENGINE                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  FOR EACH creature IN universe:                              │
│                                                              │
│    IF creature.isInMission:                                  │
│      advanceProgress(creature.mission)                       │
│      checkForScheduledEvents(creature.mission)               │
│                                                              │
│    IF creature.needs.hunger > threshold:                     │
│      seekFood(creature)                                      │
│                                                              │
│    IF creature.isInDanger:                                   │
│      flee(creature)                                          │
│                                                              │
│    IF creature.hasRoutine:                                   │
│      followRoutine(creature)                                 │
│                                                              │
│    updateStats(creature)                                     │
│    updateLocation(creature)                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- ⚡ Fast — Simple rule evaluation
- 💰 Cheap — No API calls
- 📈 Scalable — Millions of creatures
- 🤖 Predictable — Deterministic behavior
- 😐 Shallow — No creativity, no narrative depth

### Tier 2: Entity Engine (LLM-Powered)

Spawned for **specific entities** that need rich interaction. Full LLM processing.

```
┌─────────────────────────────────────────────────────────────┐
│                      ENTITY ENGINE                           │
│                    (Focus: Player A)                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  FOCUS ENTITY: Player A's character                          │
│                                                              │
│  INTERACTION RADIUS: All entities within scope               │
│    • NPCs in same location                                   │
│    • NPCs in active conversation                             │
│    • NPCs involved in same plot                              │
│                                                              │
│  FOR EACH tick:                                              │
│    FOR EACH entity IN interactionRadius:                     │
│      context = buildContext(entity)                          │
│      response = await LLM.think(context)     ← FULL LLM     │
│      executeActions(entity, response)                        │
│      updateMemory(entity, response)                          │
│                                                              │
│  RESULT: Rich, creative, narrative-driven interactions       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- 🧠 Intelligent — Full LLM reasoning
- 🎭 Creative — Unique responses, emergent behavior
- 📖 Narrative — Deep storylines, character development
- 💰 Expensive — API costs per interaction
- 🎯 Focused — Limited scope, high quality

---

## How They Work Together

```
┌─────────────────────────────────────────────────────────────────┐
│                         UNIVERSE                                 │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    GLOBAL ENGINE                           │ │
│  │           (always running, all creatures)                  │ │
│  │                                                            │ │
│  │   😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐   │ │
│  │   😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐   │ │
│  │   😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐😐   │ │
│  │                    (deterministic tick)                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│        ┌─────────────┐              ┌─────────────┐             │
│        │   ENTITY    │              │   ENTITY    │             │
│        │   ENGINE    │              │   ENGINE    │             │
│        │             │              │             │             │
│        │  🎭Player A │              │  🎭Player B │             │
│        │  🧠🧠🧠    │              │  🧠🧠      │             │
│        │  (LLM tick) │              │  (LLM tick) │             │
│        └─────────────┘              └─────────────┘             │
│                                                                  │
│  Legend:                                                         │
│  😐 = Deterministic processing (Global Engine)                  │
│  🧠 = LLM processing (Entity Engine)                            │
│  🎭 = Focus entity (player or VIP)                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Lifecycle

1. **Player disconnected** → Creature processed by Global Engine only
2. **Player connects** → Entity Engine spawns with player as focus
3. **Player enters location** → Nearby NPCs added to interaction radius
4. **Player starts conversation** → That NPC gets full LLM processing
5. **Player disconnects** → Entity Engine terminates, creature returns to Global Engine

### VIP NPCs

Some NPCs are important enough to have their own Entity Engine even without a player:

- Key plot characters
- Faction leaders
- Creatures in critical moments

These can have a dedicated Entity Engine to ensure their storylines progress with depth.

---

## Repository Structure

```
multai/
├── entity/           # The mind itself (library)
│   └── Mind class, Actions, Memory
│
├── global-engine/    # Deterministic universe processing
│   └── Rules, Tick loop, Stats updates
│
├── entity-engine/    # LLM-powered focused processing
│   └── EntityPool, LLM integration, Narrative
│
├── database/         # Already exists
└── master-plan/      # Already exists
```

### Dependencies

```
entity-engine ──imports──► entity
global-engine ──imports──► entity (for types/interfaces)
```

---

## Global Engine: Deterministic Rules

Examples of rules that don't need LLM:

### Movement
```typescript
if (creature.mission?.type === 'travel') {
  creature.mission.progress += creature.speed * tickDuration;
  if (creature.mission.progress >= 1.0) {
    creature.location = creature.mission.destination;
    creature.mission = null;
  }
}
```

### Needs
```typescript
creature.needs.hunger += 0.01 * tickDuration;
creature.needs.energy -= 0.005 * tickDuration;

if (creature.needs.hunger > 0.8 && !creature.currentAction) {
  creature.currentAction = { type: 'seek_food' };
}
```

### Routines
```typescript
const hour = getGameHour();
const routine = creature.routines.find(r => r.hour === hour);
if (routine && !creature.currentAction) {
  creature.currentAction = routine.action;
}
```

### Danger Response
```typescript
if (creature.location.dangerLevel > creature.bravery) {
  creature.currentAction = { type: 'flee', direction: 'nearest_safe' };
}
```

---

## Entity Engine: LLM Processing

When a creature is in an Entity Engine's scope:

```typescript
async function processEntityTick(entity: Entity, context: EngineContext) {
  const prompt = buildPrompt({
    identity: entity.systemPrompt,
    memories: entity.recentMemories,
    location: context.location,
    nearbyEntities: context.nearbyEntities,
    currentSituation: context.situation,
    availableActions: getAvailableActions(entity),
  });

  const response = await llm.complete(prompt);
  
  const action = parseAction(response);
  await executeAction(entity, action);
  
  await updateMemory(entity, {
    type: 'thought',
    content: response.reasoning,
  });
}
```

---

## Interaction Radius

What entities fall within an Entity Engine's scope?

### Same Location
All entities in the same location as the focus entity.

### Active Relationships
Entities with strong relationships to the focus entity, even if distant:
- Family members
- Close allies
- Sworn enemies

### Plot Participants
Entities involved in the same active plot as the focus entity.

### Communication Range
Entities currently in communication (radio, telepathy, etc.).

---

## Resource Management

### Global Engine
- Runs on a schedule (every N seconds)
- Single process can handle entire universe
- CPU-bound, not API-bound

### Entity Engines
- One per connected player (minimum)
- Additional ones for VIP NPCs
- Can be pooled/shared for efficiency
- API-bound (LLM calls)

### Scaling
```
Players: 100
Entity Engines: ~100 (one per player) + ~10 (VIP NPCs)
Global Engine: 1

Creatures in universe: 1,000,000
Creatures with LLM processing: ~500 (in interaction radii)
```

---

## Open Questions

- [ ] How large is the "interaction radius"? (5 entities? 20? Dynamic?)
- [ ] How often does Entity Engine tick? (Real-time? Every second?)
- [ ] Can Entity Engines share resources? (Batch LLM calls?)
- [ ] How do we handle Entity Engine handoffs? (Player moves, NPCs change)
- [ ] Priority system for VIP NPCs? (Who gets an Entity Engine?)
