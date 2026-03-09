# Technical Architecture

> *How the pieces fit together*

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           MULTI-AI SYSTEM                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────────┐         ┌──────────────────────────────────┐      │
│  │   WEB CLIENT     │         │         GAME SERVER              │      │
│  │   (Browser)      │◄──────►│         (Node.js)                │      │
│  │                  │   WS    │                                  │      │
│  │  • React/Next.js │         │  • WebSocket Server              │      │
│  │  • Butley UI kit │         │  • REST API                      │      │
│  │  • Real-time     │         │  • Session Management            │      │
│  └──────────────────┘         │  • World State                   │      │
│                               └──────────────┬───────────────────┘      │
│                                              │                          │
│                                              │                          │
│                               ┌──────────────┴───────────────────┐      │
│                               │         UNIVERSE ENGINE          │      │
│                               │                                  │      │
│                               │  • Tick System                   │      │
│                               │  • Plot Engine                   │      │
│                               │  • Entity Manager                │      │
│                               │  • Time Controller               │      │
│                               └──────────────┬───────────────────┘      │
│                                              │                          │
│                    ┌─────────────────────────┼─────────────────────┐    │
│                    │                         │                     │    │
│           ┌────────▼────────┐    ┌───────────▼────────┐   ┌───────▼──┐ │
│           │  ENTITY POOL    │    │     DATABASE       │   │   LLM    │ │
│           │  (pi-agent)     │    │   (PostgreSQL)     │   │  GATEWAY │ │
│           │                 │    │                    │   │          │ │
│           │ • Mind instances│    │ • Creatures        │   │ • pi-ai  │ │
│           │ • LRU cache     │    │ • Players          │   │ • Multi  │ │
│           │ • Wake/Sleep    │    │ • Worlds           │   │   provider│ │
│           └─────────────────┘    │ • Plots            │   └──────────┘ │
│                                  │ • Events           │                 │
│                                  └────────────────────┘                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Components

### 1. Web Client (Frontend)

**Stack:** React + Next.js (same as Butley)

```
/client
├── /components
│   ├── /ui          # Butley UI kit (buttons, cards, etc.)
│   ├── /game        # Game-specific components
│   │   ├── WorldMap.tsx
│   │   ├── CharacterPanel.tsx
│   │   ├── ChatInterface.tsx
│   │   ├── InventoryView.tsx
│   │   └── LocationView.tsx
│   └── /common
├── /hooks
│   ├── useWebSocket.ts
│   ├── useGameState.ts
│   └── usePlayer.ts
├── /pages
│   ├── index.tsx        # Landing
│   ├── login.tsx        # Auth
│   ├── create.tsx       # Character creation
│   └── play.tsx         # Main game interface
└── /lib
    ├── api.ts           # REST client
    ├── socket.ts        # WebSocket client
    └── types.ts         # Shared types
```

**Key Features:**
- Real-time updates via WebSocket
- Text-based interface (like a MUD/interactive fiction)
- Generated images for locations, creatures
- Voice input/output (optional)
- Mobile responsive

### 2. Game Server (Backend)

**Stack:** Node.js + TypeScript

```
/server
├── /api
│   ├── /routes
│   │   ├── auth.ts          # Login, register
│   │   ├── characters.ts    # Character CRUD
│   │   ├── world.ts         # Location info
│   │   └── actions.ts       # Player actions
│   └── /middleware
│       ├── auth.ts
│       └── rateLimit.ts
├── /websocket
│   ├── server.ts            # WS connection handler
│   ├── handlers.ts          # Message handlers
│   └── rooms.ts             # Location-based rooms
├── /engine
│   ├── universe.ts          # Main tick loop
│   ├── time.ts              # Time management
│   ├── plots.ts             # Plot engine
│   └── events.ts            # Event system
├── /entities
│   ├── pool.ts              # Entity pool (mind management)
│   ├── mind.ts              # pi-agent wrapper
│   └── actions.ts           # Entity action execution
├── /db
│   ├── client.ts            # Database connection
│   ├── queries.ts           # SQL queries
│   └── migrations/          # Schema migrations
└── /services
    ├── llm.ts               # pi-ai integration
    ├── imageGen.ts          # Image generation
    └── tts.ts               # Text-to-speech
```

---

## Communication

### WebSocket Protocol

**Connection:**
```typescript
// Client connects
ws://game.multai.com/play?token=JWT_TOKEN

// Server authenticates and sends initial state
{
  type: "INIT",
  payload: {
    character: { id, name, location, status },
    location: { id, name, description, entities, exits },
    time: { gameTime, realTime },
    messages: [ /* recent chat history */ ]
  }
}
```

**Client → Server Messages:**

```typescript
// Player action
{
  type: "ACTION",
  payload: {
    action: "speak" | "move" | "look" | "use" | "trade" | ...,
    target?: string,      // Entity/item/location ID
    data?: any            // Action-specific data
  }
}

// Chat message
{
  type: "CHAT",
  payload: {
    message: "Hello, merchant!",
    target: "entity_123"  // Who we're talking to
  }
}

// Time skip request
{
  type: "SKIP",
  payload: {
    engagement: "full" | "highlights" | "skip"
  }
}
```

**Server → Client Messages:**

```typescript
// World update
{
  type: "UPDATE",
  payload: {
    location?: LocationState,
    entities?: EntityState[],
    time?: TimeState,
    events?: GameEvent[]
  }
}

// Entity response (NPC talking)
{
  type: "ENTITY_RESPONSE",
  payload: {
    entityId: "entity_123",
    message: "Greetings, human. What brings you here?",
    emotion: "curious",
    actions?: ["offers trade", "steps closer"]
  }
}

// Narrative (story text)
{
  type: "NARRATIVE",
  payload: {
    text: "The station hums with activity...",
    image?: "https://...",  // Generated image URL
    audio?: "https://..."   // Optional narration
  }
}

// Time skip event
{
  type: "SKIP_EVENT",
  payload: {
    day: 3,
    description: "A fellow passenger approaches...",
    choices: ["Engage", "Ignore", "Skip"]
  }
}
```

### REST API

For non-real-time operations:

```
POST   /api/auth/login
POST   /api/auth/register
GET    /api/auth/me

GET    /api/characters
POST   /api/characters
GET    /api/characters/:id
DELETE /api/characters/:id

GET    /api/world/locations/:id
GET    /api/world/species
GET    /api/world/factions

GET    /api/player/inventory
GET    /api/player/relationships
GET    /api/player/history
```

---

## Player Session Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PLAYER SESSION FLOW                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. CONNECT                                                              │
│     ├── Open WebSocket connection with JWT                               │
│     ├── Server validates, loads character                                │
│     └── Server sends INIT with current state                             │
│                                                                          │
│  2. LOCATION SYNC                                                        │
│     ├── Server adds player to location "room"                            │
│     ├── Other players in location notified                               │
│     └── Nearby entities become aware of player                           │
│                                                                          │
│  3. GAMEPLAY LOOP                                                        │
│     │                                                                    │
│     ├── Player sends ACTION                                              │
│     │   └── Server processes, updates world                              │
│     │   └── Server broadcasts updates to affected clients                │
│     │                                                                    │
│     ├── Player sends CHAT (to entity)                                    │
│     │   └── Server wakes entity mind (from pool)                         │
│     │   └── Entity thinks (LLM call)                                     │
│     │   └── Server sends ENTITY_RESPONSE                                 │
│     │                                                                    │
│     ├── Player requests SKIP (travel/wait)                               │
│     │   └── Server processes time skip                                   │
│     │   └── Server generates events                                      │
│     │   └── Server sends SKIP_EVENTs or final state                      │
│     │                                                                    │
│     └── Server pushes updates (other players, world events)              │
│                                                                          │
│  4. DISCONNECT                                                           │
│     ├── Player closes connection                                         │
│     ├── Server checks location safety                                    │
│     ├── Character marked as resting/vulnerable                           │
│     └── Character continues in world (AI or frozen)                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Entity (NPC) System

### pi-agent Integration

Each entity mind runs via pi-agent-core:

```typescript
import { Agent } from '@mariozechner/pi-agent-core';
import { createLLM } from '@mariozechner/pi-ai';

class EntityMind {
  private agent: Agent;
  private entity: Entity;
  
  constructor(entity: Entity) {
    this.entity = entity;
    this.agent = new Agent({
      llm: createLLM({ 
        provider: 'anthropic', 
        model: 'claude-3-5-sonnet' 
      }),
      systemPrompt: this.buildSystemPrompt(entity),
      tools: this.getEntityTools(),
    });
  }
  
  private buildSystemPrompt(entity: Entity): string {
    return `
You are ${entity.name}, a ${entity.species.name} from ${entity.homeworld}.

PERSONALITY:
${entity.personalityPrompt}

DRIVE: ${entity.drive}

CURRENT SITUATION:
- Location: ${entity.currentLocation.description}
- Status: ${entity.status}
- Current goal: ${entity.currentGoal || 'None'}

MEMORIES:
${entity.recentMemories.join('\n')}

Respond in character. Be consistent with your personality and goals.
    `.trim();
  }
  
  private getEntityTools(): Tool[] {
    return [
      {
        name: 'speak',
        description: 'Say something to someone nearby',
        parameters: { target: 'string', message: 'string' }
      },
      {
        name: 'move',
        description: 'Move to a connected location',
        parameters: { destination: 'string' }
      },
      {
        name: 'observe',
        description: 'Look around and gather information',
        parameters: {}
      },
      // ... more tools
    ];
  }
  
  async think(context: ThinkContext): Promise<Action> {
    const response = await this.agent.run(context.prompt);
    return this.parseAction(response);
  }
  
  async respond(playerMessage: string): Promise<string> {
    const response = await this.agent.run(
      `A human player says: "${playerMessage}"\n\nRespond in character.`
    );
    return response.text;
  }
}
```

### Entity Pool Management

```typescript
class EntityPool {
  private active: Map<string, EntityMind> = new Map();
  private maxActive = 100;
  
  async get(entityId: string): Promise<EntityMind> {
    if (this.active.has(entityId)) {
      return this.active.get(entityId)!;
    }
    
    if (this.active.size >= this.maxActive) {
      await this.evictLRU();
    }
    
    const entity = await db.getEntity(entityId);
    const mind = new EntityMind(entity);
    this.active.set(entityId, mind);
    return mind;
  }
  
  private async evictLRU(): Promise<void> {
    // Find least recently used, serialize state, remove
  }
}
```

---

## Database Schema Summary

```sql
-- Core tables (see ARCHITECTURE.md for full schema)

-- Players (human accounts)
players (id, username, email, created_at, ...)

-- Characters (player-controlled entities)
characters (id, player_id, name, species_id, location_id, status, ...)

-- Creatures (AI-controlled entities)
creatures (id, name, species_id, location_id, drive, personality, ...)

-- Shared entity data
entity_memories (entity_id, content, importance, created_at)
entity_relationships (entity_id, related_id, type, strength)
entity_inventory (entity_id, item_id, quantity)

-- World
galaxies, sectors, star_systems, planets, locations
species, factions, religions

-- Narrative
plots (id, name, type, scale, status, ...)
plot_participants (plot_id, entity_id, role, faction)
world_events (id, type, description, location_id, occurred_at)

-- Sessions
player_sessions (id, player_id, character_id, connected_at, location_id)
```

---

## Deployment

### Initial Setup (Single Server)

```
┌─────────────────────────────────────────┐
│           SINGLE SERVER                  │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Docker Compose                     │ │
│  │                                     │ │
│  │  ┌─────────────┐ ┌──────────────┐  │ │
│  │  │ Game Server │ │  PostgreSQL  │  │ │
│  │  │   (Node)    │ │              │  │ │
│  │  └─────────────┘ └──────────────┘  │ │
│  │                                     │ │
│  │  ┌─────────────┐ ┌──────────────┐  │ │
│  │  │   Nginx     │ │    Redis     │  │ │
│  │  │  (reverse   │ │  (sessions)  │  │ │
│  │  │   proxy)    │ │              │  │ │
│  │  └─────────────┘ └──────────────┘  │ │
│  │                                     │ │
│  └────────────────────────────────────┘ │
│                                          │
└─────────────────────────────────────────┘
```

### Scaled Setup (Future)

```
┌──────────────────────────────────────────────────────┐
│                    SCALED SETUP                       │
│                                                       │
│  ┌─────────────────┐                                 │
│  │  Load Balancer  │                                 │
│  └────────┬────────┘                                 │
│           │                                          │
│  ┌────────┼────────────────────────┐                │
│  │        │                        │                │
│  ▼        ▼                        ▼                │
│ ┌──────┐ ┌──────┐              ┌──────┐            │
│ │ WS 1 │ │ WS 2 │    ...       │ WS N │            │
│ └──────┘ └──────┘              └──────┘            │
│                                                      │
│  ┌──────────────────────────────────────┐           │
│  │          Redis Cluster               │           │
│  │  (pub/sub for cross-server events)   │           │
│  └──────────────────────────────────────┘           │
│                                                      │
│  ┌──────────────────────────────────────┐           │
│  │         PostgreSQL Cluster           │           │
│  └──────────────────────────────────────┘           │
│                                                      │
│  ┌──────────────────────────────────────┐           │
│  │       Universe Engine Workers        │           │
│  │  (tick processing, entity AI)        │           │
│  └──────────────────────────────────────┘           │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Tech Stack Summary

| Component | Technology | Notes |
|-----------|------------|-------|
| **Frontend** | React + Next.js | Same as Butley, shared UI kit |
| **Backend** | Node.js + TypeScript | Fast, good WebSocket support |
| **Database** | PostgreSQL | Robust, good for complex queries |
| **Cache** | Redis | Sessions, pub/sub |
| **WebSocket** | ws (npm) or Socket.io | Real-time communication |
| **Entity AI** | pi-agent-core + pi-ai | Lightweight agent framework |
| **LLM** | Multiple (Claude, GPT, etc.) | Via pi-ai unified API |
| **Images** | Stable Diffusion / DALL-E | Generated on demand |
| **Hosting** | Docker + VPS | Start simple, scale later |

---

## Development Phases

### Phase 1: Core
- [ ] Basic server with WebSocket
- [ ] PostgreSQL schema
- [ ] Character creation
- [ ] Single location prototype
- [ ] One NPC with pi-agent

### Phase 2: World
- [ ] Multiple locations
- [ ] Movement system
- [ ] Time system
- [ ] Multiple NPCs

### Phase 3: Gameplay
- [ ] Actions (trade, combat, etc.)
- [ ] Inventory system
- [ ] Plot system basics
- [ ] Multiplayer (same location)

### Phase 4: Polish
- [ ] Image generation
- [ ] Voice input/output
- [ ] Mobile optimization
- [ ] Performance tuning

---

## Open Questions

- [ ] Hosting provider? (Hetzner like Butley? AWS? DigitalOcean?)
- [ ] Image generation service? (Self-hosted SD? OpenAI? Replicate?)
- [ ] Voice? (Elevenlabs? OpenAI TTS?)
- [ ] Authentication? (Email/password? OAuth? Magic links?)
