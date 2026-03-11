# python-clean-architecture-codex

A Codex CLI agent skill that provides Clean Architecture guidance for Python/FastAPI projects.

This is the [Codex CLI](https://developers.openai.com/codex/cli/) version of [python-clean-architecture](https://github.com/MKToronto/python-clean-architecture) (originally built for Claude Code).

## Attribution

The principles, patterns, and architectural approach in this skill are inspired by and synthesized from [Arjan Codes](https://www.arjancodes.com/)' courses:

- **The Software Designer Mindset** — Seven core design principles (cohesion, coupling, abstractions, composition, creation/use separation, data-first design, simplicity)
- **Pythonic Patterns** — Classic GoF patterns reimagined for Python using Protocol, Callable, functools.partial, and closures
- **Complete Extension** — Three-layer FastAPI architecture (Routers → Operations → Database) with Protocol-based dependency injection

The specific Pythonic framing (Protocol-based DI, functional pattern progression, three-layer FastAPI architecture) originates from his teaching. This skill distills those principles into actionable guidance — it is not a reproduction of course content. If you find this useful, consider supporting Arjan's work at [arjancodes.com](https://www.arjancodes.com/), [github.com/arjancodes](https://github.com/arjancodes), and [youtube.com/arjancodes](https://www.youtube.com/arjancodes).

## Installation

In Codex, type:

```
$skill-installer install https://github.com/MKToronto/python-clean-architecture-codex
```

Codex will ask you to approve a few steps — say yes each time. Once it finishes, restart Codex to pick up the skill.

The skill installs to `~/.codex/skills/clean-architecture`.

## Usage

There are no slash commands. You talk to Codex naturally and the skill triggers automatically, or you can invoke it explicitly with `$python-clean-architecture`.

### Explicit invocation

Prefix your request with the skill name:

```
$python-clean-architecture review architecture of src/
$python-clean-architecture check quality of my code
$python-clean-architecture suggest patterns for src/services/
$python-clean-architecture decouple src/
$python-clean-architecture make src/services/payment.py pythonic
$python-clean-architecture extract god classes in src/
$python-clean-architecture scaffold a FastAPI project called hotel-api
$python-clean-architecture add an endpoint for bookings
```

### Natural language (no $ needed)

The skill also triggers automatically when your prompt matches. Just ask:

```
review my code structure
scaffold a new FastAPI project
refactor to clean architecture
add a new endpoint for orders
check my code quality
suggest design patterns for this code
decouple my services
```

### Available Workflows

**Review & Analysis:**
- **Review Architecture** — Full architecture review (standard or in-depth) against the 7 design principles, 22 quality rules, and all 25 Pythonic patterns
- **Check Quality** — Quick check against 22 code quality rules
- **Suggest Patterns** — Recommend Pythonic design patterns for your code
- **Decouple** — Find tight coupling and suggest dependency injection improvements

**Refactoring:**
- **Make Pythonic** — Refactor to Pythonic patterns (ABC→Protocol, if/elif→dict, etc.)
- **Extract God Class** — Find and split god classes into focused collaborators

**Scaffolding:**
- **Scaffold API** — Generate a new FastAPI project with clean architecture
- **Add Endpoint** — Scaffold a new endpoint across all three layers

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
│           │   ├── testing-advanced.md         Pytest, property-based, stateful testing
│           │   ├── rest-api-design.md          HTTP methods, status codes, OpenAPI
│           │   ├── code-quality.md             22 rules + code review checklist
│           │   ├── classes-and-dataclasses.md  Classes vs dataclasses decision guide
│           │   ├── function-design.md          Pure functions, closures, partial, HOFs
│           │   ├── data-structures.md          Choosing the right data structure
│           │   ├── error-handling.md           Custom exceptions, context managers
│           │   ├── monadic-error-handling.md   Railway-oriented Result types
│           │   ├── types-and-type-hints.md     Python's type system, Callable types
│           │   ├── project-organization.md     Modules, packages, folder structure
│           │   ├── context-managers.md         __enter__/__exit__, @contextmanager
│           │   ├── decorators.md               Retry, logging, timing, parameterized
│           │   ├── async-patterns.md           Async/await, gather, TaskGroup
│           │   ├── pydantic-validation.md      Pydantic v2 validators, ConfigDict
│           │   ├── pattern-matching.md         match/case structural patterns
│           │   ├── grasp-principles.md         GRASP: 9 principles for responsibility assignment
│           │   ├── domain-driven-design.md     DDD: domain models, ubiquitous language
│           │   ├── domain-modeling.md          Entities, value objects, relationships
│           │   ├── dependency-injection.md     Manual DI to Protocol-based abstraction
│           │   ├── code-smells.md              Code smell catalog with targeted fixes
│           │   ├── refactoring-case-studies.md Recurring transformations from code reviews
│           │   ├── pythonic-patterns.md        Quick reference for all 25 patterns
│           │   └── patterns/
│           │       ├── strategy.md             Full OOP → functional progression
│           │       ├── abstract-factory.md     Tuples of functions + partial
│           │       ├── bridge.md               Bound methods, when to stop
│           │       ├── command.md              Undo closures, batch commands
│           │       ├── notification.md         Observer → Mediator → Pub/Sub
│           │       ├── registry.md             Dict mapping, importlib plugins
│           │       ├── template-method.md      Free function + Protocol
│           │       ├── pipeline.md             Chain of Responsibility, compose
│           │       ├── functional.md           Callback, Wrapper, Builder
│           │       ├── value-objects.md        Validated domain primitives
│           │       ├── event-sourcing.md       Immutable events, projections
│           │       ├── cqrs.md                 Separate read/write models
│           │       ├── builder.md              Fluent API, frozen product
│           │       ├── unit-of-work.md         Transaction context managers
│           │       ├── singleton.md            Module-level instance, metaclass
│           │       ├── state.md                Protocol-based state objects
│           │       ├── adapter.md              Composition + partial adaptation
│           │       ├── facade.md               Simplified subsystem interface
│           │       ├── repository.md           Separating storage from access
│           │       ├── fluent-interface.md     Method chaining, domain verbs
│           │       ├── retry.md                Exponential backoff decorator
│           │       ├── lazy-loading.md         cache, cached_property, generators
│           │       └── plugin-architecture.md  Config-driven, importlib discovery
│           └── assets/
│               └── fastapi-hotel-api/          Complete working FastAPI project
│                   ├── main.py
│                   ├── models/             Pydantic models + DataInterface Protocol
│                   ├── operations/         Business logic (accepts Protocol)
│                   ├── routers/            API endpoints (composition root)
│                   ├── db/                 SQLAlchemy + generic DBInterface
│                   └── tests/              Stub-based tests (no DB needed)
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

25 design patterns implemented the Pythonic way:

**Core Patterns:**
- **Strategy** — `Callable` type alias, pass functions as args
- **Abstract Factory** — Tuples of functions + `functools.partial`
- **Bridge** — `Callable` type alias replaces abstract reference
- **Command** — Functions returning undo closures
- **Pub/Sub** — Dict-based `subscribe(event, handler)` / `post_event(event, data)`
- **Registry** — `dict[str, Callable]` mapping + `**kwargs` unpacking
- **Template Method** — Free function + Protocol parameter
- **Pipeline** — `functools.reduce` for composition
- **Callback / Wrapper / Builder** — Functional patterns for event handling, interface translation, and configuration

**Extended Patterns:**
- **Value Objects** — Subclass built-in types with `__new__` validation, or frozen dataclass
- **Event Sourcing** — Immutable events, append-only EventStore, projection functions
- **CQRS** — Separate write model + read projection, projector function after writes
- **Builder** — Fluent API with `Self` return type, `.build()` returns frozen product
- **Unit of Work** — Context manager wrapping transaction: commit on success, rollback on error
- **Singleton** — Module-level instance (preferred), or metaclass with `_instances` dict
- **State** — Protocol-based state objects, context delegates to current state
- **Adapter** — Protocol interface + `functools.partial` for single-method adaptation
- **Facade** — Simplified interface class, `functools.partial` to bind dependencies
- **Retry** — `@retry` decorator with exponential backoff, fallback strategies
- **Lazy Loading** — `functools.cache`, `cached_property`, generators, `__getattr__`
- **Repository** — Protocol interface for CRUD, concrete implementations per storage backend
- **Fluent Interface** — Methods return `self` for chaining, domain-specific verbs for readability
- **Plugin Architecture** — Config-driven creation, `importlib` auto-discovery, self-registering modules

## Also available for

- **Claude Code** — [python-clean-architecture](https://github.com/MKToronto/python-clean-architecture) (plugin with slash commands)
