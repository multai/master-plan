# Plot Service — Architecture

> *The bridge between the player's browser and the living world*

---

## Overview

The **plot-service** is a Node.js process that runs a single plot instance, connecting players to AI-powered characters in real-time via WebSocket.

### System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INFRASTRUCTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   [user-frontend]                                                            │
│         │                                                                    │
│         │ wss://play.multai.xyz/plot/{plotId}                               │
│         ▼                                                                    │
│   ┌──────────┐     Docker labels      ┌─────────────────────────────────┐   │
│   │ TRAEFIK  │ ◄─────────────────────►│  plot-service-{plotId}          │   │
│   │ (router) │     auto-discovery     │  (container)                    │   │
│   └──────────┘                        └─────────────────────────────────┘   │
│         ▲                                                                    │
│         │ POST /plots/{id}/start                                            │
│   ┌─────┴──────────┐                                                        │
│   │ PLOT-          │     Manages container lifecycle                        │
│   │ ORCHESTRATOR   │     (spawn, stop, health, upgrade)                     │
│   └────────────────┘                                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Repositories

| Repository | Purpose | Status |
|------------|---------|--------|
| `plot-service` | WebSocket server, narrator, scene management | 📋 New |
| `plot-orchestrator` | Container lifecycle, Traefik labels, API | 📋 New |
| `entity` | Mind class, pi-agent wrapper | ✅ Exists |
| `entity-engine` | LLM tick loop for focused entities | ✅ Exists |

---

## Plot-Orchestrator

A separate service (own repo) that manages plot-service container lifecycle.

### Responsibilities

- Spawn plot-service containers on demand
- Configure Traefik labels for dynamic routing
- Health checks and auto-restart
- Upgrade/rollback deployments
- Resource limits and cleanup

### API

```
POST   /plots/{plotId}/start    → Spawn container, return WS URL
DELETE /plots/{plotId}          → Stop and remove container
GET    /plots/{plotId}/status   → Health, uptime, player count
POST   /plots/{plotId}/restart  → Restart without losing state
GET    /plots                   → List active plots
```

### Traefik Integration

When spawning a container, the orchestrator applies Docker labels:

```python
labels = {
    "traefik.enable": "true",
    "traefik.docker.network": "multai-network",
    
    # HTTP routing
    f"traefik.http.routers.plot-{plot_id}.rule": 
        f"Host(`play.multai.xyz`) && PathPrefix(`/plot/{plot_id}`)",
    f"traefik.http.routers.plot-{plot_id}.entrypoints": "websecure",
    f"traefik.http.routers.plot-{plot_id}.middlewares": f"plot-{plot_id}-strip",
    
    # Strip prefix before forwarding
    f"traefik.http.middlewares.plot-{plot_id}-strip.stripprefix.prefixes": 
        f"/plot/{plot_id}",
    
    # Target port inside container
    f"traefik.http.services.plot-{plot_id}.loadbalancer.server.port": "8080",
    
    # Metadata
    "multai.plot_id": plot_id,
    "multai.component": "plot-service",
}
```

Traefik auto-discovers containers via Docker provider — zero manual config per plot.

### Traefik Config (docker-compose)

```yaml
services:
  traefik:
    image: traefik:v2.11
    command:
      - --api.dashboard=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --providers.docker.network=multai-network
      - --entrypoints.websecure.address=:443
      # Long timeouts for WebSocket
      - --entrypoints.websecure.transport.respondingtimeouts.readtimeout=3600s
      - --entrypoints.websecure.transport.respondingtimeouts.writetimeout=3600s
    ports:
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - multai-network
```

---

## Plot-Service Internal Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PLOT SERVICE                                 │
│                    (one container per plot)                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────┐         ┌────────────────────────────────┐  │
│  │   WebSocket Server │◄───────►│        Player Client(s)        │  │
│  │   (port 8080)      │         │        (user-frontend)         │  │
│  └────────┬───────────┘         └────────────────────────────────┘  │
│           │                                                          │
│           ▼                                                          │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                       NARRATOR                                │   │
│  │              (the story's Game Master)                        │   │
│  │                                                               │   │
│  │  • Describes scenes and atmosphere                           │   │
│  │  • Advances timeline and pacing                              │   │
│  │  • Triggers scheduled plot events                            │   │
│  │  • Maintains narrative coherence                             │   │
│  │  • Handles scene transitions                                 │   │
│  └──────────────────────────────────────────────────────────────┘   │
│           │                                                          │
│           ▼                                                          │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    SCENE MANAGER                              │   │
│  │                                                               │   │
│  │  • Current location and environment                          │   │
│  │  • Present entities (NPCs + player)                          │   │
│  │  • Dialogue history                                          │   │
│  │  • World state snapshot                                      │   │
│  └──────────┬───────────────────────────┬───────────────────────┘   │
│             │                           │                            │
│             ▼                           ▼                            │
│  ┌──────────────────┐       ┌──────────────────────┐                │
│  │  ENTITY ENGINE   │       │  GLOBAL ENGINE       │                │
│  │  (LLM-powered)   │       │  (deterministic)     │                │
│  │                  │       │                      │                │
│  │  • NPC minds     │       │  • Background life   │                │
│  │  • Conversations │       │  • Needs/routines    │                │
│  │  • Decisions     │       │  • Simple movement   │                │
│  └──────────────────┘       └──────────────────────┘                │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                       STORAGE                                │   │
│  │  Convex: messages, player state, real-time sync              │   │
│  │  Local: entity mind state (serialized between ticks)         │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## The Narrator

The Narrator is a **special Entity** — an invisible, omniscient presence that serves as the automated Game Master.

> 📄 **Full documentation:** [NARRATOR.md](./NARRATOR.md)

### Key Insight: Narrator as Entity

The Narrator uses the **same infrastructure as NPCs** (Mind class, pi-agent, memory system) but with special properties:

```typescript
interface NarratorEntity extends Entity {
  type: "narrator";
  
  // What makes it special
  visibility: "hidden";       // NPCs don't see it in nearbyEntities
  knowledge: "omniscient";    // Knows all plot secrets
  voice: "third-person";      // Describes, doesn't dialogue
  drive: "storytelling";      // Goal: compelling narrative
  
  // Untargetable
  targetable: false;
  mortal: false;
  physical: false;            // No body, exists everywhere
  
  // But still has a Mind
  mind: Mind;                 // Same pi-agent, different prompt
}
```

### Why This Design?

- **Uniformity** — Same processing pipeline for all "brains"
- **Memory** — Narrator remembers what it narrated, avoids repetition
- **Tools** — Uses actions like any entity (`describe_scene`, `advance_time`)
- **Simplicity** — One entity system, not two separate systems

### What Entities See

```typescript
// When building NPC context:
const nearbyEntities = allEntities.filter(e => 
  e.visibility !== "hidden"
);
// Result: NPCs never know the Narrator exists

// But the system processes everyone:
const allToProcess = [player, ...npcs, narrator];
```

### Narrator Responsibilities

| Domain | What it does |
|--------|--------------|
| **Atmosphere** | Describes scenes, environment, mood, sensory details |
| **Timeline** | Advances time, compresses downtime ("Three hours later...") |
| **Plot Events** | Triggers scheduled events, introduces complications |
| **Coherence** | Maintains facts, prevents contradictions |
| **Transitions** | Handles scene changes, introduces/removes NPCs |
| **Pacing** | Adjusts tension, prevents stagnation |

### Example Flow

```
[NARRATOR] The cargo bay of the Dawn Runner is dimly lit, 
           emergency lights casting long shadows across stacked crates.

[PLAYER]   I approach Vex. "We need to talk."

[VEX]      *looks up from the datapad, expression guarded*
           "About what? I'm busy with inventory."

[NARRATOR] You notice Vex's hand trembles slightly as he sets down the datapad.

[PLAYER]   "I saw the manifest. Those crates aren't medical supplies."

[NARRATOR] The ship shudders — you've dropped out of hyperspace.
           Through the viewport, an unfamiliar planet looms.

[VEX]      *stands abruptly* 
           "We weren't supposed to arrive for another six hours."

[NARRATOR] Proximity alarms blare. Two smaller vessels are approaching fast.
```

### Processing Order

```
1. Player acts
2. Narrator checks scheduled events (pre-response)
3. NPCs respond (target first, then bystanders)
4. Narrator observes and maybe adds detail
5. Check for scene transition
```

The Narrator participates in the same tick loop as NPCs, but its output goes to `NARRATOR` message type instead of `ENTITY_RESPONSE`.

---

## WebSocket Protocol

### Client → Server

```typescript
// Player speaks or acts
{
  type: "PLAYER_ACTION",
  payload: {
    action: "speak" | "observe" | "move" | "use" | "attack" | "trade",
    content?: string,       // What the player says/does
    targetId?: string,      // NPC id or item id
    targetName?: string,    // "Vex", "door", "crates"
  }
}

// Player requests time skip
{ type: "SKIP", payload: { mode: "highlights" | "skip" } }

// Heartbeat
{ type: "PING" }
```

### Server → Client

```typescript
// Initial state on connect
{
  type: "INIT",
  payload: {
    plotId: string,
    plotTitle: string,
    scene: {
      location: Location,
      description: string,
      characters: EntitySummary[],
      playerRole: string,
    },
    recentMessages: Message[],
    gameTime: string,
  }
}

// Narrator speaks (descriptions, atmosphere, events)
{
  type: "NARRATOR",
  payload: {
    text: string,
    mood?: "neutral" | "tense" | "calm" | "urgent" | "mysterious",
  }
}

// NPC responds
{
  type: "ENTITY_RESPONSE",
  payload: {
    entityId: string,
    entityName: string,
    speech?: string,
    action?: string,        // *looks away nervously*
    emotion?: string,
    targetId?: string,
  }
}

// Time passes
{
  type: "TIME_SKIP",
  payload: {
    from: string,
    to: string,
    summary: string,        // "The night passes uneventfully..."
  }
}

// Location changes
{
  type: "SCENE_CHANGE",
  payload: {
    location: Location,
    description: string,
    characters: EntitySummary[],
  }
}

// Major plot moment
{
  type: "PLOT_EVENT",
  payload: {
    title: string,
    description: string,
  }
}

// Heartbeat / Error
{ type: "PONG" }
{ type: "ERROR", payload: { code: string, message: string } }
```

---

## Player Connection Flow

```
1. Player opens /play/{plotId} in browser
2. Frontend connects: wss://play.multai.xyz/plot/{plotId}
3. Traefik routes to correct plot-service container
4. Plot-service authenticates (token from Convex session)
5. Plot-service loads plot state + wakes entity minds
6. Server sends INIT with scene description
7. Player is live in the story
```

---

## Action Processing Flow

```
Player: "Vex, I know you're hiding something."
│
├─ 1. NARRATOR checks scheduled events
│     → None triggered by this dialogue
│
├─ 2. INJECT into WorldState
│     dialogueHistory += { speaker: "Kira", text: "..." }
│     recentEvents += { type: "confrontation", target: "Vex" }
│
├─ 3. DETERMINE responders
│     Target: Vex (directly addressed)
│     Bystanders: Theron, Bram (in scene, may react)
│
├─ 4. ENTITY-ENGINE processes (cascade order)
│     
│     First: Vex.mind.think() 
│       → speech: "Hide? I'm just a trader..."
│       → emotion: nervous
│     
│     Then (parallel): Theron.mind.think(), Bram.mind.think()
│       → Theron: observes silently
│       → Bram: shifts uncomfortably
│
├─ 5. NARRATOR adds color (optional)
│     → "You notice Vex's hand trembles as he speaks."
│
├─ 6. BROADCAST via WebSocket
│     → ENTITY_RESPONSE (Vex)
│     → NARRATOR (physical detail)
│     → ENTITY_RESPONSE (Theron, Bram - if they acted)
│
├─ 7. PERSIST to Convex
│     → All messages saved for history
│
└─ 8. UPDATE state
      Vex.memory += "Kira confronted me about the cargo"
      Theron.memory += "Kira suspects Vex"
      plotState.tension += 0.1
```

---

## Turn Order (Reactive Cascade)

Not round-robin. Responses flow naturally:

```
1. PLAYER acts
2. TARGET responds (whoever was addressed)
3. BYSTANDERS react in parallel (may speak, observe, or ignore)
4. NARRATOR adds context if needed
```

If player addresses no one specifically ("I look around"), all present entities are bystanders.

---

## Autonomous Ticks

NPCs don't freeze when the player is idle.

### Tick Frequency (adaptive)

| Player State | Tick Interval | Behavior |
|--------------|---------------|----------|
| Active (recent input) | 5-10s | Full NPC autonomy |
| Idle (>2 min) | 30-60s | Reduced, ambient only |
| AFK (>10 min) | 5 min | Minimal, or pause |

### What can happen autonomously

- NPC starts conversation with player
- NPCs talk to each other
- Environment changes (weather, lighting)
- Scheduled plot events trigger
- Tension escalation hints

### What won't happen to AFK player

- NPCs won't attack without plot justification
- Player won't be robbed or killed
- Major plot points wait for player engagement

---

## Entity Ownership

Each NPC belongs to **one plot-service at a time**.

### Rules

- **Fixed NPC** (bartender, shopkeeper): owned by location's plot
- **Mobile NPC** (traveling merchant): ownership transfers with movement
- **Shared appearance**: use "shadow" (read-only copy, no mind.think())

### Shadow Entities

When an NPC needs to appear in a secondary plot:

```typescript
interface ShadowEntity {
  sourceEntityId: string;
  ownerPlotId: string;        // Who runs the real mind
  
  // Read-only snapshot
  name: string;
  appearance: string;
  lastKnownState: string;
  
  // Behavior
  responses: "generic" | "busy" | "absent";
}
```

Shadow responds with canned lines: "He seems distracted..." or redirects to owner plot.

---

## Directory Structure

### plot-service

```
plot-service/
├── src/
│   ├── index.ts           # Entry point
│   ├── server.ts          # HTTP + WebSocket server
│   ├── narrator.ts        # The Game Master
│   ├── scene.ts           # Scene/location state
│   ├── orchestrator.ts    # Routes input → engines → output
│   ├── player.ts          # Human-controlled entity wrapper
│   ├── ws-protocol.ts     # Message type definitions
│   ├── storage.ts         # Convex + local persistence
│   └── types.ts
├── package.json
├── Dockerfile
└── README.md
```

### plot-orchestrator

```
plot-orchestrator/
├── src/
│   ├── main.py            # FastAPI app
│   ├── config.py          # Settings (Traefik domain, etc.)
│   ├── api/
│   │   └── routers/
│   │       └── plots.py   # /plots endpoints
│   ├── services/
│   │   ├── container_manager.py   # Docker operations
│   │   └── health.py              # Health checks
│   └── models.py
├── deploy/
│   ├── docker-compose.yml
│   └── traefik/
│       └── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## Scaling Model

| Scale | Plot Services | Entity Engines | Global Engine | Players |
|-------|---------------|----------------|---------------|---------|
| **MVP** | 1 | 1 (embedded) | None | 1 |
| **Small** | 5-10 | 1 per plot | 1 shared | 1-5 per plot |
| **Medium** | 50+ | 1 per plot | 1 shared | N per plot |
| **Large** | 100+ | N per zone | 1 | N |

---

## Decisions Log

| Question | Decision | Rationale |
|----------|----------|-----------|
| WS URL discovery | Traefik dynamic routing | Zero config, proven in Butley |
| Turn order | Reactive cascade | Natural conversation flow |
| Player AFK | Throttled autonomous ticks | World stays alive, saves tokens |
| Entity sharing | Exclusive ownership + shadow | Avoids state conflicts |

---

## Open Questions (Remaining)

- [ ] How does narrator LLM differ from entity LLM? (Same model, different prompt?)
- [ ] State persistence format for plot-service restart?
- [ ] Multi-player sync strategy? (Single writer? CRDT?)
- [ ] Voice integration point? (plot-service or frontend?)

---

*Updated: March 2026*
