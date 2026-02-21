---
name: python-clean-architecture
description: This skill should be used when the user asks to "scaffold a FastAPI project", "set up clean architecture", "refactor to clean architecture", "add a new endpoint", "add a router", "add a use case", "add a repository", "review my code structure", "review architecture", "check code quality", "apply design patterns in Python", "decouple my code", "improve code quality", "make my code testable", "make pythonic", "extract god class", or mentions layered architecture, dependency injection, Protocol-based design, or Pythonic design patterns for Python/FastAPI projects.
---

# Python Clean Architecture

Provide Clean Architecture guidance for Python projects, specifically FastAPI APIs. Based on seven core design principles, Pythonic implementations of classic patterns, and a three-layer architecture (Routers → Operations → Database).

> **Attribution:** The principles, patterns, and architectural approach in this skill are inspired by and synthesized from [Arjan Codes](https://www.arjancodes.com/)' courses: *The Software Designer Mindset*, *Pythonic Patterns*, and *Complete Extension*. The specific Pythonic framing (Protocol-based DI, functional pattern progression, three-layer FastAPI architecture) originates from his teaching. This skill distills those principles into actionable guidance — it is not a reproduction of course content.

## When to Apply

- Scaffolding new FastAPI projects with clean separation of concerns
- Refactoring existing Python code to reduce coupling and increase cohesion
- Adding new components (endpoints, use cases, repositories, models)
- Reviewing code for design quality and Pythonic idiom adherence
- Making code testable through dependency injection and Protocol-based abstractions

## Core Architecture: Three Layers

Every FastAPI project follows a strict three-layer dependency flow:

```
Routers (API layer)  →  Operations (business logic)  →  Database (persistence)
```

Each layer depends ONLY on the layer below it. Never skip layers.

### Layer Responsibilities

**Routers** — HTTP interface. Accept requests, call operations, return responses. No business logic. Act as the composition root where concrete implementations are injected.

**Operations** — Business logic. Accept a `DataInterface` (Protocol) parameter for data access. Compute derived values, enforce rules, orchestrate workflows. Never import database modules directly.

**Database** — Persistence. Implement the `DataInterface` Protocol using SQLAlchemy, file storage, or any backend. Expose `read_by_id`, `read_all`, `create`, `update`, `delete` methods. Return `DataObject = dict[str, Any]` to decouple from ORM models.

### The DataInterface Protocol

```python
from typing import Any, Protocol

DataObject = dict[str, Any]

class DataInterface(Protocol):
    def read_by_id(self, id: str) -> DataObject: ...
    def read_all(self) -> list[DataObject]: ...
    def create(self, data: DataObject) -> DataObject: ...
    def update(self, id: str, data: DataObject) -> DataObject: ...
    def delete(self, id: str) -> None: ...
```

Operations accept `data_interface: DataInterface` as a parameter. The router passes the concrete implementation. This is dependency injection — no ABC inheritance needed.

### Pydantic Models

Define separate Create and Read models per entity:

```python
class CustomerCreate(BaseModel):
    name: str
    email: str

class Customer(BaseModel):
    id: str
    name: str
    email: str
```

Use `**data.dict()` to unpack Pydantic models into DataObject dicts. Use `exclude_none=True` on update models for partial updates.

## Seven Design Principles

When writing or reviewing code, apply these principles in order of priority:

1. **High Cohesion** — Each class/function has a single responsibility. Extract class when a class does two things. Extract function when a function does two things.
2. **Low Coupling** — Minimize dependencies. Move methods to the class that owns the data (Information Expert). Pass only what a function needs.
3. **Depend on Abstractions** — Use `Protocol` for structural typing and `Callable` type aliases for function injection. Patterns emerge from good abstraction.
4. **Composition over Inheritance** — Extract shared behavior into standalone classes with a Protocol interface. Use `list[PaymentSource]` patterns. Never use mixins.
5. **Separate Creation from Use** — One composition root (the router or main function). Dictionary mapping over if/elif chains. Creator functions for complex construction.
6. **Start with the Data** — Fix data structures first; methods and layers follow. Extract domain classes from prefixed fields. Eliminate parallel lists.
7. **Keep Things Simple** — DRY (but avoid hasty abstractions). KISS (simplest design that works). YAGNI (implement only what is needed now).

## Pythonic Pattern Selection

When encountering these code smells, apply the corresponding pattern:

| Code Smell | Pattern | Pythonic Implementation |
|---|---|---|
| Long if/elif switching behavior | Strategy | `Callable` type alias, pass functions as args |
| Need to create objects from config/JSON | Registry | `dict[str, Callable]` mapping + `**kwargs` unpacking |
| Event/notification side effects mixed with core logic | Pub/Sub | `subscribe(event, handler)` / `post_event(event, data)` dict-based |
| Duplicated algorithm across classes | Template Method | Extract to free function + Protocol parameter |
| Two independent hierarchies that vary | Bridge | `Callable` type alias replaces abstract reference |
| Need undo/batch/queue operations | Command | Functions returning undo closures |
| Object creation coupled to usage | Abstract Factory | Tuples of functions + `functools.partial` |
| Sequential data transformations | Pipeline | `functools.reduce` for composition, or framework `.pipe()` |
| Need to react to events without coupling | Callback | Function passed as argument, called when event occurs |
| Reusing a function with a different interface | Function Wrapper | Calls another function, translates its arguments |
| Separating configuration from usage | Function Builder | Higher-order function returns a configured function |

### Default preferences for all patterns:

- **Protocol over ABC** — unless shared state in the superclass is needed
- **`functools.partial`** — to configure generic functions rather than creating wrapper classes
- **Closures** — to separate configuration-time args from runtime args
- **Don't go full functional** — find the happy medium; readability is the ultimate arbiter

## Scaffolding a New Project

When asked to create a new FastAPI project, generate this structure:

```
project_name/
├── src/
│   ├── main.py              # FastAPI app, include_router, startup events
│   ├── routers/
│   │   ├── __init__.py
│   │   └── {entity}.py      # APIRouter, endpoint functions
│   ├── operations/
│   │   ├── __init__.py
│   │   └── {entity}.py      # Business logic functions
│   ├── db/
│   │   ├── __init__.py
│   │   ├── database.py       # Engine, SessionLocal, Base
│   │   ├── db_interface.py   # Generic DBInterface class
│   │   └── models.py         # SQLAlchemy models
│   └── models/
│       ├── __init__.py
│       └── {entity}.py       # Pydantic models + DataInterface Protocol
├── tests/
│   └── test_{entity}.py      # Tests using DataInterfaceStub
├── requirements.txt
└── .gitignore
```

## Testing Strategy

Create a `DataInterfaceStub` base class that stores data in a plain dict. Override specific methods in test-specific subclasses. Pass the stub to operations functions — no database needed.

```python
class DataInterfaceStub:
    def __init__(self):
        self.data: dict[str, DataObject] = {}
    def read_by_id(self, id: str) -> DataObject:
        return self.data[id]
    def read_all(self) -> list[DataObject]:
        return list(self.data.values())
    def create(self, data: DataObject) -> DataObject:
        self.data[data["id"]] = data
        return data
    def update(self, id: str, data: DataObject) -> DataObject:
        self.data[id].update(data)
        return self.data[id]
    def delete(self, id: str) -> None:
        del self.data[id]
```

---

## Workflows

The following workflows are available. The user can request any of these by name or by describing what they need.

---

### Review Architecture

**Trigger:** User asks to "review architecture", "review my code", "check code structure", or similar.

**Input:** A file or directory path (default: current working directory).

Before starting the review, ask the user: "Which review depth?" with options:
1. **Standard** — Reviews against built-in checklists. No extra file loading.
2. **In-depth** — Loads detailed reference files for deeper analysis with refactoring recipes and pattern guidance.

#### Review Process

1. **Read the code** — Find and read ALL Python files in the target path recursively. Read every `.py` file — do not skip any. Understand the project structure, imports, and relationships between files.

2. **Check architecture layers** — Verify the three-layer separation:
   - Routers (API) → Operations (business logic) → Database (persistence)
   - No layer skipping (routers should not import DB modules directly)
   - Operations accept `DataInterface` Protocol, not concrete DB classes
   - Router acts as composition root (wires concrete implementations)

3. **Check the 7 design principles** — For each file/class/function, evaluate:
   - **High Cohesion** — Does each unit have a single responsibility?
   - **Low Coupling** — Are dependencies minimized? Any Law of Demeter violations?
   - **Depend on Abstractions** — Protocol/Callable used for DI? Or concrete imports?
   - **Composition over Inheritance** — Any deep hierarchies or mixins?
   - **Separate Creation from Use** — One composition root? Dict mapping over if/elif?
   - **Start with the Data** — Prefixed fields? Parallel lists? Methods far from data?
   - **Keep Things Simple** — Over-engineered? Speculative features? Unnecessary abstractions?

4. **Check code quality** — Apply the 17 design rules:
   - No type abuse, vague identifiers, flag parameters, deep nesting
   - Tell don't ask, no parallel data structures, no god classes
   - No broad exception catching, no mutable defaults, no wildcard imports
   - Type hints on all functions, enums for fixed options, context managers for resources

5. **Check Pythonic patterns** — Are patterns implemented the Pythonic way?
   - Strategy as Callable, not ABC hierarchy
   - Factory as dict mapping, not if/elif
   - Protocol over ABC (unless shared state needed)
   - functools.partial for configuration, closures for builders
   - Pub/Sub for notification side effects, not inline calls
   - Registry for data-driven object creation, not if/elif
   - Context managers for resource management, not try/finally
   - Custom exception classes, not broad except catching
   - Dataclasses for value objects and DTOs, not manual __init__
   - Pure functions where possible, side effects pushed to boundaries

6. **Check code cleanup opportunities** — Look for concrete refactoring targets:
   - Extract class when a class does two things
   - Extract function when a function does two things
   - Replace mutable globals with injected dataclass context
   - Replace boolean flag parameters with separate functions
   - Replace string constants with Enums
   - Replace deep nesting (3+ levels) with early returns or extraction
   - Replace god classes (>200 lines, >5 responsibilities) with collaborators
   - Move methods to the class that owns the data (Information Expert)
   - Replace parallel data structures with a single data class
   - Replace if/elif chains with dict mapping

7. **Report findings** — Structure the output in this order:

   #### Architecture Summary
   Classify every Python file into a layer, then show the dependency flows. Use a simple vertical list format — never multi-column ASCII art. Example:

   ```
   Layers
   ──────
   API:        routes/booking.py, routes/customer.py
   Logic:      operations/booking.py, operations/customer.py
   Database:   db/db_interface.py, db/models.py, db/database.py
   Unclear:    utils/helpers.py (mixed concerns)

   Dependency Flows
   ────────────────
   routes/booking.py → operations/booking.py → db/db_interface.py    ✓
   routes/customer.py → operations/customer.py → db/db_interface.py  ✓
   routes/admin.py → db/models.py                                    ⚠ layer skip

   Layer Violations
   ────────────────
   ⚠ routes/admin.py imports db/models.py directly — should go through operations
   ```

   Adapt to the actual project. If it does not follow the three-layer model, show the dependency flow as-is and note what a target architecture would look like. Keep it compact.

   #### What Works Well
   Call out specific things the code does right. Reference files and patterns. Be specific — cite actual files and lines, not generic praise.

   #### Findings by Severity
   Present issues organized by severity:
   - **Critical** — Layer violations, missing abstractions, tight coupling to concrete DB
   - **Important** — Cohesion issues, Law of Demeter violations, missing type hints
   - **Suggestions** — Pattern improvements, naming, simplification opportunities, cleanup refactorings

   For each finding, include all of the following:
   1. **File and line** — exact location
   2. **Principle(s)** — which of the 7 design principles are violated, by number and name (e.g., "P1 High Cohesion", "P3 Depend on Abstractions"). Can be more than one.
   3. **Pattern** — if a Pythonic design pattern would help, name it
   4. **Fix** — show the recommended fix with a code snippet

#### In-Depth Mode

If the user chose in-depth, load the following reference files for deeper analysis with detailed refactoring recipes and pattern guidance:
- `references/design-principles.md`
- `references/code-quality.md`
- `references/error-handling.md`
- `references/pythonic-patterns.md`

For detailed pattern guidance (when explaining HOW to fix), consult the relevant files in `references/patterns/`.

If the user chose standard, use only the checklists above — do not load reference files.

---

### Check Quality

**Trigger:** User asks to "check quality", "check code quality", "run quality check", or similar.

**Input:** A file or directory path (default: current working directory).

Run a focused code quality check against the 17 design rules. This is a lighter, faster check than a full architecture review.

#### Process

1. **Read the code** — Find and read ALL Python files in the target path recursively.

2. **Check the 17 rules** — For each file, check:

   **Naming & Structure:**
   1. No type abuse (don't embed type in name: `user_list` → `users`)
   2. No vague identifiers (`data`, `info`, `temp`, `result`, `handle`)
   3. No single-letter variables outside comprehensions and lambdas
   4. No abbreviations (`cfg` → `config`, `mgr` → `manager`)

   **Functions:**
   5. No flag parameters (boolean args that switch behavior → split into two functions)
   6. No deep nesting (3+ levels → early returns or extract function)
   7. Tell don't ask (don't query state then act on it — tell the object to act)
   8. Functions do one thing (extract if doing two things)

   **Classes:**
   9. No god classes (>200 lines or >5 responsibilities → extract classes)
   10. No parallel data structures (two lists tracking same entities → single dataclass)
   11. Information Expert (methods should live on the class that owns the data)

   **Types & Safety:**
   12. Type hints on all function signatures
   13. Enums for fixed option sets (not bare strings)
   14. No mutable default arguments (`def f(items=[])` → `def f(items=None)`)
   15. No wildcard imports (`from x import *`)

   **Error Handling:**
   16. No broad exception catching (`except Exception` → catch specific exceptions)
   17. Context managers for resources (`with open()` not manual try/finally)

3. **Report findings** — For each violation:
   - **Rule number and name**
   - **File and line**
   - **Fix** — short code snippet showing the correction

   Group by file for easy navigation. Show a summary count at the top:
   ```
   Quality Check: 14 files scanned, 8 issues found
   ─────────────────────────────────────────────────
   Rule 6  (deep nesting):     3 instances
   Rule 12 (missing types):    2 instances
   Rule 16 (broad except):     2 instances
   Rule 5  (flag parameter):   1 instance
   ```

---

### Suggest Patterns

**Trigger:** User asks to "suggest patterns", "recommend patterns", "what patterns should I use", or similar.

**Input:** A file or directory path (default: current working directory).

Scan the code and recommend which of the 11 Pythonic design patterns would improve it.

#### Process

1. **Read the code** — Find and read ALL Python files in the target path recursively.

2. **Match code smells to patterns** — Look for these specific smells:

   | Code Smell | Pattern | Pythonic Implementation |
   |---|---|---|
   | Long if/elif switching behavior | **Strategy** | `Callable` type alias, pass functions as args |
   | Need to create families of related objects | **Abstract Factory** | Tuples of functions + `functools.partial` |
   | Two independent hierarchies that vary | **Bridge** | `Callable` type alias replaces abstract reference |
   | Need undo/batch/queue operations | **Command** | Functions returning undo closures |
   | Side effects mixed with core logic | **Pub/Sub** | Dict-based `subscribe(event, handler)` / `post_event()` |
   | Object creation from config/JSON data | **Registry** | `dict[str, Callable]` mapping + `**kwargs` unpacking |
   | Same algorithm duplicated across classes | **Template Method** | Free function + Protocol parameter |
   | Sequential data transformations | **Pipeline** | `functools.reduce` for composition |
   | Need to react to events without coupling | **Callback** | Function passed as argument |
   | Reusing a function with a different interface | **Function Wrapper** | Wraps another function, translates args |
   | Separating configuration from usage | **Function Builder** | Higher-order function returns configured function |

3. **Report suggestions** — For each match, show:
   - **File and line** — where the smell is
   - **Code smell** — what makes this a candidate
   - **Pattern** — which pattern applies and why
   - **Before** — current code snippet
   - **After** — refactored code using the Pythonic implementation
   - **Trade-off** — is the refactoring worth it here? (sometimes the simple version is fine)

4. **Prioritize** — Order suggestions by impact. A pattern that simplifies 50 lines beats one that saves 3 lines. Note when the current code is simple enough that applying a pattern would be over-engineering.

For full pattern progressions (OOP → functional), consult `references/patterns/`.

---

### Decouple

**Trigger:** User asks to "decouple", "find coupling", "reduce coupling", "improve dependency injection", or similar.

**Input:** A file or directory path (default: current working directory).

Analyze the code for tight coupling and suggest how to decouple using dependency injection.

#### Process

1. **Read the code** — Find and read ALL Python files in the target path recursively. Read every `.py` file. Map imports between modules.

2. **Identify coupling issues** — Look for:
   - **Concrete imports** — modules importing concrete classes instead of depending on abstractions
   - **Global mutable state** — module-level variables shared across files
   - **Content coupling** — accessing private members (`_var`) of other modules
   - **Law of Demeter violations** — chained attribute access (`a.b.c.method()`)
   - **Circular imports** — two modules importing each other
   - **God modules** — one module imported by nearly everything
   - **Missing composition root** — object creation scattered across multiple files instead of one place
   - **Hard-coded dependencies** — `import` inside functions, direct instantiation instead of injection

3. **Generate dependency map** — Show which modules depend on which:
   ```
   Module Dependencies
   ───────────────────
   routes/booking.py → operations/booking.py, models/booking.py
   operations/booking.py → models/booking.py  (no DB import ✓)
   routes/admin.py → db/models.py             ⚠ concrete DB import
   ```

4. **Propose decoupling** — For each issue, show:
   - **File and line**
   - **Principle violated** — which of the 7 design principles (P1–P7)
   - **Current coupling** — what the code does now
   - **Decoupled version** — using Protocol, Callable, or composition root
   - **Before/after code**

5. **Ask before applying** — Confirm which changes to apply before editing any files.

6. **Apply changes** — Edit the files with the approved decoupling.

For guidance on Protocol-based DI, consult `references/layered-architecture.md` and `references/design-principles.md`.

---

### Make Pythonic

**Trigger:** User asks to "make pythonic", "make it pythonic", "refactor to pythonic", "use pythonic patterns", or similar.

**Input:** A file or directory path (default: current working directory).

Analyze code and apply Pythonic pattern replacements.

#### Process

1. **Read the code** — Find and read ALL Python files in the target path recursively. Read every `.py` file.

2. **Identify non-Pythonic patterns** — Look for these specific code smells:

   | Code Smell | Replace With |
   |---|---|
   | ABC/abstract base class hierarchy | Protocol (structural typing) |
   | Single-method abstract class | `Callable` type alias |
   | Deep class inheritance / mixins | Composition with Protocol interface |
   | Wrapper classes for configuration | `functools.partial` |
   | Factory class hierarchies | Closures or tuples of functions + `partial` |
   | Long if/elif switching behavior | Strategy pattern — pass functions as args |
   | if/elif for object creation | Registry pattern — `dict[str, Callable]` mapping |
   | Inline notification side effects | Pub/Sub — `subscribe(event, handler)` / `post_event()` |
   | Duplicated algorithm across classes | Template Method — free function + Protocol parameter |
   | try/finally for resource cleanup | Context managers |
   | Manual `__init__` for data holders | `@dataclass` |
   | Bare string constants for options | `Enum` or `StrEnum` |

3. **Propose changes** — For each finding, show:
   - **File and line**
   - **Current pattern** — what it does now
   - **Pythonic replacement** — what it should become
   - **Before/after code** — complete snippets ready to apply

4. **Ask before applying** — Confirm before editing. Let the user review.

5. **Apply changes** — Edit the files with the approved refactorings.

For detailed pattern guidance, consult the relevant files in `references/patterns/`.

---

### Extract God Class

**Trigger:** User asks to "extract god class", "split large class", "break up class", or similar.

**Input:** A file or directory path (default: current working directory).

Analyze code for god classes and generate the split.

#### Process

1. **Read the code** — Find and read ALL Python files in the target path recursively. Read every `.py` file.

2. **Identify god classes** — A class is a god class if it has any of:
   - More than 200 lines
   - More than 5 distinct responsibilities
   - Methods that cluster into unrelated groups
   - Too many instance variables (>7)
   - Methods that don't use most of the class's state

3. **Analyze responsibilities** — For each god class:
   - Group methods by the data they access and the responsibility they serve
   - Identify natural boundaries between responsibilities
   - Determine which methods belong together
   - Name each responsibility group

4. **Propose extraction** — Show:
   - **Original class** — file, line count, responsibilities found
   - **Proposed split** — one new class per responsibility group with:
     - Class name
     - Methods it takes
     - Its `__init__` parameters
     - Complete code for the new class
   - **Updated original** — the slimmed-down class that composes the extracted ones
   - **Updated callers** — any files that need import changes

5. **Ask before applying** — Confirm the split. The user may want to adjust which methods go where.

6. **Apply changes** — Create the new files and update the original class and its callers.

Principles: P1 High Cohesion, P4 Composition over Inheritance. Consult `references/design-principles.md` for refactoring recipes.

---

### Scaffold API

**Trigger:** User asks to "scaffold API", "scaffold FastAPI", "create a new FastAPI project", "generate project", or similar.

**Input:** A project name.

Generate a new FastAPI project with the three-layer clean architecture structure.

#### Process

1. **Ask for entities** — Ask the user: "What are the main entities for this API? (e.g., users, products, orders)" to determine what to scaffold.

2. **Create project structure** — Generate the full directory tree:

   ```
   {project_name}/
   ├── src/
   │   ├── main.py                  # FastAPI app, include_router, startup
   │   ├── routers/
   │   │   ├── __init__.py
   │   │   └── {entity}.py          # APIRouter, endpoints, composition root
   │   ├── operations/
   │   │   ├── __init__.py
   │   │   └── {entity}.py          # Business logic, accepts DataInterface
   │   ├── db/
   │   │   ├── __init__.py
   │   │   ├── database.py          # Engine, SessionLocal, Base
   │   │   ├── db_interface.py      # Generic DBInterface implementing DataInterface
   │   │   └── models.py            # SQLAlchemy ORM models
   │   └── models/
   │       ├── __init__.py
   │       └── {entity}.py          # Pydantic models + DataInterface Protocol
   ├── tests/
   │   ├── __init__.py
   │   └── test_{entity}.py         # Tests using DataInterfaceStub
   ├── requirements.txt
   └── .gitignore
   ```

3. **Generate files** — For each entity, create:
   - **Pydantic models** — separate Create and Read models + `DataInterface` Protocol
   - **Operations** — business logic functions accepting `data_interface: DataInterface`
   - **Router** — CRUD endpoints acting as composition root, injecting concrete DB
   - **DB model** — SQLAlchemy ORM model
   - **Tests** — using `DataInterfaceStub` with no database dependency

4. **Wire it together** — `main.py` includes all routers and sets up the database.

Consult `references/layered-architecture.md` and `references/testable-api.md` for the full architecture patterns. Use `assets/fastapi-hotel-api/` as a working reference.

---

### Add Endpoint

**Trigger:** User asks to "add endpoint", "add a new endpoint", "add entity", "add a router", or similar.

**Input:** An entity name.

Scaffold a new endpoint across all three layers of the clean architecture.

#### Process

1. **Understand the project** — Read the existing project structure to understand naming conventions, existing patterns, and where files live. Identify the routers/, operations/, db/, and models/ directories.

2. **Ask for details** — Ask the user: "What operations does this endpoint need?" with options: CRUD (all), Read-only, Custom (describe).

3. **Generate across all layers** — Create or update files in each layer:

   **Models layer** (`models/{entity}.py`):
   - `{Entity}Create` — Pydantic model for creation input
   - `{Entity}` — Pydantic model for read output (includes id)
   - `{Entity}Update` — Pydantic model for partial updates (all fields optional)
   - `DataInterface` Protocol if not already defined, or extend existing one

   **Operations layer** (`operations/{entity}.py`):
   - Business logic functions that accept `data_interface: DataInterface`
   - `create_{entity}()`, `get_{entity}()`, `list_{entities}()`, `update_{entity}()`, `delete_{entity}()`
   - No database imports — only depends on the Protocol

   **Database layer** (`db/models.py`):
   - SQLAlchemy ORM model for the entity
   - Extend `DBInterface` if using generic implementation, or add methods to existing DB class

   **Router layer** (`routers/{entity}.py`):
   - `APIRouter` with CRUD endpoints
   - Acts as composition root — instantiates/injects concrete DB into operations
   - Proper HTTP methods and status codes (201 for create, 204 for delete)

   **Tests** (`tests/test_{entity}.py`):
   - Tests using `DataInterfaceStub` — no database needed
   - Cover create, read, update, delete, and not-found cases

4. **Update main.py** — Add `include_router()` for the new router.

Follow existing project conventions for naming, imports, and style. Consult `references/layered-architecture.md` for the layer patterns.

---

## Additional Resources

### Reference Files

For detailed guidance beyond this overview, consult:

**Architecture & Design:**
- **`references/design-principles.md`** — Full treatment of the seven design principles with refactoring recipes and code examples
- **`references/layered-architecture.md`** — Detailed three-layer architecture guide: DataInterface, DBInterface, router composition, Pydantic models, to_dict utility
- **`references/testable-api.md`** — Testing strategy: stub-based testing, DataInterfaceStub, test isolation, no-database testing

**Python Fundamentals:**
- **`references/classes-and-dataclasses.md`** — When to use classes vs dataclasses, @dataclass, field(), frozen, encapsulation
- **`references/function-design.md`** — Pure functions, higher-order functions, closures, functools.partial, classes vs functions vs modules
- **`references/data-structures.md`** — Choosing list vs dict vs tuple vs set, enums, performance trade-offs
- **`references/types-and-type-hints.md`** — Python's type system, Callable types, nominal vs structural typing, best practices
- **`references/error-handling.md`** — Custom exceptions, context managers, error handling layers, anti-patterns
- **`references/code-quality.md`** — 17 code quality rules: naming, nesting, flags, type abuse, and code review checklist
- **`references/project-organization.md`** — Modules, packages, imports, folder structure, avoid "utils" anti-pattern

**Pythonic Patterns:**
- **`references/pythonic-patterns.md`** — Quick reference lookup table for all 10 patterns (use for reviews and pattern selection)

**Pythonic Patterns (full progressions from OOP → functional):**
- **`references/patterns/strategy.md`** — Callable type alias, closures, functools.partial
- **`references/patterns/abstract-factory.md`** — Tuples of functions, partial, builder functions
- **`references/patterns/bridge.md`** — Bound methods as callables, when to stop going functional
- **`references/patterns/command.md`** — Functions returning undo closures, batch via list comprehension
- **`references/patterns/notification.md`** — Observer, Mediator, and Pub/Sub with dict-based subscribe/post_event
- **`references/patterns/registry.md`** — Dict mapping + **kwargs unpacking, self-registering plugins via importlib
- **`references/patterns/template-method.md`** — Free function + Protocol parameters, protocol segregation
- **`references/patterns/pipeline.md`** — Chain of Responsibility, functools.reduce composition, pandas .pipe()
- **`references/patterns/functional.md`** — Callback, Function Wrapper, Function Builder patterns

### Example Files

- **`assets/fastapi-hotel-api/`** — Complete working FastAPI project demonstrating all patterns: rooms, customers, bookings with computed prices
