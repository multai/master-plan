# Time System

> *The universe never stops. But how fast does it move?*

---

## The Core Problem

**Real-time is too slow.**

- Space travel taking actual weeks = boring
- Wars lasting actual months = players lose interest  
- Character aging in real-time = impractical

**But we want persistence.**

- The universe should feel alive
- Events should unfold whether you're watching or not
- Multiple players sharing the same timeline

---

## Proposed Solution: Hybrid Time

### The Concept

**Different time flows for different situations:**

| Situation | Time Flow | Why |
|-----------|-----------|-----|
| **Direct interaction** | Real-time | Conversations, combat, trading need responsiveness |
| **Travel** | Compressed/Skippable | Nobody wants to wait 3 real days in hyperspace |
| **Offline** | Accelerated | World keeps moving, but faster |
| **Background simulation** | Tick-based | Efficient processing |

### Universal Clock

The universe runs on a **universal tick system**:

```
1 REAL MINUTE = 1 GAME HOUR (60x acceleration)
1 REAL HOUR = ~2.5 GAME DAYS
1 REAL DAY = ~60 GAME DAYS (~2 months)
1 REAL WEEK = ~1 GAME YEAR
```

This means:
- A play session of 2 hours = ~5 game days
- Sleeping for 8 hours = ~20 game days pass
- Wars can unfold over a "week" in a few real days
- Characters can have full lives in months of play

---

## How It Works

### When You're Playing (Active)

```
┌─────────────────────────────────────────────────────────────┐
│                    ACTIVE TIME                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  DIRECT INTERACTION (conversation, combat, trade)           │
│  └── Real-time — you type, they respond immediately         │
│                                                              │
│  TRAVEL                                                      │
│  ├── Short (same planet): Real-time with description        │
│  ├── Medium (same system): Offer skip: "3 hours to arrive"  │
│  └── Long (other systems): Force skip with events           │
│                                                              │
│  WAITING/CRAFTING                                            │
│  └── Offer skip: "This will take 2 hours. Skip?"           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### When You're Offline (Passive)

```
┌─────────────────────────────────────────────────────────────┐
│                   PASSIVE TIME                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  World continues at accelerated pace (60x)                   │
│                                                              │
│  Your character:                                             │
│  ├── In safe zone: Rests, heals, maybe passive income       │
│  └── In danger: Survival AI tries to keep you alive         │
│                                                              │
│  The universe:                                               │
│  ├── Plots advance                                           │
│  ├── NPCs live their lives                                   │
│  ├── Wars are fought                                         │
│  └── Events unfold                                           │
│                                                              │
│  When you return:                                            │
│  └── Summary: "14 game days passed. Here's what happened:"  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Time Skips (The Magic)

The key to making compressed time feel good: **narrative time skips**.

### Example: Space Travel

**Without skip (boring):**
> "You're traveling. Wait 3 real hours."

**With narrative skip (engaging):**
> "The journey to Kepler Station will take 3 days.
> 
> **Day 1:** You settle into your cabin. The ship hums quietly.
> *(Do you want to: Explore the ship / Rest / Skip ahead?)*
> 
> **Day 2:** You meet another passenger in the galley — a nervous merchant named Vor.
> *(Do you want to: Talk to Vor / Ignore him / Skip ahead?)*
> 
> **Day 3:** Alarms blare. The captain announces a pirate sighting.
> *(Encounter begins — real-time)*"

The AI generates interesting moments during skips. You can engage or skip through.

### Skip Controls

```typescript
interface TimeSkipOptions {
  // What's happening
  duration: GameDuration; // "3 days", "2 weeks"
  activity: 'travel' | 'rest' | 'wait' | 'craft' | 'heal';
  
  // Player choices
  engagement: 'full' | 'highlights' | 'skip';
  // full: Experience every generated moment
  // highlights: Only stop for significant events
  // skip: Jump to end immediately
  
  // What can interrupt
  interruptOn: {
    danger: boolean;      // Stop if attacked
    opportunity: boolean; // Stop if something interesting
    message: boolean;     // Stop if contacted
    arrival: boolean;     // Stop when destination reached
  };
}
```

---

## Multiplayer Synchronization

**The hard problem:** Two players in the same place need the same time.

### Solution: Temporal Anchoring

When players are in the same location, they **anchor** to each other:

```
Player A (active) ←→ Player B (active)
         │
         └── Same time flow (real-time)

Player A (active) ←→ Player B (offline)
         │
         └── B is "frozen" or on simple AI
             B experiences time skip when they return
```

### Instanced Time (Optional)

For certain situations, create **time instances**:

```
MAIN TIMELINE
     │
     ├── Player A doing solo exploration (their own time)
     │
     ├── Player B + C in a dungeon (shared instance)
     │
     └── The universe (accelerated background)

When players meet → merge into same time stream
```

---

## Time Zones Across Space

Different parts of the universe might experience time differently:

### Standard Time
- Most of the galaxy
- 60x acceleration
- Synchronized universal clock

### Slow Zones
- Near black holes, ancient artifacts
- Time moves slower
- 30x or even 10x acceleration
- Strategic implications (safe havens?)

### Fast Zones
- Unstable regions
- Time moves faster
- 120x acceleration
- Things change quickly here

### Frozen Zones
- Outside normal spacetime
- Time stopped completely
- Useful for "parking" characters?

---

## The Player Experience

### Logging In After Time Away

> **Welcome back, Captain Zara.**
> 
> You've been resting at Waypoint Station for **18 game days** (7 real hours).
> 
> **What happened while you were away:**
> 
> 📰 **News:**
> - The Merchant Guild announced new tariffs
> - Pirate activity increased in Sector 7
> - The Keltar revolution entered its third month
> 
> 📬 **Messages (3):**
> - Vor the merchant wants to propose a trade route
> - Your faction requests your presence at HQ
> - Unknown sender: "I know what you did on Vega-3"
> 
> 💰 **Finances:**
> - Passive income from investments: +1,200 credits
> - Docking fees: -50 credits
> - Net: +1,150 credits
> 
> 🏥 **Status:**
> - Fully rested and healed
> - Ship fully repaired
> 
> **What would you like to do?**

### During a Time Skip

> **Traveling to the Outer Rim...**
> *Estimated journey: 12 days*
> 
> ▶️ **Day 4** — Encounter Available
> *Your ship's sensors detect a distress signal from a small vessel.*
> 
> [ Investigate ] [ Ignore ] [ Skip All ]

---

## Technical Implementation

### Universal Tick

```typescript
// Core tick system
const TICK_INTERVAL_MS = 60_000; // 1 minute real time
const GAME_HOURS_PER_TICK = 1;   // 1 game hour per tick

class UniverseTime {
  private gameEpoch: Date;  // When the universe "started"
  private currentTick: number;
  
  getCurrentGameTime(): GameDateTime {
    return this.gameEpoch.plus(this.currentTick * GAME_HOURS_PER_TICK);
  }
  
  async processTick(): Promise<void> {
    // 1. Advance world time
    this.currentTick++;
    
    // 2. Process all active plots
    await this.processPlots();
    
    // 3. Process entity ticks (batched)
    await this.processEntities();
    
    // 4. Generate events
    await this.generateEvents();
    
    // 5. Update player states
    await this.updatePlayers();
  }
}
```

### Player Time Skip

```typescript
async function processTimeSkip(
  player: Player, 
  skipRequest: TimeSkipRequest
): Promise<TimeSkipResult> {
  
  const events: SkipEvent[] = [];
  let currentTime = player.gameTime;
  const endTime = skipRequest.endTime;
  
  while (currentTime < endTime) {
    // Check for generated events
    const event = await generateSkipEvent(player, currentTime);
    
    if (event && shouldInterrupt(event, skipRequest.interruptOn)) {
      return {
        completedTime: currentTime,
        interrupted: true,
        interruptEvent: event,
        events,
      };
    }
    
    if (event) events.push(event);
    
    // Advance time
    currentTime = currentTime.plus(1, 'hour');
  }
  
  return {
    completedTime: endTime,
    interrupted: false,
    events,
  };
}
```

---

## Comparison: Other Approaches

| Approach | Pros | Cons |
|----------|------|------|
| **Pure Real-Time** | Immersive, simple | Too slow, boring waits |
| **Pure Turn-Based** | Strategic, no waiting | Feels disconnected, no urgency |
| **Session-Based** | Easy to implement | No persistence |
| **Hybrid (proposed)** | Best of both worlds | Complex to implement |

---

## Open Questions

- [ ] Exact time ratio? (60x feels right, but needs testing)
- [ ] How to handle very long offline periods? (Months away = years in game?)
- [ ] Should players be able to "pause" their character? (Cryo-sleep?)
- [ ] PvP timing — how to ensure fairness?
- [ ] Economic balance with accelerated time?

---

## My Recommendation

**Start with 60x acceleration + narrative time skips.**

Why:
1. **Fast enough** — Players see meaningful progress each session
2. **Slow enough** — Still feels like living a life, not a montage
3. **Engaging skips** — Travel and waits become story opportunities
4. **Flexible** — Can adjust ratio based on playtesting

The key is making time skips **interesting**, not just fast-forwarding. Every skip is a chance for the AI to generate a mini-story.
