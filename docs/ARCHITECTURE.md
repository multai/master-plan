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

**Recommendation: PostgreSQL** — Battle-tested, excellent JSON support, good for both structured and semi-structured data.

### Full Database Schema

```sql
-- ============================================
-- WORLDS & LOCATIONS
-- ============================================

CREATE TABLE star_systems (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  coordinates POINT,
  description TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE planets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  system_id UUID REFERENCES star_systems(id),
  name TEXT NOT NULL,
  description TEXT,
  environment JSONB, -- { climate, gravity, atmosphere, etc. }
  cultures TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  planet_id UUID REFERENCES planets(id),
  name TEXT NOT NULL,
  type TEXT, -- city, wilderness, dungeon, spaceport, etc.
  description TEXT,
  properties JSONB, -- { danger_level, resources, population, etc. }
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE location_connections (
  from_location_id UUID REFERENCES locations(id),
  to_location_id UUID REFERENCES locations(id),
  travel_method TEXT, -- walk, fly, teleport, ship
  travel_time_minutes INT,
  PRIMARY KEY (from_location_id, to_location_id)
);

-- ============================================
-- SPECIES CATALOG
-- ============================================

CREATE TABLE species (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  homeworld_id UUID REFERENCES planets(id),
  physical_description TEXT,
  culture TEXT,
  common_drives TEXT[], -- ['family', 'wealth', 'power']
  traits JSONB, -- { telepathic: true, pack_mentality: true, etc. }
  lifespan_years INT,
  language TEXT,
  base_personality_prompt TEXT, -- LLM system prompt additions for this species
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- CREATURES
-- ============================================

CREATE TYPE creature_status AS ENUM ('alive', 'dead', 'missing', 'imprisoned', 'hibernating');
CREATE TYPE drive_type AS ENUM ('family', 'wealth', 'freedom', 'power', 'knowledge', 'survival', 'revenge', 'love');

CREATE TABLE creatures (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  species_id UUID REFERENCES species(id),
  birth_planet_id UUID REFERENCES planets(id),
  current_location_id UUID REFERENCES locations(id),
  
  -- AI Personality
  drive drive_type NOT NULL,
  personality_traits JSONB, -- { brave: 0.8, greedy: 0.3, kind: 0.6 }
  backstory TEXT,
  system_prompt TEXT, -- Full LLM system prompt for this creature
  
  -- State
  status creature_status DEFAULT 'alive',
  inventory JSONB DEFAULT '[]',
  stats JSONB, -- { health, energy, wealth, reputation, etc. }
  
  -- Timestamps
  born_at TIMESTAMPTZ,
  last_active_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_creatures_location ON creatures(current_location_id);
CREATE INDEX idx_creatures_species ON creatures(species_id);
CREATE INDEX idx_creatures_status ON creatures(status) WHERE status = 'alive';

-- ============================================
-- CREATURE MEMORY (AI Context)
-- ============================================

CREATE TABLE creature_memories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  creature_id UUID REFERENCES creatures(id) ON DELETE CASCADE,
  memory_type TEXT, -- 'event', 'conversation', 'knowledge', 'emotion'
  content TEXT NOT NULL,
  importance FLOAT DEFAULT 0.5, -- 0-1, used for memory pruning
  related_creature_id UUID REFERENCES creatures(id),
  location_id UUID REFERENCES locations(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_memories_creature ON creature_memories(creature_id);
CREATE INDEX idx_memories_importance ON creature_memories(creature_id, importance DESC);

-- ============================================
-- RELATIONSHIPS
-- ============================================

CREATE TYPE relationship_type AS ENUM (
  'family', 'friend', 'enemy', 'rival', 'lover', 
  'employer', 'employee', 'ally', 'acquaintance'
);

CREATE TABLE creature_relationships (
  creature_id UUID REFERENCES creatures(id) ON DELETE CASCADE,
  related_id UUID REFERENCES creatures(id) ON DELETE CASCADE,
  relationship relationship_type NOT NULL,
  strength FLOAT DEFAULT 0.5, -- 0-1
  notes TEXT,
  since TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (creature_id, related_id)
);

-- ============================================
-- PLAYERS (Humans)
-- ============================================

CREATE TABLE players (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username TEXT UNIQUE NOT NULL,
  display_name TEXT,
  email TEXT UNIQUE,
  current_location_id UUID REFERENCES locations(id),
  inventory JSONB DEFAULT '[]',
  stats JSONB,
  preferences JSONB, -- { voice_enabled, image_style, etc. }
  created_at TIMESTAMPTZ DEFAULT NOW(),
  last_seen_at TIMESTAMPTZ
);

-- ============================================
-- WORLD EVENTS (History)
-- ============================================

CREATE TABLE world_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type TEXT, -- 'battle', 'trade', 'birth', 'death', 'discovery'
  description TEXT,
  location_id UUID REFERENCES locations(id),
  participants UUID[], -- creature/player IDs
  outcome JSONB,
  occurred_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_events_location ON world_events(location_id);
CREATE INDEX idx_events_time ON world_events(occurred_at DESC);
```

---

## Mind Pooling System

Running thousands of AI minds simultaneously is expensive. We use a **lazy loading + pooling** architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    MIND POOL MANAGER                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────┐     ┌─────────────────────────┐    │
│  │   ACTIVE MINDS      │     │    SLEEPING MINDS       │    │
│  │   (in memory)       │     │    (in database)        │    │
│  │                     │     │                         │    │
│  │  • Creature A 🧠    │     │  • 10,000 creatures     │    │
│  │  • Creature B 🧠    │     │  • Context serialized   │    │
│  │  • Creature C 🧠    │     │  • Wake on demand       │    │
│  │                     │     │                         │    │
│  │  Max: 50 concurrent │     │                         │    │
│  └─────────────────────┘     └─────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  LRU EVICTION                        │    │
│  │  When pool full: serialize oldest mind → database    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Implementation

```typescript
interface MindState {
  creatureId: string;
  conversationHistory: Message[];
  shortTermMemory: string[];
  lastActiveAt: Date;
}

class MindPool {
  private activeMinds: Map<string, MindState> = new Map();
  private maxActive: number = 50;

  async getOrWake(creatureId: string): Promise<MindState> {
    // Check if already active
    if (this.activeMinds.has(creatureId)) {
      const mind = this.activeMinds.get(creatureId)!;
      mind.lastActiveAt = new Date();
      return mind;
    }

    // Evict if pool is full
    if (this.activeMinds.size >= this.maxActive) {
      await this.evictOldest();
    }

    // Wake from database
    return await this.wakeFromDB(creatureId);
  }

  private async evictOldest(): Promise<void> {
    let oldest: [string, MindState] | null = null;
    
    for (const entry of this.activeMinds) {
      if (!oldest || entry[1].lastActiveAt < oldest[1].lastActiveAt) {
        oldest = entry;
      }
    }

    if (oldest) {
      await this.serializeToDB(oldest[0], oldest[1]);
      this.activeMinds.delete(oldest[0]);
    }
  }

  private async serializeToDB(id: string, mind: MindState): Promise<void> {
    // Save conversation history and memory to creature_memories table
    // Compress if needed
  }

  private async wakeFromDB(creatureId: string): Promise<MindState> {
    // Load creature + recent memories from DB
    // Reconstruct conversation context
    // Return hydrated MindState
  }
}
```

---

## Background Simulation (World Tick System)

Creatures should *live* even when no player is watching. We use a **tick system**:

```
┌──────────────────────────────────────────────────────────────┐
│                    WORLD TICK SYSTEM                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Every N minutes (configurable):                              │
│                                                               │
│  1. SELECT creatures needing simulation:                      │
│     - Important creatures (high influence)                    │
│     - Creatures with pending goals                            │
│     - Random sample of others                                 │
│                                                               │
│  2. For each selected creature:                               │
│     - Wake mind from pool                                     │
│     - Ask: "What do you do next?"                            │
│     - Execute actions (move, trade, talk to others)           │
│     - Save results to DB                                      │
│     - Generate world events if significant                    │
│                                                               │
│  3. Process consequences:                                     │
│     - Relationships change                                    │
│     - Resources shift                                         │
│     - News spreads                                            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Tick Priority System

```typescript
interface TickPriority {
  creatureId: string;
  priority: number; // 0-100
  reason: string;
}

function calculateTickPriority(creature: Creature): number {
  let priority = 0;

  // Important characters tick more often
  if (creature.stats.influence > 0.8) priority += 30;
  
  // Creatures with active goals
  if (creature.pendingActions.length > 0) priority += 25;
  
  // Creatures in populated areas (more interactions)
  const locationPopulation = getLocationPopulation(creature.currentLocationId);
  priority += locationPopulation * 0.1;
  
  // Recently active creatures stay active
  const hoursSinceActive = hoursSince(creature.lastActiveAt);
  if (hoursSinceActive < 1) priority += 20;
  
  // Random factor for variety
  priority += Math.random() * 10;
  
  return Math.min(priority, 100);
}
```

### Lightweight Simulation Mode

For massive scale, not every tick needs full LLM inference:

```typescript
async function simulateTick(creature: Creature): Promise<void> {
  const complexity = determineTickComplexity(creature);

  switch (complexity) {
    case 'full':
      // Full LLM inference - creature makes meaningful decision
      await fullMindTick(creature);
      break;
    
    case 'routine':
      // Simple rule-based - creature continues routine
      await routineTick(creature);
      break;
    
    case 'idle':
      // No action - creature is resting/waiting
      await idleTick(creature);
      break;
  }
}
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
