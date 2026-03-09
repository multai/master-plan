# Plots & Storylines System

> *The narratives that weave the universe together*

**Repository candidate:** `plots`, `stories`, `narratives`, `quests`

---

## What Are Plots?

Plots are **events and storylines** that happen across the universe, drawing creatures together around common interests, conflicts, or goals.

### Scale Spectrum

| Scale | Example | Affected Entities |
|-------|---------|-------------------|
| **Galactic** | War between empires | Millions |
| **Planetary** | Civil war, revolution | Thousands |
| **Regional** | Crime syndicate takeover | Hundreds |
| **Local** | Village dispute, merchant guild conflict | Dozens |
| **Personal** | Individual quest, bureaucratic task | 1-5 |

---

## Plot Types

### Political & Military
- **War** — Two factions fighting for territory/ideology
- **Revolution** — Popular uprising against ruling power
- **Coup** — Power grab within government
- **Invasion** — External force attacking
- **Cold War** — Tension without direct conflict

### Economic
- **Trade War** — Economic sanctions, smuggling routes
- **Resource Scarcity** — Competition for rare materials
- **Market Crash** — Economic collapse, opportunity for some
- **Monopoly Rise** — One entity controlling an industry

### Social & Religious
- **Religious Movement** — New faith spreading, old faith declining
- **Cultural Clash** — Different species/cultures conflicting
- **Social Reform** — Movement for rights, equality
- **Persecution** — A group being hunted or oppressed

### Criminal & Underground
- **Crime Syndicate** — Organized crime expanding
- **Bounty Hunt** — High-value target on the run
- **Smuggling Route** — Illegal goods pipeline
- **Heist** — Major theft being planned

### Personal & Individual
- **Bureaucratic Quest** — Get a permit, clear a debt, find a document
- **Family Drama** — Inheritance dispute, missing relative
- **Revenge** — Personal vendetta
- **Romance** — Love story across boundaries
- **Discovery** — Finding something ancient/valuable

---

## Plot Structure

```yaml
plot:
  id: "keltar-revolution-2847"
  name: "The Keltar Uprising"
  type: "revolution"
  scale: "planetary"
  
  # Where and when
  location: 
    planet: "keltar-prime"
    regions: ["northern-territories", "capital-city"]
  started_at: "cycle-2847"
  estimated_duration: "50-100 cycles"
  
  # The story
  summary: |
    The working class of Keltar Prime rises against the 
    merchant oligarchy after decades of exploitation.
    
  factions:
    - id: "revolutionaries"
      name: "The People's Dawn"
      goals: ["overthrow oligarchy", "redistribute wealth"]
      
    - id: "oligarchy"
      name: "Merchant Council"
      goals: ["maintain power", "crush rebellion"]
      
    - id: "neutral"
      name: "Rural Communities"
      goals: ["survive", "stay out of conflict"]
  
  # How entities can participate
  roles:
    - role: "revolutionary_soldier"
      faction: "revolutionaries"
      requirements: { drive: ["freedom", "justice"], homeworld: "keltar-prime" }
      
    - role: "oligarch_enforcer"
      faction: "oligarchy"
      requirements: { drive: ["wealth", "power"] }
      
    - role: "smuggler"
      faction: "neutral"
      requirements: { drive: ["wealth", "survival"] }
      description: "Profit from both sides by running supplies"
      
    - role: "journalist"
      faction: "neutral"
      requirements: { drive: ["knowledge", "truth"] }
      description: "Document the conflict"
  
  # Plot progression
  phases:
    - phase: 1
      name: "Underground Organizing"
      events: ["secret meetings", "propaganda spreading"]
      
    - phase: 2
      name: "Open Revolt"
      events: ["first battle", "capital siege"]
      
    - phase: 3
      name: "Resolution"
      outcomes: ["revolutionary victory", "oligarchy wins", "stalemate"]
  
  # World state changes
  consequences:
    on_revolutionary_victory:
      - { type: "government_change", planet: "keltar-prime" }
      - { type: "economy_shift", effect: "redistribution" }
    on_oligarchy_wins:
      - { type: "crackdown", targets: "revolutionaries" }
      - { type: "reputation_change", faction: "oligarchy", delta: -20 }
```

---

## Universe Context

### Technology Level

The Multi-AI universe features **highly developed galactic civilizations**:

- **Faster-than-light travel** — Ships crossing systems in days/weeks
- **Advanced weapons** — Energy weapons, drones, orbital strikes
- **Communication networks** — Instant messaging across star systems
- **AI & automation** — Robots, automated systems (but entities are organic beings with AI minds, not robots)

### Social Structures

Entities organize themselves in various ways:

#### Governmental Systems
- **Empire** — Single ruler, hierarchical
- **Federation** — Alliance of autonomous states
- **Republic** — Elected representatives
- **Theocracy** — Religious leadership
- **Corporatocracy** — Corporations as government
- **Anarchy** — No central authority
- **Hive Mind** — Collective consciousness (some species)

#### Economic Systems
- **Capitalism** — Free market, private ownership
- **Socialism** — Collective ownership, redistribution
- **Feudalism** — Lords and vassals
- **Barter** — Direct trade, no currency
- **Post-scarcity** — Abundance, no economic need

#### Religious Structures
- **Polytheism** — Multiple gods
- **Monotheism** — Single deity
- **Ancestor Worship** — Veneration of the dead
- **Tech Worship** — Technology as divine
- **Atheism** — No supernatural beliefs
- **Cosmic Philosophy** — Universe itself as sacred

---

## Entity Attributes for Plot Integration

Entities need attributes that let them fit into plots:

```typescript
interface EntityPlotAttributes {
  // What the entity cares about
  drive: DriveType; // family, wealth, freedom, power, knowledge, survival
  
  // Deeper values
  values: {
    justice: number;      // 0-1
    loyalty: number;      // 0-1
    ambition: number;     // 0-1
    compassion: number;   // 0-1
    tradition: number;    // 0-1
    independence: number; // 0-1
  };
  
  // Social position
  socialClass: 'elite' | 'middle' | 'working' | 'outcast';
  
  // Affiliations
  affiliations: {
    government?: string;      // Which government they serve/support
    religion?: string;        // Religious affiliation
    guild?: string;          // Professional organization
    faction?: string;        // Political faction
    criminalOrg?: string;    // Underworld connection
  };
  
  // Skills relevant to plots
  skills: {
    combat: number;
    diplomacy: number;
    stealth: number;
    technology: number;
    leadership: number;
    survival: number;
  };
  
  // Current plot involvement
  activePlots: {
    plotId: string;
    role: string;
    faction: string;
    commitment: number; // How dedicated (0-1)
  }[];
}
```

---

## Plot Engine

The system that matches entities to plots and progresses storylines:

```
┌─────────────────────────────────────────────────────────────┐
│                      PLOT ENGINE                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. PLOT MATCHING                                            │
│     - Scan entities for plot-compatible attributes           │
│     - Recruit entities into appropriate roles                │
│     - Some entities actively seek plots (ambitious)          │
│     - Some get pulled in reluctantly (location-based)        │
│                                                              │
│  2. PLOT PROGRESSION                                         │
│     - Each world tick, advance active plots                  │
│     - Check phase transitions                                │
│     - Generate events based on entity actions                │
│     - Determine outcomes based on faction strength           │
│                                                              │
│  3. CONSEQUENCE RESOLUTION                                   │
│     - Apply world state changes                              │
│     - Update entity statuses                                 │
│     - Spawn follow-up plots                                  │
│     - Record in world history                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Repositories Updated

| Repository | Addition |
|------------|----------|
| `entity` / `creature` | Add plot attributes (values, affiliations, skills) |
| `plots` | Plot definitions, engine, matching system |
| `worlds` | Government types, religions, economic systems |
| `database` | Plot tables, faction tables, entity-plot relationships |

---

## Example: Personal Bureaucratic Quest

Not all plots are epic. Some are mundane but still drive behavior:

```yaml
plot:
  id: "cargo-permit-7789"
  name: "Getting a Cargo Permit"
  type: "bureaucratic"
  scale: "personal"
  
  summary: |
    Kreth needs a cargo hauling permit to legally transport 
    goods through Federated space. The bureaucracy is slow.
    
  participant:
    entity_id: "kreth-9182"
    role: "applicant"
    
  steps:
    - step: 1
      action: "Submit application at Sector Office"
      status: "completed"
      
    - step: 2
      action: "Wait for background check"
      status: "in_progress"
      duration: "5-10 cycles"
      
    - step: 3
      action: "Interview with licensing officer"
      status: "pending"
      
    - step: 4
      action: "Pay fees and receive permit"
      status: "pending"
  
  complications:
    - "Background check reveals old smuggling charge"
    - "Licensing officer demands bribe"
    - "Another applicant tries to steal the slot"
```

---

## Open Questions

- [ ] How do plots spawn? (Random? Based on world conditions? Designer-created?)
- [ ] Can entities start their own plots? (Entity decides to start a revolution?)
- [ ] How do plots interact? (War makes smuggling more profitable)
- [ ] How do players interact with plots? (Join, influence, create?)
