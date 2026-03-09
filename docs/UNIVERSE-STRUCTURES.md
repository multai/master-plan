# Universe Structures

> *The civilizations, governments, and beliefs that shape the galaxy*

---

## Technology Level

Multi-AI is set in a **highly advanced galactic civilization**:

### Space Travel
- **FTL (Faster Than Light)** — Jump drives, warp technology, wormhole gates
- **Sub-light** — Ion engines, solar sails for in-system travel
- **Travel times:** Hours within system, days/weeks between stars

### Weapons & Defense
- **Personal:** Energy pistols, plasma rifles, vibro-blades, neural disruptors
- **Ship-mounted:** Laser batteries, missile systems, railguns
- **Planetary:** Orbital defense platforms, shield generators
- **WMDs:** Planet crackers, bio-weapons (heavily restricted)

### Communication
- **Ansible Network** — Instant communication across star systems
- **Local Comms** — Radio, tight-beam lasers
- **Data Storage** — Vast archives, AI-assisted search

### Medicine & Biology
- **Life extension** — Most species live longer than natural
- **Cybernetics** — Augmentations common in some cultures
- **Cloning** — Possible but regulated
- **Genetic engineering** — Designer organisms, modified species

---

## Governmental Systems

Different regions of the galaxy operate under various systems:

### Empires
```yaml
type: empire
characteristics:
  - Single ruler (Emperor, Empress, Overlord)
  - Hereditary or conquest-based succession
  - Hierarchical bureaucracy
  - Central authority, regional governors
examples:
  - "The Kelthari Dominion"
  - "Zyrathian Star Empire"
```

### Federations
```yaml
type: federation
characteristics:
  - Alliance of autonomous systems
  - Central council with representatives
  - Shared military, independent economies
  - Voluntary membership (usually)
examples:
  - "United Galactic Federation"
  - "The Neutral Territories Compact"
```

### Republics
```yaml
type: republic
characteristics:
  - Elected representatives
  - Constitutional governance
  - Term limits, voting rights
  - Can be democratic or oligarchic
examples:
  - "Terran Republic"
  - "Merchant Republic of Voss"
```

### Theocracies
```yaml
type: theocracy
characteristics:
  - Religious leaders as government
  - Laws based on sacred texts
  - Faith as citizenship requirement
  - Temple bureaucracy
examples:
  - "The Holy Covenant of Azura"
  - "Ancestor Dominion"
```

### Corporatocracies
```yaml
type: corporatocracy
characteristics:
  - Corporations as governing bodies
  - Citizenship = employment
  - Profit-driven policies
  - Corporate security as police
examples:
  - "Omnicorp Territories"
  - "Free Trade Zone 7"
```

### Tribal / Clan Systems
```yaml
type: tribal
characteristics:
  - Blood-based or honor-based governance
  - Chieftains, elders, councils
  - Oral traditions, honor codes
  - Often decentralized
examples:
  - "The Vrolk Clans"
  - "Nomad Fleets"
```

---

## Economic Systems

How different civilizations handle resources and trade:

### Free Market Capitalism
- Private ownership
- Supply/demand pricing
- Minimal regulation
- Wealth inequality common

### State-Controlled Economy
- Government owns production
- Central planning
- Resource allocation
- Can be efficient or corrupt

### Guild Economy
- Professional guilds control industries
- Apprentice → Journeyman → Master progression
- Quality standards, price fixing
- Medieval-in-space feel

### Post-Scarcity
- Automation provides abundance
- Basic needs are free
- Work is optional/creative
- Status through reputation, not wealth

### Barter Networks
- No universal currency
- Direct trade of goods/services
- Reputation-based trust
- Common in frontier areas

### Credit Systems
- Universal digital currency
- Banking conglomerates
- Debt as control mechanism
- Identity-linked accounts

---

## Religious & Philosophical Systems

Belief systems that influence entity behavior:

### The Cosmic Way
```yaml
type: philosophical
beliefs:
  - Universe is conscious
  - All life is connected
  - Death is transformation
  - Balance is sacred
practices:
  - Meditation, contemplation
  - Non-violence preferred
  - Environmental harmony
influence_on_entities:
  drives: [knowledge, harmony]
  values: { compassion: +0.2, tradition: +0.1 }
```

### The Machine Faith
```yaml
type: tech_religion
beliefs:
  - Technology is divine path
  - Augmentation brings enlightenment
  - AI are prophets/angels
  - Flesh is weakness
practices:
  - Cybernetic enhancement
  - Data worship
  - Machine maintenance as ritual
influence_on_entities:
  drives: [knowledge, power]
  skills: { technology: +0.3 }
```

### Ancestor Veneration
```yaml
type: ancestor_worship
beliefs:
  - Dead watch over living
  - Ancestors guide decisions
  - Honor family above all
  - Disgrace affects afterlife
practices:
  - Shrines, offerings
  - Consulting the dead (real or symbolic)
  - Genealogy as sacred knowledge
influence_on_entities:
  drives: [family, tradition]
  values: { loyalty: +0.3, tradition: +0.3 }
```

### The Void Cult
```yaml
type: nihilistic
beliefs:
  - Universe is meaningless
  - Only power matters
  - Death is the end
  - Take what you can
practices:
  - No rituals
  - Self-interest above all
  - Might makes right
influence_on_entities:
  drives: [power, survival]
  values: { compassion: -0.2, ambition: +0.3 }
```

### Elemental Pantheon
```yaml
type: polytheistic
beliefs:
  - Multiple gods govern aspects of existence
  - Fire, Water, Earth, Air, Void
  - Gods can be appeased or angered
  - Fate is malleable through prayer
practices:
  - Temple worship
  - Sacrifices (symbolic in modern times)
  - Festivals, holy days
  - Priestly class
influence_on_entities:
  drives: [varies by patron deity]
  values: { tradition: +0.2 }
```

---

## Social Classes

How entities fit into society:

### Elite
- Rulers, nobles, mega-wealthy
- Access to everything
- Political power
- Responsibility (in good societies) or exploitation (in bad ones)

### Professional Class
- Skilled workers, managers, officers
- Comfortable living
- Some political voice
- Education, expertise

### Working Class
- Manual labor, service workers
- Basic needs met (usually)
- Limited upward mobility
- Backbone of economy

### Outcast
- Criminals, refugees, exiles
- Outside normal society
- Survival-focused
- May form own communities

### Slave/Indentured
- No freedom (in some civilizations)
- Property of others
- Varies from brutal to "benevolent"
- Abolition movements exist

---

## Database Schema Additions

```sql
-- Government types
CREATE TABLE government_types (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  characteristics JSONB,
  typical_laws JSONB
);

-- Religions
CREATE TABLE religions (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  type TEXT, -- monotheistic, polytheistic, philosophical, etc.
  beliefs JSONB,
  practices JSONB,
  influence_modifiers JSONB -- how it affects entity attributes
);

-- Entity affiliations (updated)
CREATE TABLE entity_affiliations (
  entity_id UUID REFERENCES creatures(id),
  affiliation_type TEXT, -- government, religion, guild, faction
  affiliation_id UUID,
  rank TEXT,
  joined_at TIMESTAMPTZ,
  loyalty FLOAT DEFAULT 0.5,
  PRIMARY KEY (entity_id, affiliation_type, affiliation_id)
);
```

---

## How This Affects Entities

When an entity is created, their background includes:

1. **Homeworld government** — Shapes their understanding of authority
2. **Economic background** — Affects relationship with wealth
3. **Religious upbringing** — Influences values and worldview
4. **Social class** — Determines starting resources and connections

These factors feed into the entity's:
- System prompt (personality)
- Default values
- Available affiliations
- Plot compatibility
