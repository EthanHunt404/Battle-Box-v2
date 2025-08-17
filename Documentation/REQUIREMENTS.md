# BattleBox v2 – Turn‑Based Battle Engine Library

> **Version:** 0.4 · **Author:** Ethan Buttazzi · **Updated:** 2025‑08‑02

---

## Contents
1. Why BattleBox  
2. Goals  
3. Scope (v2)  
4. Data Model & Rules  
5. Core Features  
6. Quality Bar  
7. Architecture  
8. Logging  
9. Assumptions  
10. Completion Criteria  
11. Risks & Mitigations  
12. Roadmap  
13. Future Ideas (v3+)  
14. Glossary

---

## 1 · Why BattleBox

Open‑source **C# library** that simulates deterministic, headless, turn‑based **encounters**. Adapters (CLI, web, batch) consume the library; all combat content is JSON‑driven.

**Key Terms:** Actor (player‑controlled combatant), Enemy (specialized Actor with a Brain strategy), Brain (fixed AI decision component), Status Effect (DOT/State/CC), Stage (future battlefield context), JSONL (log format), Deterministic (same seed ⇒ same outcome), Encounter (full match), Round (complete rotation of turns), Turn (single combatant action)

---

## 2 · Goals

| ID | Objective | KPI |
|----|-----------|-----|
| G‑01 | Deliver the library + two thin adapters in 3 months | 2 adapters, zero engine edits |
| G‑02 | Hot‑load units / moves / brains via JSON | No recompilation |
| G‑03 | Maintain low latency in bulk simulations | p95 ≤ 50 ms @ 100 **encounters** |

*Version boundary: v2 uses fixed rules—no dynamic modifiers; richer behavior reserved for v3.*

---

## 3 · Scope (v2)

### **Included**
- Turn scheduler & action pipeline with fixed‑rule AI brains (Aggressive, Defensive, Support, Wild, Tactical).
- JSON‑defined content: actors, enemies, moves, brains, global config.
- Status effects (**DOT**, **State**, **CC**) with refresh‑on‑reapply (no stacking).
- Targeting modes: **Fixed N‑M**, **Zone**, **AOE**.
- Real‑time event stream and per‑round **JSONL** logging.
- Default persistence to `%APPDATA%/BattleBox` (overrideable).

### **Excluded**
- Conditional/passive abilities and dynamic rule modifiers.
- Networked multiplayer or live services.
- Rich graphical UI (only thin adapters).

---

## 4 · Data Model & Rules
- **Global Config** — designates which stats map to *health* and *initiative*.
- **Actor** — stats · moveset · mitigation; starts at level 1; earns XP.
- **Enemy** — Actor + Brain; level set at initialization (not persisted); grants configured XP on defeat.
- **Move** — declares targeting mode; lists cost, cooldown, damage types; specifies attribute used for power.
- **Status Effect** — category = DOT | State | CC; re‑apply refreshes duration.
- **Damage ⟂ Mitigation** — flat / percent resistances; fixed order; no conditional modifiers.
- **Progression** — actors gain XP, the experience curve follows one of these patterns (Linear · Quadratic · Exponential ), level‑up after an encounter, apply **stat‑bias vector** (e.g., [3,2,1,2]), and unlock moves via `"moveId:level"`,

---

## 5 · Core Features
| ID | Priority | Description |
|----|:------:|-------------|
| **E‑01** | **M** | Initiative‑based turn cycle with tie‑breakers |
| **E‑02** | **M** | Validate → execute → log every action |
| **E‑03** | **M** | JSON‑driven content (actors, enemies, moves, brains) |
| **E‑05** | **M** | Comprehensive log: live stream + JSONL per round |
| **E‑08** | **M** | Mandatory targeting per move (Fixed N‑M · Zone · AOE) |
| **E‑09** | **M** | Deterministic ordered mitigation |
| **E‑07** | **M** | Extensible stats & mitigation profiles |
| **E‑10** | **M** | Built‑in AI brains (Aggressive · Defensive · Support · Wild · Tactical) *(no emergent behavior in v2)* |
| **E‑04** | **S** | Status effects refresh on re‑apply (no stacking) |
| **E‑11** | **M** | Actor progression: XP, growth curves, stat‑bias gains, move unlocks; enemies have runtime‑only levels |

---

## 6 · Quality Bar
| Metric | Target |
|--------|--------|
| Performance | p95 turn ≤ 50 ms @ 100 encounters |
| Determinism | 100 % with seed |
| Logging | ≥ 99 % of state events captured |

---

## 7 · Architecture
- **Core Library** – encapsulates scheduling, resolution, mitigation, status, and fixed AI.
- **Content (JSON)** – actors, enemies, moves, brains, global settings.
- **Adapters** – thin CLI / web clients; no game logic outside library.
- **Persistence** – `%APPDATA%/BattleBox` by default (override via env var or library setting).

---

## 8 · Logging
- **Encounter lifecycle:** log when an encounter starts and ends.
- **Real‑time stream:** adapters subscribe to live events.
- **Round file copy:** events appended to timestamped JSONL after each round (non‑blocking).
- **Events captured:** EncounterStart · TurnStart · Attack · Damage · StatusApplied · DotTick · StatusExpired · TurnEnd · RoundComplete · EncounterEnd · XPGranted · ActorLeveledUp.
- **Deterministic replay** reproduces the same encounter with identical content files.

---

## 9 · Assumptions
1. Library embedded in host process; out‑of‑proc adapters optional.  
2. Config set modest for MVP.  
3. Single‑instance runtime; scaling later.  
4. Content version hash recorded to ensure replay consistency.

---

## 10 · Completion Criteria
- All **Must** features implemented & unit‑tested.
- Two reference adapters run unmodified.
- Log replay identical with same seed and content hash.
- Performance SLA met in benchmark harness.

---

## 11 · Risks & Mitigations
| Risk | L | I | Plan |
|------|:-:|:-:|------|
| Rule‑set brittleness | M | H | Keep rules static; extensive tests |
| Bad JSON | M | M | Fail‑fast validation via config checker |
| RNG drift | L | H | Central seed; replay tests |

---

## 12 · Roadmap
| Milestone | ETA |
|-----------|-----|
| Spec freeze | 05 Aug 2025 |
| Core MVP | 01 Sep 2025 |
| Adapters live | 10 Sep 2025 |
| Performance harness validated | 15 Sep 2025 |

---

## 13 · Future Ideas (v3+)
### Stage & Environment Mechanics
- Encounters occur on a **Stage** object that maintains terrain, weather, or hazards.
- Moves or status abilities can **shift stage state**, adding global buffs/debuffs or DOTs.

### Dynamic Adaptations
- Stage state can **buff** or **nerf** move types (e.g., “Flames active → Fire moves +20 % damage”).
- Stage may react to existing status effects (e.g., *Frozen* units trigger a “Slippery” debuff on ground moves).

### Upgrade Implications
- Conditional rule evaluation required (breaks v2 constraint).
- AI brains must consider stage + statuses.
- Logs must capture stage transitions + cascades.

*(Exploratory; not in v2 scope)*

---

## 14 · Glossary
| Term | Meaning |
|------|---------|
| Actor | Player‑controlled combatant |
| Enemy | Actor with a Brain strategy |
| Brain | Fixed AI component (v2) |
| Status Effect | DOT, State, or Crowd‑Control effect |
| Encounter | Full match: initiative → victory/defeat |
| Round | One full rotation of Turns |
| Turn | Single combatant’s action |
| Stage | Future battlefield that can change state |
| JSONL | Newline‑delimited JSON log format |
| Deterministic | Same seed + content ⇒ identical outcome |

