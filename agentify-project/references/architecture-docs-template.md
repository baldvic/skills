# Architecture documentation template

Structure for the `docs/` output the architecture-documentation stage generates or merges. Grounded strictly in what the stack analysis actually finds — never invent a service, endpoint, or data model that isn't backed by something present in the repo. Read this before drafting or updating architecture docs.

## `docs/ARCHITECTURE.md` — always generated (every repo, any size)

Sections, in order:

```markdown
# Architecture

## System context

<1-3 paragraphs: what this system is, who/what it talks to (users, other systems,
external APIs) — inferred from README, entry points, and config actually present.>

## Component/container diagram

```mermaid
graph TD
  <One node per independently-versioned service/component found by stack detection,
  plus external dependencies (databases, message brokers, third-party APIs) actually
  referenced in config or code. Edges only for connections you can point to a file for.>
```

## Service/component catalog

| Component | Tech stack | Entry point | Role |
|---|---|---|---|
<One row per component found. Cite the actual manifest/entry-point file.>

## Messaging topology

<Only if a message broker/queue is actually present in the stack — a table of
topics/queues/exchanges and producer/consumer components. Omit this section
entirely if no messaging system is detected; don't include an empty table.>

## Cross-component sequences

```mermaid
sequenceDiagram
  <One diagram per notable cross-component flow you can trace through actual code
  (e.g. request handling, an async job). Skip if the repo is a single component
  with no internal cross-calls to diagram.>
```

## Shared data model

```mermaid
erDiagram
  <Entities and relationships actually defined in schemas/models found in the repo.
  Omit if there's no shared data model — e.g. a CLI tool with no persistence.>
```

## Known issues

<Grounded observations only — e.g. a component with no tests, a hardcoded config
value, a TODO left in code — each citing the specific file:line. Omit the section
if analysis surfaces nothing worth flagging; don't pad it with generic advice.>
```

Every section is conditional on the analysis actually finding something to put there, except System context and the Component/container diagram and Service/component catalog, which every repo gets in some form (even a single-component repo gets a one-node diagram and a one-row catalog).

## `docs/services/` tier — multi-service repos only

If the stack analysis finds **more than one** independently-versioned service or component, additionally generate:

- **`docs/services/README.md`** — an index: one line per service linking to its detail file.
- **`docs/services/<name>.md`** per component:

```markdown
# <service name>

## Role

<One paragraph: what this service does in the system.>

## Tech stack & entry point

<Language/framework, main entry file (cite it).>

## API surface

<Endpoints/routes actually defined in the code, with method + path, citing file:line.
Omit if the service exposes no API (e.g. a background worker).>

## Data models

<Models/schemas this service owns, citing file:line.>

## Messaging in/out

<Topics/queues this service produces to or consumes from, if any.>

## Known issues

<Grounded, file:line-cited observations specific to this service. Omit if none.>
```

**Single-service repos do not get this tier at all** — no `docs/services/` directory, no single padded entry standing in for a catalog that doesn't apply. `docs/ARCHITECTURE.md` alone is the complete output for a single-service repo.

## Merge behavior for existing docs

If `docs/ARCHITECTURE.md` (or a `docs/services/<name>.md`) already exists:

1. Read the existing file in full.
2. Compute what's demonstrably stale against the current detection-map/stack-analysis findings (a component added since the doc was written, an endpoint that no longer exists, a data model field that's been removed) versus what's hand-authored prose with no mechanical signal to check it against (a "why this system exists" paragraph, a design-rationale note).
3. Update only the stale/missing sections. Preserve hand-authored prose untouched, even if it's not something the analysis could have generated itself — the same merge-first principle used for agent instructions (see [agent-readiness-checklist.md](agent-readiness-checklist.md)) applies here.
4. Never wholesale-regenerate an existing architecture doc just because some section is stale.

This is committed as its own stage in the pull request (see [pr-delivery.md](pr-delivery.md)) — never written directly outside the PR flow.
