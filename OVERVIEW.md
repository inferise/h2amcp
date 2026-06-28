# H2A — Overview by Audience

H2A (Human Interface to Agentic Interface) is a specification that transforms product interfaces designed for human interaction, like websites and apps, into tool interfaces for AI agents. It establishes a stable contract defining a website's capabilities for AI agents, while intricate details like interactions and text are encapsulated in an updatable layer. This ensures consistent and stable AI agent tools even after redesigns.

Two entities, the agent and the orchestrator, are involved in this contract. The agent defines the task, like hiring a driver and specifying the destination. The orchestrator handles the execution, like the driver navigating the roads. H2A bridges the gap: it allows the agent to request an outcome without knowing the route, and it enables the orchestrator to execute it without knowing the user’s ultimate desire.

This document explains the same concept four times, each for a different level of technical depth. Read the one that fits; each is self-contained.

The four levels, from least to most technical:

| Level          | Who they are                                                    | What they want from this page                      |
| -------------- | --------------------------------------------------------------- | -------------------------------------------------- |
| **End User**   | Anyone who just wants to know what it is                        | A plain-language analogy, no jargon                |
| **Power User** | Technically comfortable non-engineers — analysts, ops, PMs      | A working mental model with light jargon           |
| **Technical**  | Engineers integrating or consuming H2A, not designing it        | The architecture and why it's shaped this way      |
| **Expert**     | Spec authors, protocol designers, people who will implement H2A | The precise model and artifact shapes              |

---

## End User

Imagine contracting a driver to take you from New York to San Francisco. The two of you agree on a contract: the inputs you set (your pickup spot, the fare, constraints like "no overnight stops") and the output you're owed (you, delivered to San Francisco), plus what happens if something goes wrong. Notice the line the contract draws — **what** gets done is yours to name; **how** it gets done, which streets and which route, is entirely the driver's. If there's roadwork and lanes move, the driver reroutes on their own and the contract never changes, because it only ever named the destination and the terms, not the roads.

H2A is that same arrangement for an AI agent using a website. A site is built for people, not agents, so the agent doesn't inherently know the way. H2A writes down the contract — the inputs you give and the result you get back — and keeps it separate from the route, the actual steps that pull it off on this particular site. A redesign only redraws the route; the contract is untouched, so your agent still delivers exactly what you asked for.

## Power User

H2A lets an AI agent reliably _use_ a website or app on your behalf, and keep working when the site changes. It runs on the same bargain as contracting a driver: you settle **what** is to be done, the contractor settles **how**. Three pieces carry that split. The **contract** is the *what* — the inputs you supply and the output you're owed, written once and shared by every provider, never dictating how the work is done. The **manifest** says which of a contract's actions a given provider actually offers, and over which channel (website, app, or direct connection). The **binding** is the *how* — the specific route on this particular site that fulfills the contract; the contractor's business, not yours. The agent reads only the contract and manifest, so it never sees the route. A redesign redraws the route while the contract holds still, so the agent doesn't break.

## Technical

H2A is a stable API facade over an unstable UI, split into three layers. The **contract** is the *what*: an abstract interface for a category — typed inputs, outputs, and a fixed error set — written once and shared by every provider. The **manifest** is per provider and per surface (web, app, REST); it declares which of the contract's methods are live and pins exact versions. The **binding** is the *how*: a private recording of the steps to perform each method on that site — resilient multi-strategy locators, slots where parameters are injected, and rules for reading results and recognizing errors.

An agent reads manifest + contracts to know what tools exist and emits the call; a separate orchestrator resolves the binding and drives the page. When the site is redesigned, only the binding is re-recorded and version-bumped — the contract doesn't move, so nothing the agent sees changes, and a validator keeps the layers consistent.

## Expert

H2A models a category's actions as a **contract** — the *what*: an abstract, shared tool interface whose methods each declare `params`, `returns`, and a closed `errors` set in a constrained JSON-Schema subset, authored once and reused across providers. Concrete reality is partitioned by **org** and **surface** (`www`, `app`, `rest`). A **manifest** (`org/<org>/<surface>/manifest.json`) declares which contracts and which subset of methods are live and pins exact contract/binding versions — the single source of truth for what is callable. A **binding** (`org/<org>/<surface>/<contract>@<version>.json`) is the *how*: a `steps` recording (`navigate`, `click`, `input`, `select`, `submit`, `waitFor`, `assert`, `read`) over ordered multi-strategy locators, with a `{{param}}`→field input map, `extract`→`returns` output map, explicit waits, and a failure→error-code map.

The contract is the boundary between two planes: the **agent** consumes manifest + contracts to render the tool surface and validate args, emitting `org.surface.contract.method(...)`; the **orchestrator** consumes manifest + bindings to run the recording and needs no contract at runtime. Versioning is independent per artifact: the contract version is the agreement between manifest and binding, while the binding version floats freely beneath it, so a redesign is a binding-only republish. Pins are exact and nothing auto-upgrades, and publish-time invariants keep the three artifacts coherent with each other.
