# python-clean-architecture-codex

A Codex CLI agent skill that provides Clean Architecture guidance for Python/FastAPI projects.

This is the [Codex CLI](https://developers.openai.com/codex/cli/) version of [python-clean-architecture](https://github.com/MKToronto/python-clean-architecture) (originally built for Claude Code).

## Attribution

The principles, patterns, and architectural approach in this skill are inspired by and synthesized from [Arjan Codes](https://www.arjancodes.com/)' courses:

- **The Software Designer Mindset** — Seven core design principles (cohesion, coupling, abstractions, composition, creation/use separation, data-first design, simplicity)
- **Pythonic Patterns** — Classic GoF patterns reimagined for Python using Protocol, Callable, functools.partial, and closures
- **Complete Extension** — Three-layer FastAPI architecture (Routers → Operations → Database) with Protocol-based dependency injection

This skill distills those principles into actionable guidance. It is not a reproduction of course content. If you find this useful, consider supporting Arjan's work at [arjancodes.com](https://www.arjancodes.com/).

## Installation

### Per-project (recommended)

Clone into your project's `.agents/skills/` directory:

```bash
mkdir -p .agents/skills
git clone https://github.com/MKToronto/python-clean-architecture-codex.git .agents/skills/python-clean-architecture
```

Codex will automatically discover the skill from `.agents/skills/`.

### User-wide

Install to your home directory so the skill is available in all projects:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/MKToronto/python-clean-architecture-codex.git ~/.agents/skills/python-clean-architecture
```

### Verify installation

Start Codex and ask:

```
review my code structure
```

or

```
scaffold a FastAPI project
```

The skill should trigger automatically.

## Usage

The skill triggers automatically when you mention clean architecture, FastAPI scaffolding, code quality, design patterns, or related topics.

### Available Workflows

**Review & Analysis:**
- **Review Architecture** — Full architecture review (standard or in-depth) against the 7 design principles, 17 quality rules, and Pythonic patterns
- **Check Quality** — Quick check against 17 code quality rules
- **Suggest Patterns** — Recommend Pythonic design patterns for your code
- **Decouple** — Find tight coupling and suggest dependency injection improvements

**Refactoring:**
- **Make Pythonic** — Refactor to Pythonic patterns (ABC→Protocol, if/elif→dict, etc.)
- **Extract God Class** — Find and split god classes into focused collaborators

**Scaffolding:**
- **Scaffold API** — Generate a new FastAPI project with clean architecture
- **Add Endpoint** — Scaffold a new endpoint across all three layers

### Example prompts

```
review architecture of src/
check quality of my code
suggest patterns for src/services/
decouple src/
make src/services/payment.py pythonic
extract god classes in src/
scaffold a FastAPI project called hotel-api
add an endpoint for bookings
```

## What's Inside

```
python-clean-architecture-codex/
├── .agents/
│   └── skills/
│       └── clean-architecture/
│           ├── SKILL.md                        Skill definition + all workflows
│           ├── references/
│           │   ├── design-principles.md        7 principles with refactoring recipes
│           │   ├── layered-architecture.md     3-layer FastAPI guide with full code
│           │   ├── testable-api.md             Stub-based testing strategy
│           │   ├── code-quality.md             17 rules + code review checklist
│           │   ├── classes-and-dataclasses.md  Classes vs dataclasses decision guide
│           │   ├── function-design.md          Pure functions, closures, partial, HOFs
│           │   ├── data-structures.md          Choosing the right data structure
│           │   ├── error-handling.md           Custom exceptions, context managers
│           │   ├── types-and-type-hints.md     Python's type system, Callable types
│           │   ├── project-organization.md     Modules, packages, folder structure
│           │   ├── pythonic-patterns.md        Quick reference lookup table
│           │   └── patterns/
│           │       ├── strategy.md             Full OOP → functional progression
│           │       ├── abstract-factory.md     Tuples of functions + partial
│           │       ├── bridge.md               Bound methods, when to stop
│           │       ├── command.md              Undo closures, batch commands
│           │       ├── notification.md         Observer → Mediator → Pub/Sub
│           │       ├── registry.md             Dict mapping, importlib plugins
│           │       ├── template-method.md      Free function + Protocol
│           │       ├── pipeline.md             Chain of Responsibility, compose
│           │       └── functional.md           Callback, Wrapper, Builder
│           └── assets/
│               └── fastapi-hotel-api/          Complete working FastAPI project
│                   ├── main.py
│                   ├── models/                 Pydantic models + DataInterface Protocol
│                   ├── operations/             Business logic (accepts Protocol)
│                   ├── routers/                API endpoints (composition root)
│                   └── db/                     SQLAlchemy + generic DBInterface
├── README.md                                   This file
└── .gitignore
```

## Key Concepts

### Three-Layer Architecture

```
Routers (API)  →  Operations (business logic)  →  Database (persistence)
```

Each layer depends only on the layer below. The router is the composition root where concrete DB implementations are injected into operations via the `DataInterface` Protocol.

### Seven Design Principles

1. High Cohesion — Single responsibility per unit
2. Low Coupling — Minimize dependencies, Law of Demeter
3. Depend on Abstractions — Protocol + Callable for DI
4. Composition over Inheritance — Never mixins, shallow hierarchies only
5. Separate Creation from Use — Dict mapping, creator functions, one composition root
6. Start with the Data — Information Expert, fix data structures first
7. Keep Things Simple — DRY, KISS, YAGNI (but avoid hasty abstractions)

### Pythonic Pattern Defaults

- Protocol over ABC (unless shared superclass state needed)
- `functools.partial` over wrapper classes
- Closures over factory class hierarchies
- `Callable` type aliases over single-method abstract classes
- Dict mapping over if/elif chains
- Readability over dogmatic functional purity

## Also available for

- **Claude Code** — [python-clean-architecture](https://github.com/MKToronto/python-clean-architecture) (plugin with slash commands)
