# Cosmic Scale

> *Not a galaxy. A universe.*

---

## The True Scale

Multi-AI isn't set in "a galaxy" — it's set in **the universe**.

Like our real universe:
- Billions of galaxies
- Each galaxy with billions of stars
- Countless worlds, civilizations, species
- We don't know what's out there

### What This Means

Different galaxies can have:
- **Wildly different technology levels** — Stone age to post-singularity
- **Completely alien civilizations** — No common reference points
- **Unknown contact status** — Some galaxies have never met
- **Varying physics?** — Optional: Some regions have different rules

---

## Galaxy Classifications

### By Technology Level

| Level | Name | Characteristics |
|-------|------|-----------------|
| **0** | Pre-Industrial | No machines, agrarian or hunter-gatherer |
| **1** | Industrial | Combustion, electricity, early computing |
| **2** | Atomic | Nuclear power, early space travel |
| **3** | Digital | AI, global networks, local space colonization |
| **4** | Stellar | FTL travel, solar system mastery |
| **5** | Galactic | Galaxy-spanning civilization |
| **6** | Intergalactic | Travel between galaxies |
| **7** | Cosmic | Universe-scale influence, near-godlike |

Most gameplay happens at Levels 3-6.

### By Contact Status

```yaml
contact_status:
  isolated: 
    description: "No known contact with other galaxies"
    percentage: "60%"
    
  contacted:
    description: "Aware of other galaxies, limited interaction"
    percentage: "25%"
    
  integrated:
    description: "Active trade/diplomacy with neighbors"
    percentage: "10%"
    
  dominant:
    description: "Controls or influences multiple galaxies"
    percentage: "5%"
```

### By Habitability

- **Thriving** — Abundant life, many civilizations
- **Sparse** — Few habitable worlds, isolated pockets
- **Dead** — No known life (yet?)
- **Hostile** — Life exists but conditions are extreme
- **Artificial** — Civilizations have engineered habitability

---

## Scale Hierarchy

```
UNIVERSE
│
├── GALACTIC CLUSTER (group of galaxies)
│   │
│   ├── GALAXY (e.g., "The Spiral Dominion")
│   │   │
│   │   ├── GALACTIC REGION (core, arm, halo)
│   │   │   │
│   │   │   ├── SECTOR (administrative division)
│   │   │   │   │
│   │   │   │   ├── STAR SYSTEM (sun + planets)
│   │   │   │   │   │
│   │   │   │   │   ├── PLANET (world)
│   │   │   │   │   │   │
│   │   │   │   │   │   ├── CONTINENT/REGION
│   │   │   │   │   │   │   │
│   │   │   │   │   │   │   ├── NATION/TERRITORY
│   │   │   │   │   │   │   │   │
│   │   │   │   │   │   │   │   ├── CITY/SETTLEMENT
│   │   │   │   │   │   │   │   │   │
│   │   │   │   │   │   │   │   │   └── LOCATION (building, area)
```

---

## Example Galaxies

### The Spiral Dominion (Level 5-6)
```yaml
name: "The Spiral Dominion"
type: spiral
technology_level: 5-6
dominant_species: "Kelthari"
government: "Imperial Hegemony"
contact_status: "integrated"
notable_features:
  - "Ancient gateway network connecting all arms"
  - "Central black hole is worshipped"
  - "Unified currency: Spiral Credits"
population: "~10 trillion sentient beings"
player_start_available: true
```

### The Scattered Reach (Level 3-4)
```yaml
name: "The Scattered Reach"
type: irregular
technology_level: 3-4
dominant_species: "Multiple (fragmented)"
government: "None (many small nations)"
contact_status: "isolated"
notable_features:
  - "No FTL — civilizations developed independently"
  - "First contact events happening"
  - "Resource-rich but dangerous"
population: "Unknown (~100 billion?)"
player_start_available: true
danger_level: high
```

### The Quiet Expanse (Level 0-2)
```yaml
name: "The Quiet Expanse"
type: dwarf
technology_level: 0-2
dominant_species: "None (early life)"
government: "Tribal/Feudal"
contact_status: "isolated"
notable_features:
  - "No advanced civilizations"
  - "Rich in primitive cultures"
  - "Untouched resources"
population: "~10 billion (primitive)"
player_start_available: true
note: "Starting here means starting from scratch"
```

### The Machine Collective (Level 6-7)
```yaml
name: "The Machine Collective"
type: elliptical
technology_level: 6-7
dominant_species: "None (AI only)"
government: "Distributed Intelligence"
contact_status: "dominant"
notable_features:
  - "All organic life long gone"
  - "Entire galaxy is computational substrate"
  - "Rarely interacts with 'meatspace'"
population: "N/A (1 distributed mind)"
player_start_available: false
note: "Exists as environmental factor, not playable"
```

---

## Travel Between Galaxies

### Methods

| Method | Speed | Availability |
|--------|-------|--------------|
| **Generation Ship** | Sub-light, centuries | Level 3+ |
| **Cryo-Sleep Voyage** | Sub-light, decades | Level 4+ |
| **Wormhole Gates** | Instant, fixed points | Level 5+ (rare) |
| **Fold Drive** | Days/weeks | Level 6+ |
| **Unknown** | ??? | Level 7 |

### Practical Implications

- Intergalactic travel is **rare and expensive**
- Most players will stay within one galaxy
- Galaxies feel like separate "servers" but share the same universe
- High-level play involves galaxy-hopping

---

## Database Schema

```sql
-- Galactic clusters
CREATE TABLE galactic_clusters (
  id UUID PRIMARY KEY,
  name TEXT,
  position_x FLOAT,
  position_y FLOAT,
  position_z FLOAT,
  description TEXT
);

-- Galaxies
CREATE TABLE galaxies (
  id UUID PRIMARY KEY,
  cluster_id UUID REFERENCES galactic_clusters(id),
  name TEXT NOT NULL,
  type TEXT, -- spiral, elliptical, irregular, dwarf
  technology_level_min INT,
  technology_level_max INT,
  contact_status TEXT,
  dominant_species_id UUID,
  description TEXT,
  player_start_available BOOLEAN DEFAULT true,
  danger_level TEXT,
  population_estimate TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Sectors (within galaxies)
CREATE TABLE sectors (
  id UUID PRIMARY KEY,
  galaxy_id UUID REFERENCES galaxies(id),
  name TEXT,
  region TEXT, -- core, arm, halo, fringe
  controlling_faction_id UUID,
  stability TEXT, -- stable, contested, lawless
  description TEXT
);

-- Star systems (within sectors) - existing table enhanced
ALTER TABLE star_systems ADD COLUMN sector_id UUID REFERENCES sectors(id);
ALTER TABLE star_systems ADD COLUMN technology_level INT;
ALTER TABLE star_systems ADD COLUMN discovery_status TEXT; -- charted, explored, unknown
```

---

## Gameplay Implications

### Starting Galaxy Choice

When creating a character, players choose:
1. **Galaxy** — Determines overall tech level, culture, danger
2. **Region** — Core (safe, developed) vs Frontier (dangerous, free)
3. **Specific location** — Planet, city, role

### Different Galaxies = Different Experiences

- **High-tech galaxy:** Politics, intrigue, established factions
- **Low-tech galaxy:** Exploration, survival, discovery
- **War-torn galaxy:** Combat, factions, constant danger
- **Frontier galaxy:** Freedom, lawlessness, opportunity

### Cross-Galaxy Play

- Rare but possible
- Characters who travel between galaxies are legendary
- Creates interesting culture clash stories
- Could be late-game content

---

## Open Questions

- [ ] How many galaxies at launch? (Start small, expand?)
- [ ] Player distribution system? (Prevent overcrowding)
- [ ] Time dilation between galaxies? (Same timeline or different?)
- [ ] Can events in one galaxy affect another?
