# AI Driver Platform — Architecture

> **The inversion:** *You tell us the outcome. The **AI is the Driver** — it does the work.
> A human ("Bob") is the **Co-Pilot** — approves, steers, and handles exceptions.*
>
> This is the single source-of-truth architecture file. Every diagram below is written in
> **Mermaid** with the **hand-drawn (Excalidraw-style)** look. It renders on GitHub as-is,
> and can be exported to a real Excalidraw canvas (see [§9](#9-render--export-to-excalidraw)).

> **Companion matrix:** [`tools-matrix.md`](./tools-matrix.md) maps **153 business tools** by price band, category, and outcome, then shows how the Orchestrator chooses tools for local business, AI SDR, Meta/Google ads, reputation, voice, finance, ops, and forecasting.

---

## Table of Contents
1. [The One-Line Thesis](#1-the-one-line-thesis)
2. [The Inversion: Driver vs Co-Pilot](#2-the-inversion-driver-vs-co-pilot)
3. [Product Family — The Drivers](#3-product-family--the-drivers)
4. [Platform Architecture (Layers)](#4-platform-architecture-layers)
5. [The Driver Execution Loop](#5-the-driver-execution-loop)
6. [Model Layer — Unlimited OpenRouter](#6-model-layer--unlimited-openrouter)
7. [CV-as-a-Skill (`skill.md`) → "Get Selected" Automation](#7-cv-as-a-skill-skillmd--get-selected-automation)
8. [End-to-End Sequence (one outcome)](#8-end-to-end-sequence-one-outcome)
9. [Render / Export to Excalidraw](#9-render--export-to-excalidraw)
10. [Component Glossary](#10-component-glossary)

---

## 1. The One-Line Thesis

**An AI Driver is an outcome-taking machine.** The customer states a *result* ("lower my Google Ads
CPA to $40", "ship this feature", "book 10 investor meetings"). The Driver plans, acts, measures,
and iterates **unattended** — pulling any model it needs from OpenRouter with **no token budget for
customer work** — while a human Co-Pilot sits on an approval/steering rail.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart LR
  C["🎯 Customer states an OUTCOME"] --> D{{"🤖 AI Driver"}}
  D --> W["Does the work<br/>(plan → act → measure → iterate)"]
  W --> B["🧑‍✈️ Human Co-Pilot 'Bob'<br/>approves / steers / handles exceptions"]
  B -->|approved| W
  W --> R["✅ Outcome delivered<br/>+ cost ledger + evidence"]
```

---

## 2. The Inversion: Driver vs Co-Pilot

Traditional "AI co-pilots" keep the human in the driver's seat doing the work. **We flip it.**

| | Traditional Co-Pilot | **AI Driver (this platform)** |
|---|---|---|
| Who does the work | Human | **AI** |
| Who supervises | AI suggests | **Human ("Bob") approves & steers** |
| Unit of instruction | Task / keystroke | **Outcome / KPI** |
| Default mode | Human-triggered | **Unattended, always-on** |
| Human touch points | Constant | **Exception + approval gates only** |
| Accountability | Human | **Per-task economic ledger on the Driver** |

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart LR
  subgraph OLD["❌ Traditional Co-Pilot"]
    H1["🧑‍✈️ Human = DRIVER<br/>does the work"] --> A1["🤖 AI = Co-Pilot<br/>autocompletes / suggests"]
  end
  subgraph NEW["✅ AI Driver"]
    A2["🤖 AI = DRIVER<br/>executes to the outcome"] --> H2["🧑‍✈️ Human 'Bob' = Co-Pilot<br/>approves · steers · exceptions"]
  end
  OLD == "flip the seats" ==> NEW
```

---

## 3. Product Family — The Drivers

One substrate, many Drivers. Each Driver = **domain adapter + skills + KPIs** plugged into the
shared runtime. New Drivers are added by writing skills and wiring a domain tool adapter — the
engine, model layer, memory, safeguard, and co-pilot rail are reused.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart TB
  CORE(["🧠 AI Driver Runtime<br/>(shared substrate)"])
  CORE --> G["📈 Google Ads Driver<br/>KPI: CPA / ROAS"]
  CORE --> M["📘 Meta / Facebook Ads Driver<br/>KPI: CPA / ROAS"]
  CORE --> E["💻 AI Engineer Driver<br/>KPI: shipped & green PRs"]
  CORE --> AROS["💰 AROS — Revenue Driver<br/>KPI: pipeline / closed-won"]
  CORE --> ARIA["🏦 ARIA — Capital Driver<br/>KPI: investor meetings"]
  CORE --> NEXT["➕ Any future Driver<br/>(write skills + adapter)"]
```

**Driver contract (every Driver implements the same interface):**

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart LR
  IN["outcome + constraints"] --> DRV{{"Driver"}}
  SK["skills/*.md"] --> DRV
  KPI["target KPIs"] --> DRV
  DRV --> ACT["domain actions<br/>(via adapter)"]
  DRV --> OUT["result + ledger + evidence"]
```

---

## 4. Platform Architecture (Layers)

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart TB
  subgraph L1["① Outcome Intake"]
    OI["Outcome spec · KPIs · budget · guardrails · deadline"]
  end

  subgraph L2["② Driver Runtime (the Harness)"]
    PL["Planner<br/>outcome → task graph"]
    EX["Executor loop<br/>act · observe · re-plan"]
    RT["Model Router<br/>right model per step"]
    SKL["Skill Loader<br/>skill.md registry"]
  end

  subgraph L3["③ Model Layer — OpenRouter (UNLIMITED for customer work)"]
    OR["OpenRouter gateway<br/>any model, no token cap"]
  end

  subgraph L4["④ Action / Tool Layer (domain adapters)"]
    GA["Google Ads API"]
    FB["Meta Ads API"]
    DEV["Git / CI / Editor"]
    OUTR["Email · LinkedIn · CRM"]
  end

  subgraph L5["⑤ Human Co-Pilot Rail"]
    INBOX["Approval inbox"]
    STEER["Steering / overrides"]
    ESC["Exception escalation"]
  end

  subgraph L6["⑥ Memory & Learning"]
    MEM["Shared brain (MemCollab)<br/>outcomes → learned policy"]
  end

  subgraph L7["⑦ Safeguard & Ledger"]
    SG["SafeSeek circuit breakers<br/>+ compliance gates"]
    LED["Economic ledger<br/>cost + evidence per task"]
  end

  OI --> PL --> EX
  EX <--> RT --> OR
  SKL --> PL
  EX --> GA & FB & DEV & OUTR
  EX --> INBOX
  INBOX --> STEER --> EX
  EX --> ESC
  EX --> MEM --> PL
  EX --> SG
  EX --> LED
```

**Reading the layers**
- **① Intake** turns a human wish into a machine-checkable **outcome contract** (KPI, budget, guardrails, deadline).
- **② Runtime** is the Driver brain: plan → execute → re-plan, loading **skills** and routing each step to a model.
- **③ Model layer** is OpenRouter with an **unlimited-usage policy for all customer-related work** — the Driver never rations tokens on a customer's behalf.
- **④ Action layer** are thin adapters; each Driver only needs its own.
- **⑤ Co-Pilot rail** is where "Bob" lives — approvals, steering, escalation. The AI drives; the human governs.
- **⑥ Memory** makes every Driver smarter from every outcome (shared, cross-Driver).
- **⑦ Safeguard + ledger** guarantee nothing unverified ships, and every action has a cost + evidence trail.

---

## 5. The Driver Execution Loop

The heartbeat of every Driver — the same loop whether it's tuning ad bids or shipping code.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart TD
  S(["🎯 Outcome accepted"]) --> P["Plan<br/>decompose into task graph"]
  P --> A["Act<br/>call domain tool / model step"]
  A --> V["Verify<br/>did it move the KPI?"]
  V --> GATE{"Risky or<br/>low-confidence?"}
  GATE -- "yes" --> HITL["🧑‍✈️ Co-Pilot approval<br/>(Bob)"]
  HITL -- "approved" --> COMMIT["Commit action<br/>(send / deploy / spend)"]
  HITL -- "rejected" --> P
  GATE -- "no (high confidence)" --> COMMIT
  COMMIT --> MEAS["Measure & log<br/>ledger + evidence"]
  MEAS --> LEARN["Learn<br/>update policy in shared brain"]
  LEARN --> DONE{"KPI met?"}
  DONE -- "no" --> P
  DONE -- "yes" --> END(["✅ Outcome delivered"])
```

**Key rule:** confidence + risk decide whether the human is pulled in. High-confidence, low-risk
actions run unattended; anything spending real money, sending to a real person, or shipping to prod
crosses the **Co-Pilot gate** first.

---

## 6. Model Layer — Unlimited OpenRouter

Every customer-facing Driver has **no token budget**: it may call *any* model on OpenRouter as many
times as the outcome requires. Cost is tracked (ledger) but never rationed for customer work. A
router picks the right model per step — cheap-fast for classification, deep-reasoning for planning.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart LR
  STEP["Driver step<br/>(needs a model)"] --> ROUTER{{"Model Router"}}
  ROUTER -->|"classify / extract"| FAST["fast + cheap model"]
  ROUTER -->|"plan / reason"| DEEP["frontier reasoning model"]
  ROUTER -->|"write / personalize"| WRITE["strong writing model"]
  ROUTER -->|"code"| CODE["code model"]
  FAST & DEEP & WRITE & CODE --> OR["🔀 OpenRouter<br/>UNLIMITED usage for customer work"]
  OR --> LED["📒 Ledger: log cost + tokens<br/>(track, don't ration)"]
```

**Policy in one sentence:** *unlimited tokens, any model, for all customer-related work — on every
AI Driver — with full cost transparency but no customer-facing throttling.*

---

## 7. CV-as-a-Skill (`skill.md`) → "Get Selected" Automation

The CV is not a static PDF — it's a **`skill.md`**: a portable capability file the Driver loads. That
turns "getting hired / selected / funded" into an **outcome the Driver can drive**. Connect the CV
skill to a target source (jobs, RFPs, investors) and the Driver runs the same loop until it books the
meeting or lands the selection.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
flowchart TD
  CV["📄 cv.skill.md<br/>(CV expressed as a Skill)"] --> LOAD["Skill Loader<br/>registers capability"]
  LOAD --> DRV{{"🤖 Selection Driver"}}
  SRC["🎯 Target source<br/>jobs · RFPs · investors · leads"] --> DRV
  DRV --> MATCH["Match & score<br/>fit vs target"]
  MATCH --> TAILOR["Tailor artifact<br/>(CV / pitch to this target)"]
  TAILOR --> REACH["Reach out<br/>email · LinkedIn · portal"]
  REACH --> GATE{"Send?"}
  GATE -- "review" --> BOB["🧑‍✈️ Bob approves"]
  BOB --> SENT["Sent"]
  GATE -- "auto" --> SENT
  SENT --> TRACK["Track replies / outcomes"]
  TRACK --> WIN{"Selected / meeting?"}
  WIN -- "no" --> MATCH
  WIN -- "yes" --> DONE(["🏆 Got selected"])
  DONE --> MEM["Learn what worked<br/>→ shared brain"]
```

**Why this matters:** the same machinery that runs ads and ships code also **markets its own
operator** — the CV skill lets an AI Driver autonomously pursue "get selected" as a measurable KPI
(applications sent, replies, meetings, offers), with Bob approving anything that leaves the building.

---

## 8. End-to-End Sequence (one outcome)

A single outcome, start to finish, showing where the human enters.

```mermaid
%%{init: {'look':'handDrawn','theme':'neutral'}}%%
sequenceDiagram
  actor Cust as Customer
  participant Intake as Outcome Intake
  participant Driver as AI Driver Runtime
  participant OR as OpenRouter (unlimited)
  participant Tool as Domain Adapter
  participant Bob as Co-Pilot (Bob)
  participant Ledger as Ledger + Memory

  Cust->>Intake: "Hit this outcome / KPI"
  Intake->>Driver: outcome contract (KPI, budget, guardrails)
  loop until KPI met
    Driver->>OR: plan / reason (any model, no cap)
    OR-->>Driver: plan + next action
    Driver->>Tool: execute action
    Tool-->>Driver: result / metric
    alt risky or low confidence
      Driver->>Bob: request approval
      Bob-->>Driver: approve / steer / reject
    end
    Driver->>Ledger: log cost + evidence, learn
  end
  Driver-->>Cust: outcome delivered + evidence trail
```

---

## 9. Render / Export to Excalidraw

- **On GitHub / any Mermaid viewer:** diagrams render directly; the `look: handDrawn` directive gives
  the Excalidraw sketch aesthetic with zero extra tooling.
- **To a real Excalidraw canvas:**
  1. Open [https://mermaid.live](https://mermaid.live), paste any diagram block.
  2. In Excalidraw, use **Insert → Mermaid to Excalidraw**, paste the same code → fully editable shapes.
- **Keep it one file:** this document is the *single* architecture artifact by design. Diagrams live
  inline as code so they stay version-controlled and diffable — no binary image drift.

---

## 10. Component Glossary

| Component | Role |
|---|---|
| **AI Driver** | Outcome-taking agent that does the work unattended. |
| **Co-Pilot ("Bob")** | Human on the approval/steering/exception rail. |
| **Outcome Contract** | Machine-checkable KPI + budget + guardrails + deadline. |
| **Driver Runtime / Harness** | Plan → act → verify → learn engine shared by all Drivers. |
| **Model Router** | Picks the right model per step. |
| **OpenRouter (unlimited)** | Model gateway; unlimited usage for all customer work. |
| **Domain Adapter** | Thin per-Driver connector (Google Ads, Meta, Git, CRM…). |
| **Co-Pilot Rail** | Approval inbox + steering + escalation surface. |
| **Shared Brain (MemCollab)** | Cross-Driver memory; every outcome improves all Drivers. |
| **SafeSeek / Safeguard** | Circuit breakers + compliance so nothing unverified ships. |
| **Economic Ledger** | Cost + evidence trail per task. |
| **`cv.skill.md`** | CV expressed as a loadable Skill for the "get selected" Driver. |

---

*Single-file architecture · Mermaid (hand-drawn / Excalidraw look) · one source of truth.*
