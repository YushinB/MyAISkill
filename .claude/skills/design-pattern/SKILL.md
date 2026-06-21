---
name: design-pattern
description: Identify, explain, and apply software design patterns and architecture patterns. Trigger with "which pattern should I use", "design pattern for", "implement a factory/observer/strategy/decorator", "refactor with a pattern", "what architecture fits", "microservices vs monolith", "explain singleton/adapter/command", or any question about GoF patterns or modern architecture styles.
argument-hint: "<problem description, code snippet, or pattern name>"
---

# /design-pattern

Diagnose the design problem, recommend the right pattern, and show how to apply it — covering all 23 GoF patterns and 6 modern architecture styles.

## Usage

```
/design-pattern <problem, code snippet, or pattern name>
```

## How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                     DESIGN PATTERN                               │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 1 — DIAGNOSE                                              │
│  ✓ Identify the category of problem (creation / structure /      │
│    behavior / architecture)                                      │
│  ✓ Surface constraints and tradeoffs                             │
│  ✓ Warn when a pattern would over-engineer the solution          │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 2 — RECOMMEND                                             │
│  ✓ Name the pattern(s) that fit                                  │
│  ✓ Explain why this pattern, not another                         │
│  ✓ List the key participants and their roles                     │
├──────────────────────────────────────────────────────────────────┤
│  PHASE 3 — IMPLEMENT                                             │
│  ✓ Show structure (UML-style or code skeleton)                   │
│  ✓ Provide concrete before/after example                         │
│  ✓ Flag known pitfalls and when NOT to use it                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## Part I — GoF Design Patterns

Design patterns are typical solutions to recurring problems in software design — blueprints to customize, not copy verbatim.

### Creational Patterns
*Provide object creation mechanisms that increase flexibility and code reuse.*

#### Factory Method
- **Intent:** Interface for creating objects in a superclass; subclasses alter the type created.
- **Use when:** Exact type of object isn't known upfront; you want to provide extension points for components; or you need to reuse/pool existing objects.
- **Participants:** `Product` (interface) · `ConcreteProduct` · `Creator` (declares factory method) · `ConcreteCreator` (overrides it)
- **Pitfall:** Can require subclassing just to change product type — consider Abstract Factory if product families grow.

#### Abstract Factory
- **Intent:** Produce families of related objects without specifying their concrete classes.
- **Use when:** Code must work with multiple families of related products; concrete classes shouldn't be dependencies; a class has multiple Factory Methods blurring its main responsibility.
- **Participants:** `AbstractFactory` · `ConcreteFactory` · `AbstractProduct` · `ConcreteProduct` · `Client`
- **Pitfall:** Adding a new product type requires changing every factory — plan the family interface upfront.

#### Builder
- **Intent:** Construct complex objects step by step; same process builds different representations.
- **Use when:** Constructor has too many optional parameters ("telescopic constructor"); code must create different representations of the same product; need to build complex object trees (e.g., AST, HTML).
- **Participants:** `Builder` (interface) · `ConcreteBuilder` · `Product` · `Director` (defines build order) · `Client`
- **Pitfall:** Overkill for simple objects with 2–3 fields.

#### Prototype
- **Intent:** Clone existing objects without depending on their concrete class.
- **Use when:** Object creation is expensive and a copy is cheaper; you want to reduce subclass proliferation where subclasses differ only in initialization.
- **Participants:** `Prototype` (interface with `clone()`) · `ConcretePrototype` · `PrototypeRegistry` (optional) · `Client`
- **Pitfall:** Deep vs. shallow copy — objects containing references must handle inner-object cloning explicitly.

#### Singleton
- **Intent:** Ensure one instance of a class; provide a global access point.
- **Use when:** Shared resource (DB connection pool, config, logger) must have exactly one instance; you need stricter control than global variables.
- **Participants:** `Singleton` (private constructor + static `getInstance()`)
- **Pitfall:** Global state makes unit testing hard; consider dependency injection instead for testability.

---

### Structural Patterns
*Explain how to assemble objects and classes into larger structures while keeping them flexible.*

#### Adapter
- **Intent:** Allow objects with incompatible interfaces to collaborate.
- **Use when:** You want to use a class whose interface doesn't match what you need; you're reusing legacy code or third-party libraries.
- **Participants:** `Client` · `ClientInterface` · `Service` (incompatible) · `Adapter` (wraps Service, implements ClientInterface)
- **Pitfall:** If both interfaces are too different, the adapter becomes a façade of hacks — consider rewriting.

#### Bridge
- **Intent:** Split a large class into Abstraction and Implementation hierarchies developed independently.
- **Use when:** A class has multiple orthogonal dimensions of variation (e.g., shape × color); you want to switch implementations at runtime; you're avoiding a Cartesian-product explosion of subclasses.
- **Participants:** `Abstraction` · `RefinedAbstraction` · `Implementation` (interface) · `ConcreteImplementation` · `Client`
- **Pitfall:** Adds indirection — only worth it when you have two genuinely independent dimensions.

#### Composite
- **Intent:** Compose objects into tree structures; treat individual objects and compositions uniformly.
- **Use when:** You need a tree-like object structure (e.g., file system, UI component tree, org chart); client code should treat leaf and container nodes the same way.
- **Participants:** `Component` (common interface) · `Leaf` · `Container` (holds children, delegates) · `Client`
- **Pitfall:** Makes it hard to restrict the type of components inside a container — may need runtime checks.

#### Decorator
- **Intent:** Attach new behaviors to objects dynamically by wrapping them.
- **Use when:** You need to assign extra behaviors at runtime without breaking existing code; extending via inheritance is impractical (sealed class, too many combinations).
- **Participants:** `Component` (interface) · `ConcreteComponent` · `BaseDecorator` · `ConcreteDecorator` · `Client`
- **Pitfall:** Stack of many small decorators can be hard to debug; ordering matters.

#### Facade
- **Intent:** Provide a simplified interface to a complex subsystem.
- **Use when:** You want a limited, clean entry point into a complex library or framework; you're layering a subsystem and want to define entry points between layers.
- **Participants:** `Facade` · `AdditionalFacade` (optional) · `Complex Subsystem classes` · `Client`
- **Pitfall:** A "god object" façade that does too much becomes a bottleneck — keep it thin.

#### Flyweight
- **Intent:** Share common parts of state across many fine-grained objects to save RAM.
- **Use when:** Application must support a huge number of similar objects that barely fit in RAM; the majority of each object's state can be made extrinsic (passed in at call time).
- **Participants:** `Flyweight` (intrinsic/shared state) · `Context` (extrinsic/unique state) · `FlyweightFactory` · `Client`
- **Pitfall:** Trades RAM for CPU — calculating extrinsic state on every call can hurt performance.

#### Proxy
- **Intent:** Provide a substitute that controls access to another object.
- **Use when:** Lazy initialization (virtual proxy); access control (protection proxy); local execution of remote service (remote proxy); logging, caching results, or reference counting (smart proxy).
- **Participants:** `ServiceInterface` · `Service` (real logic) · `Proxy` · `Client`
- **Pitfall:** Adding many proxy layers increases latency and complicates the call stack.

---

### Behavioral Patterns
*Concerned with algorithms and the assignment of responsibilities between objects.*

#### Chain of Responsibility
- **Intent:** Pass requests along a chain of handlers; each decides to process or forward.
- **Use when:** Handlers and their order are unknown at compile time; you want to decouple the sender from receivers; you need to process requests in a specific order.
- **Participants:** `Handler` (interface) · `BaseHandler` (optional) · `ConcreteHandlers` · `Client`
- **Pitfall:** If no handler processes the request, it silently falls through — ensure a default handler or explicit failure.

#### Command
- **Intent:** Turn a request into a stand-alone object; parameterize, queue, and undo/redo operations.
- **Use when:** You need to queue or schedule operations; you need reversible operations (undo/redo); you want to parameterize objects with operations.
- **Participants:** `Sender` (invoker) · `Command` (interface) · `ConcreteCommand` · `Receiver` · `Client`
- **Pitfall:** Can produce many small Command classes — consider lambdas for simple one-liners.

#### Iterator
- **Intent:** Traverse elements of a collection without exposing its underlying representation.
- **Use when:** The collection has complex traversal logic you want to hide from clients; you need multiple simultaneous traversal modes; you want a uniform traversal interface across different collection types.
- **Participants:** `Iterator` (interface) · `ConcreteIterator` · `Collection` (interface) · `ConcreteCollection` · `Client`
- **Pitfall:** Custom iterators can be a premature abstraction — use language-built-in iterables where possible.

#### Mediator
- **Intent:** Restrict direct communication between objects; force collaboration via a mediator.
- **Use when:** Classes are tightly coupled and hard to change; a component is too dependent to be reused in isolation; you want to avoid creating many subclasses just to reuse basic behavior.
- **Participants:** `Components` · `Mediator` (interface) · `ConcreteMediator`
- **Pitfall:** The mediator can become a "god object" — keep it focused on coordination, not logic.

#### Memento
- **Intent:** Save and restore an object's previous state without exposing its internals.
- **Use when:** You need snapshots for undo/redo; accessing fields directly would violate encapsulation.
- **Participants:** `Originator` (state owner, creates/restores mementos) · `Memento` (snapshot) · `Caretaker` (history manager)
- **Pitfall:** Storing many large mementos can consume significant memory — consider incremental snapshots.

#### Observer
- **Intent:** Define a subscription mechanism to notify multiple objects about events.
- **Use when:** Changes to one object require updating others whose set is unknown or dynamic; objects must observe each other for a limited or unknown time.
- **Participants:** `Publisher` (subject) · `Subscriber` (interface) · `ConcreteSubscriber` · `Client`
- **Pitfall:** Subscribers that hold references to publishers can cause memory leaks — use weak references or explicit unsubscribe.

#### State
- **Intent:** Let an object alter its behavior when its state changes — appears to change its class.
- **Use when:** Behavior depends heavily on current state and state-specific code changes frequently; you want to clean up massive conditional blocks scattered across methods.
- **Participants:** `Context` · `State` (interface) · `ConcreteStates`
- **Pitfall:** If states are few and stable, simple conditionals are cleaner — don't introduce State just to eliminate one `if`.

#### Strategy
- **Intent:** Define a family of algorithms; make them interchangeable; vary them independently of clients.
- **Use when:** You need to switch algorithms at runtime; you want to isolate business logic from algorithm implementation details; multiple classes differ only in their behavior.
- **Participants:** `Context` · `Strategy` (interface) · `ConcreteStrategies` · `Client`
- **Pitfall:** If the algorithm family rarely changes or has only one variant, lambdas/functions are simpler.

#### Template Method
- **Intent:** Define an algorithm's skeleton in a superclass; let subclasses fill in specific steps.
- **Use when:** Clients need to extend only particular steps of an algorithm, not its structure; multiple classes contain near-identical algorithms with minor differences.
- **Participants:** `AbstractClass` (template method + abstract steps) · `ConcreteClasses`
- **Pitfall:** Inverts control — subclasses are constrained to the parent's structure; can lead to fragile base class problems.

#### Visitor
- **Intent:** Separate algorithms from the objects they operate on.
- **Use when:** You need to perform many distinct, unrelated operations on a complex object structure (e.g., AST); you want to keep auxiliary behavior out of primary business logic.
- **Participants:** `Visitor` (interface) · `ConcreteVisitors` · `Element` (interface with `accept(visitor)`) · `ConcreteElements` · `Client`
- **Pitfall:** Adding a new element type requires updating every concrete visitor — bad for frequently changing hierarchies.

---

## Part II — Modern Architecture Patterns

*Macro-level patterns for structuring entire systems.*

### Pattern Comparison

| Pattern | Best For | Scalability | Complexity | Cost |
|---------|----------|-------------|------------|------|
| Monolithic | Small/medium apps, MVP | Low | Low | Low |
| Layered (N-Tier) | Traditional enterprise | Low–Medium | Low–Medium | Low–Medium |
| Serverless | Event-driven, variable load | Very High (auto) | Medium | Pay-per-use |
| Event-Driven | Real-time, async systems | High | High | Medium–High |
| Microservices | Large systems, multiple teams | Very High | Very High | High |
| Domain-Driven (DDD) | Complex business domains | Medium–High | High | Medium |

### Decision Guidelines
- **Start Monolithic** — for new projects, small teams, uncertain requirements, or MVP. Fastest path to production.
- **Choose Layered** — for traditional enterprise apps needing clear separation of concerns.
- **Use Serverless** — for event-driven workloads, variable traffic, background tasks, or when minimizing operational overhead.
- **Adopt Event-Driven** — when building real-time systems, requiring loose coupling, high scalability, or async processing.
- **Go Microservices** — when you have multiple teams, need independent service scaling, or are building large-scale distributed systems.
- **Apply DDD** — for complex business domains with intricate rules, long-term projects, or when close collaboration with domain experts is possible.

### Monolithic
- **Intent:** Single self-contained unit — all components share one codebase, one DB, deployed together.
- **Strengths:** Simple development, testing, and deployment; no network overhead for internal calls.
- **Weaknesses:** Scaling requires replicating the whole app; tech stack lock-in; grows harder to maintain over time.

### Layered (N-Tier)
- **Intent:** Horizontal layers (Presentation → Business Logic → Data Access → Database), each communicating only with the adjacent layer.
- **Strengths:** Clear separation of concerns; easy to understand; well-established for enterprise apps.
- **Weaknesses:** Performance overhead from passing through all layers; can become monolithic over time.

### Serverless
- **Intent:** Functions triggered by events, executed on-demand by a cloud provider — no infrastructure management.
- **Strengths:** Auto-scales to zero; pay only for execution; fast time-to-market.
- **Weaknesses:** Cold start latency; vendor lock-in; debugging distributed functions is harder.

### Event-Driven
- **Intent:** Components communicate via events (state changes); producers and consumers are decoupled.
- **Strengths:** Loose coupling; high scalability; real-time responsiveness; fault tolerance via event replay.
- **Weaknesses:** Complex debugging; eventual consistency; event ordering and duplicate handling.

### Microservices
- **Intent:** Application as a collection of small, independent services organized around business capabilities — each owns its DB, deployed independently.
- **Strengths:** Independent scaling; technology diversity; fault isolation; team autonomy.
- **Weaknesses:** Distributed system complexity; network latency; data consistency challenges; high operational overhead.

### Domain-Driven Design (DDD)
- **Intent:** Model software around the core business domain — ubiquitous language, bounded contexts, aggregates, entities, value objects.
- **Strengths:** Business-aligned code; maintainable as requirements evolve; clear boundaries reduce coupling.
- **Weaknesses:** Steep learning curve; requires domain expert collaboration; overkill for simple CRUD apps.

---

## Output Format

```markdown
## Design Pattern Recommendation: [problem summary]

### Problem Diagnosis
[What kind of design problem this is and which category it falls into]

### Recommended Pattern: [Pattern Name]
**Category:** Creational / Structural / Behavioral / Architecture
**Why this pattern:** [1–2 sentences linking the problem to the pattern's intent]

### Participants & Roles
| Role | Responsibility |
|------|---------------|
| [Name] | [What it does] |

### Structure
[Code skeleton or UML-style pseudocode]

### Before / After
[Concrete example showing the improvement]

### When NOT to use this
[Honest tradeoffs and simpler alternatives]
```

---

## Quick Reference

| If you need to... | Consider... |
|-------------------|-------------|
| Create objects without knowing their exact class | Factory Method, Abstract Factory |
| Build complex objects step by step | Builder |
| Clone objects cheaply | Prototype |
| Guarantee one instance | Singleton |
| Wrap an incompatible interface | Adapter |
| Separate abstraction from implementation | Bridge |
| Treat leaves and containers uniformly | Composite |
| Add behavior at runtime | Decorator |
| Simplify a complex subsystem | Facade |
| Share state across many objects | Flyweight |
| Control access to an object | Proxy |
| Pass a request through a chain | Chain of Responsibility |
| Queue, undo, or log operations | Command |
| Traverse a collection | Iterator |
| Reduce class coupling via a hub | Mediator |
| Save/restore object state | Memento |
| Notify dependents of state changes | Observer |
| Change behavior based on state | State |
| Swap algorithms at runtime | Strategy |
| Define an algorithm skeleton | Template Method |
| Add operations without modifying classes | Visitor |
