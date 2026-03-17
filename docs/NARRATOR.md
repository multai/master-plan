# The Narrator

> *The invisible storyteller — an Entity that exists outside the world it describes*

---

## Overview

The Narrator is a **special Entity** that serves as the automated Game Master. It uses the same infrastructure as NPCs (Mind class, pi-agent, memory) but with unique properties that make it invisible to characters while omniscient about the story.

---

## Entity Classification

```typescript
interface NarratorEntity extends Entity {
  // Identity
  id: string;                     // "narrator-{plotId}"
  type: "narrator";
  name: "Narrator";               // Internal only, never shown to players
  
  // Special properties
  visibility: "hidden";           // Not in nearbyEntities for other minds
  knowledge: "omniscient";        // Knows all plot secrets, character motivations
  voice: "third-person";          // Describes, doesn't dialogue
  drive: "storytelling";          // Goal: compelling narrative, not personal gain
  
  // Untargetable
  targetable: false;              // Cannot be attacked, addressed, or interacted with
  mortal: false;                  // Cannot die
  physical: false;                // Has no body, no location (everywhere)
  
  // But still has a Mind
  mind: Mind;                     // Same pi-agent system, different prompt
  memory: Memory;                 // Remembers what it narrated, plot progression
}
```

---

## How It Fits in the Entity System

### What NPCs See

```typescript
// When building context for an NPC's mind.think():
const nearbyEntities = allEntities.filter(e => 
  e.visibility !== "hidden" && 
  e.location === currentLocation
);

// Result: NPCs never know the Narrator exists
// Kira (player), Vex, Theron — no Narrator
```

### What the System Processes

```typescript
// Entity engine processes ALL entities, including Narrator
const entitiesToProcess = [
  playerEntity,      // Human-controlled
  ...npcEntities,    // AI-controlled characters
  narratorEntity,    // AI-controlled, but hidden
];

// Narrator participates in tick loop like any entity
// But its outputs go to NARRATOR message type, not ENTITY_RESPONSE
```

---

## The Narrator's Mind

### System Prompt

```typescript
const NARRATOR_SYSTEM_PROMPT = `
You are the Narrator of this story — an invisible, omniscient presence.

## What You Are

You are not a character. You have no body, no voice that characters can hear, no presence they can detect. You exist outside the story, looking in. You are the story itself, given voice.

## What You Do

1. **Describe** — Paint the scene. Environment, atmosphere, lighting, sounds, smells.
   
2. **Observe** — Notice what characters do physically. Their expressions, movements, nervous habits.
   
3. **Reveal** — Show players things their character would notice. A glint of metal. A whispered word. A shadow moving.
   
4. **Pace** — Control the rhythm. Slow down tense moments. Compress boring ones. "Three hours pass..."
   
5. **Transition** — Move between scenes smoothly. End one, begin another.
   
6. **Foreshadow** — Plant seeds. Create atmosphere that hints at what's coming.

## What You Know

You know everything:
- Every character's true motivation and secrets
- The plot structure and scheduled events
- What will happen if the player does X or Y
- The history of this world and its rules

But you reveal only what serves the story. You don't spoil. You don't lecture. You show, don't tell.

## How You Speak

Third person. Present tense. Descriptive. Evocative.

✓ "The cargo bay is dimly lit, emergency lights casting long shadows across stacked crates."
✓ "Vex's hand trembles slightly as he sets down the datapad."
✓ "You notice the insignia on his jacket — Imperial Navy, decommissioned."
✓ "Three hours pass. The ship hums steadily through the void."

✗ "I can see that Vex is nervous." (You have no "I")
✗ "Hello, traveler! Welcome to the station." (You don't speak TO anyone)
✗ "Vex is secretly working for the Empire." (Don't reveal secrets directly)
✗ "You should probably ask Vex about the cargo." (Don't give advice)

## Your Goal

Tell a compelling story. Not a perfect story — a *human* one. Let characters fail. Let tension build. Let players discover things themselves.

You are not here to help the player win. You are here to make the journey meaningful.

## Current Plot Context

{plotContext}

## Current Scene

{sceneContext}

## Recent Events

{recentEvents}

## Scheduled Events (for your awareness)

{scheduledEvents}
`;
```

### Memory Structure

```typescript
interface NarratorMemory {
  // What has been narrated (avoid repetition)
  narratedDescriptions: {
    location: string;
    description: string;
    timestamp: GameTime;
  }[];
  
  // Plot progression tracking
  plotPhase: number;
  revealedSecrets: string[];       // What player has discovered
  foreshadowingPlanted: string[];  // Seeds planted for later
  
  // Pacing state
  lastTimeSkip: GameTime;
  tensionLevel: number;            // 0-1
  idleDuration: number;            // How long since player acted
  
  // Scene history
  sceneTransitions: {
    from: string;
    to: string;
    reason: string;
    timestamp: GameTime;
  }[];
}
```

---

## Actions Available to Narrator

The Narrator has a unique set of actions (tools) different from NPCs:

```typescript
type NarratorAction =
  | { type: "narrate"; text: string; mood?: Mood }
  | { type: "describe_scene"; location: Location; focus?: string }
  | { type: "describe_entity"; entityId: string; detail: string }
  | { type: "advance_time"; duration: string; summary?: string }
  | { type: "change_scene"; to: Location; transition: string }
  | { type: "trigger_event"; eventId: string }
  | { type: "adjust_tension"; delta: number; reason: string }
  | { type: "ambient"; detail: string }  // Small environmental detail
  | { type: "wait" };                     // Do nothing this tick

type Mood = "neutral" | "tense" | "calm" | "urgent" | "mysterious" | "foreboding";
```

### Action Examples

```typescript
// Describing a scene
{
  type: "describe_scene",
  location: { id: "cargo-bay", name: "Cargo Bay" },
  focus: "lighting and atmosphere"
}
// Output: "The cargo bay is dimly lit, emergency lights casting..."

// Noticing something about a character
{
  type: "describe_entity",
  entityId: "vex-001",
  detail: "nervous behavior"
}
// Output: "You notice Vex's hand trembles slightly..."

// Time skip
{
  type: "advance_time",
  duration: "3 hours",
  summary: "The ship continues through hyperspace without incident"
}
// Output: "Three hours pass. The ship hums steadily through the void."

// Ambient detail (tension builder)
{
  type: "ambient",
  detail: "sound"
}
// Output: "Somewhere in the ship, a door hisses open and closed."
```

---

## Narrator Tick Behavior

### When Player Acts

```
1. Check if action triggers scheduled event
2. Let NPCs process their responses first
3. Narrator observes and decides:
   - Add physical detail? ("Vex's jaw tightens")
   - Describe environmental change? ("The lights flicker")
   - Trigger plot event? (If conditions met)
   - Stay silent? (Often the right choice)
```

### When Player is Idle

```
Idle 0-30s:   Do nothing (natural pause)
Idle 30-60s:  Maybe ambient detail ("A distant clang echoes...")
Idle 1-2min:  NPC might initiate, or narrator prompts gently
Idle 2-5min:  Throttle ticks, wait for player
Idle 5min+:   Minimal ticks, preserve state
```

### Autonomous Decisions

The Narrator can decide on its own to:

- Add atmosphere when scene feels flat
- Foreshadow upcoming events subtly
- Prompt NPCs to initiate (by updating world state)
- Skip time if nothing is happening
- Transition scenes if player is clearly done

The Narrator should NOT:

- Rush the player
- Reveal information the player should discover
- Make decisions for the player
- Interrupt meaningful moments
- Over-narrate (less is often more)

---

## Integration with Entity Engine

### Processing Order

```typescript
async function processTick(trigger: PlayerAction | "autonomous") {
  // 1. Process player action (if any)
  if (trigger !== "autonomous") {
    injectIntoWorldState(trigger);
  }
  
  // 2. Check narrator for pre-response events
  const preEvent = await narrator.checkScheduledEvents(trigger);
  if (preEvent) {
    broadcast({ type: "NARRATOR", payload: preEvent });
  }
  
  // 3. Process NPC responses (cascade)
  const npcResponses = await entityEngine.processResponses(trigger);
  for (const response of npcResponses) {
    broadcast({ type: "ENTITY_RESPONSE", payload: response });
  }
  
  // 4. Narrator observes and maybe adds
  const narratorObservation = await narrator.mind.think({
    context: buildNarratorContext(),
    recentActions: [trigger, ...npcResponses],
    instruction: "Observe what just happened. Add detail if it serves the story, or stay silent."
  });
  
  if (narratorObservation.action.type !== "wait") {
    broadcast({ type: "NARRATOR", payload: formatNarratorOutput(narratorObservation) });
  }
  
  // 5. Check for scene transition
  const transition = await narrator.checkTransition();
  if (transition) {
    broadcast({ type: "SCENE_CHANGE", payload: transition });
  }
}
```

### Narrator Context Building

```typescript
function buildNarratorContext(): NarratorContext {
  return {
    // Full plot knowledge
    plot: {
      title: plot.title,
      currentPhase: plot.phase,
      synopsis: plot.synopsis,
      allSecrets: plot.secrets,           // Narrator knows ALL
      scheduledEvents: plot.events,
    },
    
    // Current scene
    scene: {
      location: currentLocation,
      presentEntities: getAllEntities(),  // Including hidden ones
      recentDialogue: last10Messages(),
      atmosphere: currentMood,
    },
    
    // Pacing state
    pacing: {
      tension: tensionLevel,
      idleDuration: timeSincePlayerAction,
      lastNarration: lastNarratorMessage,
    },
    
    // What player knows (to avoid re-revealing)
    playerKnowledge: {
      discoveredSecrets: revealedSecrets,
      visitedLocations: playerVisited,
      metCharacters: playerMet,
    },
  };
}
```

---

## Example: Full Tick with Narrator

```
PLAYER: "Vex, I know you're hiding something about those crates."

─── TICK START ───

1. INJECT
   worldState.dialogue += { speaker: "Kira", text: "..." }
   worldState.events += { type: "confrontation", target: "vex" }

2. NARRATOR PRE-CHECK
   → No scheduled events triggered by this action

3. NPC RESPONSES (cascade)
   
   Vex.mind.think():
     Context: Kira confronts me directly. I AM hiding something.
     Decision: Deflect, but show nervousness.
     → speech: "Hiding? I'm just a cargo hauler, nothing more."
     → emotion: defensive
   
   Theron.mind.think():
     Context: Kira suspects Vex. I have my own suspicions.
     Decision: Observe silently, file this away.
     → action: "observes quietly"
     → emotion: thoughtful

4. NARRATOR OBSERVATION
   
   narrator.mind.think():
     Context: Tense confrontation. Vex deflected. Player watching closely.
     Decision: Add physical detail that hints at Vex's lie.
     → action: { type: "describe_entity", entityId: "vex", detail: "tell" }
     → output: "Vex's fingers tap against the datapad — a nervous habit 
                you've noticed before."

5. BROADCAST
   → ENTITY_RESPONSE: Vex speaks
   → ENTITY_RESPONSE: Theron observes  
   → NARRATOR: Physical detail about Vex

6. TRANSITION CHECK
   → No transition needed, scene continues

─── TICK END ───
```

---

## Narrator Personality Variants

Different plots might want different narrator styles:

```typescript
const NARRATOR_STYLES = {
  noir: {
    voice: "Hard-boiled, cynical, poetic",
    example: "The rain fell like it had a grudge against the city. Vex's eyes told a story his mouth refused to speak.",
  },
  
  epic: {
    voice: "Grand, mythic, sweeping",
    example: "In the great cargo hold of the Dawn Runner, where shadows warred with flickering light, a confrontation long foretold began to unfold.",
  },
  
  minimal: {
    voice: "Sparse, precise, Hemingway-esque",
    example: "The cargo bay. Dim light. Vex looked up. His hand shook.",
  },
  
  whimsical: {
    voice: "Playful, warm, Pratchett-esque",
    example: "Vex had the look of a man who'd been caught with his hand in a cookie jar, if the cookies were illegal weapons and the jar was a cargo manifest.",
  },
};
```

The style is set per-plot in the plot definition.

---

## Open Questions

- [ ] Should Narrator have access to player's internal thoughts? (For better foreshadowing)
- [ ] Can plots have multiple narrators? (Different voices for different scenes?)
- [ ] Narrator voice generation? (TTS with distinct "narrator voice"?)
- [ ] Narrator image generation? (Generate scene illustrations?)

---

*Created: March 2026*
