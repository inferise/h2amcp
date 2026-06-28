# H2A MCP

**Human Interface to Agentic Interface** — an MCP specification for re-describing products built for human operation (websites, desktop and mobile apps, REST surfaces) as agent-callable MCP tools.

H2A maps a human interface onto an agentic interface: it wraps an interface originally built for human use so an agent can operate it programmatically. Human-facing interfaces change continually, and H2A's central property is that it **confines that volatility to the binding layer**, beneath a stable, versioned, agent-facing tool surface. A site redesign becomes a binding-only republish — nothing the agent sees changes.

## Core idea

A category's actions (check a balance, transfer funds, pay a bill) are modeled as a **contract** — an abstract tool interface shared across every site that implements the category. Each org declares, per interaction **surface** (`www`, `app`, `rest`), which contracts and which methods are live in a **manifest**, and the concrete recording that performs each method lives in a **binding**.

At its heart is a **what/how split**: the contract fixes *what* an agent can do — the inputs it supplies and the result it gets back — while the binding holds *how* that gets carried out on a given site. Contracts are abstract and shared; manifests and bindings are concrete and partitioned by org and surface. Confining the volatile *how* to the binding is what lets a site change without touching the agent-visible *what*.

## Participants

Two parties meet at the contract, and the spec exists to bridge them. The **agent** defines the task — the *what*. It reads *manifest + contracts*, sees only the live tools, and emits a fully-qualified call like `somebank.www.banking.transfer(...)`, asking for an outcome without knowing the route. The **orchestrator** carries it out — the *how*. It reads *manifest + bindings*, resolves the private recording, and drives the page, executing the task without knowing the user's ultimate goal. Both pivot on the manifest; neither needs the other's half at runtime.

## Artifacts

| Artifact     | Scope                          | Defines                                                        | Consumed by        |
| ------------ | ------------------------------ | ------------------------------------------------------------- | ------------------ |
| **Contract** | Shared, per category           | Abstract methods + param/return/error schemas                 | Agent + Binding    |
| **Manifest** | Per (org × surface)            | Which contracts, which methods, and which binding are live    | Agent              |
| **Binding**  | Per (org × surface × contract) | The recording that performs each method                       | Orchestrator only  |
| **Surface**  | Global vocabulary              | The closed set of interaction surfaces                        | Manifest + Binding |

The contract is the boundary between the two planes (see [Participants](#participants)): the agent consumes *manifest + contracts*, the orchestrator *manifest + bindings*.

## Repository layout

```
spec/1.0.md                              # the normative specification
schema/1.0/                              # JSON Schemas (the machine-readable spec)
  contract.json  manifest.json  binding.json  surface.json
contract/
  banking@1.0.json                       # shared, abstract; versioned in the filename
org/
  <org>/<surface>/
    manifest.json                        # the (org × surface) declaration
    <contract>@<bindingVersion>.json     # the binding/recording
tools/validate.py                        # conformance checker
Makefile
```

Schemas are versioned as a set (`schema/1.0/`); instances pin the spec they conform to via their `$schema` pointer and carry their own version in the filename.

## Example

A complete worked slice lives under `contract/banking@1.0.json` and `org/somebank/www/` (`manifest.json` + `banking@1.0.json`). A bank website would exposes the `banking` category on its `www` surface, declaring only `checkBalance`, `transfer`, and `payBill` of the contract's full method set.

## Validate

```
make validate
```

Walks every org manifest, resolves the contracts and bindings it pins, and checks structure plus the spec invariants (method subset, conformance floor, binding coverage, contract-pin match, bind-map integrity). Pure `python3`, no dependencies; if `jsonschema` is importable it additionally validates each instance against `schema/1.0/`.

## Specification

`spec/1.0.md` is the full normative reference: artifact definitions, the agent/orchestrator split, the fully-qualified tool name, resolution flow, versioning rules, and the validation invariants. Start there for anything beyond this overview.
