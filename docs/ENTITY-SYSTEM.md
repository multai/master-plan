# Entity System

> *The brain and heartbeat of every creature in the universe*

**Repository name candidates:** `entity`, `creature`, `mind`, `soul`, `being`

**Stack:** Built on top of [pi-agent](https://github.com/badlogic/pi-mono) — NOT OpenClaw (too heavy for this use case)

---

## Core Concept

Every non-player creature in the universe is an **Entity** — an autonomous AI agent with:

- **Actions** — Things it can do (speak, move, trade, fight, craft, etc.)
- **Perception** — What it knows about its surroundings
- **Drive** — What motivates it
- **Memory** — What it remembers
- **Tick** — Internal heartbeat that keeps it "alive"

---

## The Entity Loop

Each entity runs a simple internal loop:

```
┌─────────────────────────────────────────┐
│              ENTITY TICK                │
├─────────────────────────────────────────┤
│                                         │
│  1. PERCEIVE                            │
│     - What's happening around me?       │
│     - Who's nearby?                     │
│     - What changed since last tick?     │
│                                         │
│  2. THINK (LLM call)                    │
│     - Given my drive and situation...   │
│     - What should I do next?            │
│                                         │
│  3. ACT                                 │
│     - Execute the chosen action         │
│     - Update world state                │
│                                         │
│  4. REMEMBER                            │
│     - Store important events            │
│     - Update relationships              │
│                                         │
└─────────────────────────────────────────┘
```

---

## Universe Tick Loop

The universe runs a master loop that gives every creature a chance to "live":

```
┌──────────────────────────────────────────────────────────────┐
│                    UNIVERSE TICK LOOP                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  while (universe.running) {                                   │
│                                                               │
│    // 1. Get all active entities                              │
│    entities = getEntitiesNeedingTick()                        │
│                                                               │
│    // 2. Process each entity                                  │
│    for (entity of entities) {                                 │
│                                                               │
│      // What is this entity doing?                            │
│      if (entity.isInMission()) {                              │
│        // Traveling on spaceship, on a quest, etc.            │
│        processMissionTick(entity)                             │
│      }                                                        │
│                                                               │
│      if (entity.hasInteraction()) {                           │
│        // Someone is talking to them                          │
│        processInteractionTick(entity)                         │
│      }                                                        │
│                                                               │
│      if (entity.isIdle()) {                                   │
│        // Living their daily life                             │
│        processIdleTick(entity)                                │
│      }                                                        │
│                                                               │
│      // LLM decides what entity does                          │
│      action = await entity.think()                            │
│      await entity.execute(action)                             │
│    }                                                          │
│                                                               │
│    // 3. Advance world time                                   │
│    advanceWorldTime()                                         │
│                                                               │
│    // 4. Sleep until next tick                                │
│    await sleep(TICK_INTERVAL)                                 │
│  }                                                            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Entity States

An entity can be in different states that affect how their tick is processed:

| State | Description | Tick Behavior |
|-------|-------------|---------------|
| **Idle** | Going about daily life | Light tick, routine actions |
| **In Mission** | Traveling, questing | Progress mission, check for events |
| **In Conversation** | Talking to player/entity | Full LLM response |
| **In Combat** | Fighting | Tactical decisions |
| **Sleeping** | Resting | Skip or minimal tick |
| **Dead** | No longer alive | No tick |

---

## Mission System

When an entity is on a mission (e.g., traveling between planets):

```
Mission: Travel from Keltar Prime to Earth
├── Started: Day 145
├── Duration: 30 days
├── Current Progress: Day 12
├── Ship: Cargo Freighter "Dawn Runner"
└── Events Queue:
    ├── Day 15: Asteroid field (roll for danger)
    ├── Day 20: Encounter another ship (random)
    └── Day 30: Arrival

On each tick:
  1. Advance mission progress
  2. Check for scheduled events
  3. Ask LLM: "You're on day 12 of your journey. What do you do?"
  4. Entity might: rest, maintain ship, reflect, plan, etc.
```

---

## Available Actions

Each entity has a set of actions it can perform:

### Universal Actions
- `speak(target, message)` — Talk to another entity or player
- `move(destination)` — Go to a location
- `observe()` — Look around, gather information
- `wait(duration)` — Do nothing for a while
- `think()` — Internal reflection (stores to memory)

### Contextual Actions
- `trade(target, offer, request)` — Exchange goods
- `attack(target)` — Initiate combat
- `flee()` — Run away
- `hide()` — Become hard to find
- `craft(item)` — Create something
- `use(item)` — Use an inventory item

### Social Actions
- `befriend(target)` — Improve relationship
- `threaten(target)` — Intimidate
- `lie(target, falsehood)` — Deceive
- `recruit(target, cause)` — Ask to join

---

## The Think Function (LLM Call)

The core of entity intelligence — a call to the LLM:

```typescript
async function think(entity: Entity, context: TickContext): Promise<Action> {
  
  const prompt = buildPrompt({
    // Who am I?
    identity: entity.systemPrompt,
    personality: entity.personalityTraits,
    drive: entity.drive,
    
    // What do I know?
    memories: entity.recentMemories,
    relationships: entity.relationships,
    inventory: entity.inventory,
    
    // What's happening?
    currentLocation: context.location,
    nearbyEntities: context.nearbyEntities,
    recentEvents: context.recentEvents,
    currentMission: entity.activeMission,
    
    // What can I do?
    availableActions: getAvailableActions(entity, context),
  });

  const response = await llm.complete(prompt);
  
  return parseAction(response);
}
```

---

## Keeping It Simple (Why Not OpenClaw)

OpenClaw is powerful but heavy — it has:
- Messaging integrations (WhatsApp, Discord, etc.)
- Cron systems
- Node networks
- Session management
- Too much infrastructure for game NPCs

For Multi-AI entities, we need:
- Simple agent loop (pi-agent-core)
- LLM API (pi-ai)
- Action execution
- Memory persistence
- That's it.

**Philosophy:** Each entity is a lightweight agent that can:
1. Receive context (what's happening)
2. Think (LLM call)
3. Act (execute action)
4. Remember (persist important stuff)

No need for webhooks, channels, or external integrations. Just pure AI agency.

---

## Open Questions

- [ ] How do entities communicate with each other? (direct function calls vs. message queue?)
- [ ] How do we batch LLM calls for efficiency? (multiple entities in one request?)
- [ ] How detailed should the tick be? (every in-game hour? day?)
- [ ] Should entities have "instincts" (rule-based shortcuts) to reduce LLM calls?
- [ ] How do we handle entity "death" and "birth"?
