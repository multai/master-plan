# Player System

> *One life. One chance. Make it count.*

---

## Core Philosophy

### Permadeath

**You have ONE life.**

- When your character dies, they're gone. Forever.
- No respawns. No save points. No second chances.
- You must create a new character — different place, different form, different story.

This creates **real stakes**. Every decision matters. Every fight could be your last.

### Persistent World

The universe doesn't pause when you log off.

- Other players continue playing
- AI creatures continue living
- Events keep happening
- **Your character exists in the world while you're offline**

If you log off in a dangerous place, your character can still be killed, robbed, or captured.

---

## Player Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│                    PLAYER LIFECYCLE                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. CREATE ACCOUNT                                            │
│     └── One account per human                                 │
│                                                               │
│  2. CREATE CHARACTER                                          │
│     ├── Choose species (from available catalog)               │
│     ├── Choose starting location (galaxy → planet → city)     │
│     ├── Define backstory and personality                      │
│     ├── Select starting drive                                 │
│     └── Character is born into the world                      │
│                                                               │
│  3. LIVE                                                      │
│     ├── Explore, interact, trade, fight                       │
│     ├── Join plots, factions, relationships                   │
│     ├── Build reputation, wealth, power                       │
│     └── Make a mark on the universe                           │
│                                                               │
│  4. LOG OFF (SAFELY!)                                         │
│     ├── Find a safe location (home, inn, station)             │
│     ├── Character enters "resting" state                      │
│     └── Protected from most harm while offline                │
│                                                               │
│  5. LOG OFF (UNSAFELY)                                        │
│     ├── Character remains in world, vulnerable                │
│     ├── Can be attacked, robbed, captured                     │
│     └── You may return to a dead character                    │
│                                                               │
│  6. DEATH                                                     │
│     ├── Character is permanently dead                         │
│     ├── Legacy effects may persist (children, reputation)     │
│     ├── Account continues — create new character              │
│     └── Start fresh somewhere else in the universe            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Character Creation

When starting fresh, players choose:

### Species
- From the species catalog
- Each has different traits, homeworlds, cultures
- Some are widespread, some are rare
- Species affects how NPCs react to you

### Starting Location
```
Universe
└── Galaxy (choose from dozens+)
    └── Region (core, frontier, uncharted)
        └── Star System
            └── Planet
                └── Location (city, wilderness, station)
```

**Different starting points = wildly different experiences:**
- Start in a peaceful federation core world → safe but regulated
- Start in frontier territory → dangerous but free
- Start in a theocratic empire → religious obligations
- Start in a war zone → immediate danger, opportunity

### Backstory & Drive
- Who were you before the game started?
- What do you want? (drive)
- What do you believe? (values)
- Who do you know? (starting relationships)

---

## Safe Zones

Logging off safely is critical. Safe zones include:

### Owned Property
- Your home, ship cabin, rented room
- Only you can enter (unless you grant access)
- Safest option

### Public Safe Zones
- Inns, stations, sanctuaries
- Protected by local law/security
- NPCs will defend you
- May require payment

### Faction Territories
- If you're a member in good standing
- Faction protects its own
- Lose membership = lose protection

### Unsafe Everywhere Else
- Wilderness, space, hostile territory
- You're on your own
- Log off at your own risk

---

## Offline Behavior

When a player logs off, their character:

### In Safe Zone
```yaml
state: "resting"
vulnerability: "protected"
behavior:
  - Cannot be attacked by players or hostile NPCs
  - Cannot be robbed
  - May still participate in passive events (conversations initiated by others)
  - Time passes but safely
```

### Outside Safe Zone
```yaml
state: "vulnerable"
vulnerability: "full"
behavior:
  - Character controlled by simple AI (survival instincts)
  - Will try to flee if attacked
  - Can be killed, robbed, captured
  - Player returns to whatever state character is in
```

### AI Autopilot (Optional Feature?)
```yaml
# Player can set instructions before logging off
autopilot:
  priority: "survive"
  if_attacked: "flee"
  if_approached: "hide"
  stay_in_area: true
  max_duration: "24 hours"
```

---

## Death & Consequences

### What Happens When You Die

1. **Character is gone** — No recovery, no resurrection (unless plot/magic allows in rare cases)
2. **Inventory lost** — Others can loot your body
3. **Relationships end** — NPCs remember you, but you're gone
4. **Account continues** — You can create a new character immediately

### Legacy System (Optional)

Death isn't entirely without continuation:

- **Children** — If your character had offspring, they exist as NPCs (or future character options?)
- **Reputation echo** — "Aren't you related to the famous [name]?"
- **Inheritance** — Wealth/property can be willed to specific entities
- **Historical record** — Your deeds are recorded in world history

### New Character Restrictions

When creating a new character after death:

- Cannot be in the same location as previous character
- Cannot be same species (optional rule?)
- Cannot have direct contact with previous character's associates immediately
- Fresh start, fresh story

---

## Player vs AI Characters

| Aspect | Player Character | AI Character (NPC) |
|--------|------------------|-------------------|
| **Control** | Human player | LLM-driven |
| **Death** | Permadeath | Can die (affects world) |
| **Offline** | Vulnerable or protected | Always "online" |
| **Creation** | Player choice | System-generated |
| **Motivation** | Player-driven | Drive-driven |

---

## Multiple Characters?

**Question:** Should players be allowed multiple simultaneous characters?

### Option A: One at a time
- More immersion
- True commitment to character
- Simpler system

### Option B: Multiple allowed
- Different experiences
- One dies, others continue
- Risk of meta-gaming

### Option C: Permadeath cooldown
- After death, wait X hours before new character
- Forces reflection
- Prevents death from being trivial

---

## Technical Considerations

### Player Session
```typescript
interface PlayerSession {
  accountId: string;
  
  // Current character (null if dead/none)
  activeCharacter: {
    entityId: string;
    connectedAt: Date;
    lastActivity: Date;
    safeZoneStatus: 'safe' | 'unsafe' | 'unknown';
  } | null;
  
  // Dead characters (history)
  deceasedCharacters: {
    entityId: string;
    name: string;
    species: string;
    livedFrom: Date;
    diedAt: Date;
    causeOfDeath: string;
    finalLocation: string;
  }[];
  
  // Account stats
  totalPlayTime: number;
  charactersCreated: number;
  charactersDied: number;
}
```

### Character State Persistence
```sql
-- When player logs off
UPDATE creatures 
SET 
  status = CASE 
    WHEN is_in_safe_zone(current_location_id) THEN 'resting'
    ELSE 'vulnerable'
  END,
  player_online = false,
  last_player_activity = NOW()
WHERE id = $character_id;

-- On world tick, process vulnerable offline characters
SELECT * FROM creatures 
WHERE status = 'vulnerable' 
AND player_online = false;
-- Run survival AI for each
```

---

## The Experience

**Logging in:**
> "Welcome back. Your character Krix is resting safely at the Waypoint Station on Vega-7. 
> 14 hours have passed. During that time:
> - The trade convoy you invested in arrived safely (+500 credits)
> - A message arrived from Merchant Guild
> - The local news reports increased pirate activity in the sector"

**Logging in after unsafe logout:**
> "Warning: Your character Zeth was in an unsafe location.
> While you were away, your character was attacked by raiders.
> Zeth fled successfully but lost their cargo.
> Current location: Wilderness, 20km from Outpost Gamma.
> Health: Injured. Resources: Minimal."

**Logging in to death:**
> "We're sorry. Your character Mira was killed while you were offline.
> Cause: Ambush by rival faction.
> Location: The Crimson Wastes.
> Mira lived for 47 days and achieved the rank of Journeyman Trader.
> Their story has been recorded.
> 
> [Create New Character]"

---

## Open Questions

- [ ] Resurrection mechanics? (Clone backup technology in advanced civilizations?)
- [ ] Character limit per account?
- [ ] Offline protection duration limit? (Can't stay safe forever?)
- [ ] PvP rules? (Can players kill other players?)
- [ ] Starter protection? (New characters immune for X time?)
