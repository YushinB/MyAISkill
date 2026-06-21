---
name: coding-refactor
description: Structured code refactoring session — identify code smells, apply targeted refactoring techniques, and improve code quality without changing behavior. Trigger with "refactor this", "this code smells", "clean up this class/method", "too many parameters", "duplicate code", or when code has grown hard to read, change, or maintain.
argument-hint: "<file path, code snippet, or description of the problem>"
---

# /coding-refactor

Improve existing code structure without changing its behavior. Diagnose code smells, prescribe the right refactoring techniques, and apply them step by step.

## Usage

```
/coding-refactor <file path, code snippet, or description>
```

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    CODING REFACTOR                              │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 1 — DIAGNOSE                                             │
│  ✓ Identify code smells by category                            │
│  ✓ Assess severity and priority                                 │
│  ✓ Map smells to applicable refactoring techniques             │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 2 — PRESCRIBE                                            │
│  ✓ Recommend specific refactoring techniques                    │
│  ✓ Explain why each technique applies                           │
│  ✓ Flag which changes are safe vs. risky                        │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 3 — APPLY                                                │
│  ✓ Refactor step by step (one smell at a time)                  │
│  ✓ Show before/after for each change                            │
│  ✓ Verify behavior is preserved (no new functionality added)    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Code Smell Reference

### Bloaters — Code that has grown too large
| Smell | Sign | Technique |
|-------|------|-----------|
| Long Method | > 10 lines; hard to name | Extract Method |
| Large Class | Too many fields/methods | Extract Class, Extract Subclass |
| Long Parameter List | > 3–4 params | Introduce Parameter Object, Preserve Whole Object |
| Primitive Obsession | Using primitives for domain concepts | Replace Data Value with Object |
| Data Clumps | Same group of variables appearing together | Extract Class, Introduce Parameter Object |

### Object-Orientation Abusers — Misuse of OOP
| Smell | Sign | Technique |
|-------|------|-----------|
| Switch Statements | Complex switch/if chains on type | Replace Conditional with Polymorphism |
| Temporary Field | Fields used only in some scenarios | Extract Class, Introduce Null Object |
| Refused Bequest | Subclass ignores inherited members | Replace Inheritance with Delegation, Extract Superclass |
| Alternative Classes w/ Different Interfaces | Two classes do the same thing, named differently | Rename Method, Move Method |

### Change Preventers — Code that resists modification
| Smell | Sign | Technique |
|-------|------|-----------|
| Divergent Change | One class changed for many unrelated reasons | Extract Class |
| Shotgun Surgery | One change requires edits across many classes | Move Method, Move Field, Inline Class |
| Parallel Inheritance Hierarchies | Adding a subclass requires adding another elsewhere | Move Method, Move Field |

### Dispensables — Unnecessary code
| Smell | Sign | Technique |
|-------|------|-----------|
| Comments | Code needs comments to be understood | Extract Method, Rename Method |
| Duplicate Code | Identical or near-identical fragments | Extract Method, Pull Up Method, Template Method |
| Lazy Class | Class does too little to justify its existence | Inline Class, Collapse Hierarchy |
| Dead Code | Unused variables, methods, classes | Delete it |
| Speculative Generality | "We might need this someday" abstractions | Inline Class, Collapse Hierarchy, Remove Parameter |
| Data Class | Class with only getters/setters, no behavior | Move Method, Extract Method |

### Couplers — Excessive dependency between classes
| Smell | Sign | Technique |
|-------|------|-----------|
| Feature Envy | Method uses another class's data more than its own | Move Method, Extract Method |
| Inappropriate Intimacy | Classes reach into each other's internals | Move Method, Move Field, Hide Delegate |
| Message Chains | `a.b().c().d()` style navigation | Hide Delegate, Extract Method |
| Middle Man | Class only delegates — does nothing itself | Inline Class, Replace Delegation with Inheritance |

---

## Refactoring Techniques

### Composing Methods
- **Extract Method** — Pull a code fragment into a named method
- **Inline Method** — Replace a trivial method call with its body
- **Extract Variable** — Name a complex expression
- **Replace Temp with Query** — Replace local variable with a method call
- **Split Temporary Variable** — One variable = one responsibility

### Moving Features Between Objects
- **Move Method** — Method belongs in the class that uses it most
- **Move Field** — Field belongs with the methods that use it
- **Extract Class** — Class doing two things becomes two classes
- **Inline Class** — Merge a class that no longer justifies its existence
- **Hide Delegate** — Don't expose internal objects to clients

### Organizing Data
- **Replace Data Value with Object** — Promote a primitive to a class
- **Replace Magic Number with Symbolic Constant** — Named constants over bare numbers
- **Encapsulate Field** — Make fields private; expose via accessors
- **Replace Type Code with Subclasses / State / Strategy** — Polymorphism over integer codes

### Simplifying Conditionals
- **Decompose Conditional** — Extract condition and branches into named methods
- **Consolidate Conditional Expression** — Merge conditions with the same outcome
- **Replace Nested Conditional with Guard Clauses** — Early returns flatten nesting
- **Replace Conditional with Polymorphism** — Let subclasses handle variants
- **Introduce Null Object** — Eliminate null checks with a do-nothing object

### Simplifying Method Calls
- **Rename Method** — Name should reveal intent
- **Add / Remove Parameter** — Adjust method signature
- **Separate Query from Modifier** — Methods either return data or change state, not both (CQRS)
- **Preserve Whole Object** — Pass the object, not its extracted values
- **Introduce Parameter Object** — Replace a repeating parameter group with an object
- **Replace Constructor with Factory Method** — More expressive object creation
- **Replace Error Code with Exception** — Communicate failures via exceptions

### Dealing with Generalization
- **Pull Up Field / Method** — Move shared members to superclass
- **Push Down Field / Method** — Move specialized members to subclass
- **Extract Superclass / Interface** — Formalize shared behavior
- **Collapse Hierarchy** — Merge superclass and subclass when the distinction is gone
- **Replace Inheritance with Delegation** — Subclass uses only part of superclass
- **Replace Delegation with Inheritance** — Simple delegation that could just be inheritance

---

## Design Principles to Apply

| Principle | What it means in practice |
|-----------|--------------------------|
| **SRP** — Single Responsibility | One class/method = one reason to change |
| **OCP** — Open/Closed | Extend via new classes, not by modifying existing ones |
| **LSP** — Liskov Substitution | Subclasses must be substitutable for their parent |
| **Encapsulation** | Hide internal state; expose behavior |
| **Tell-Don't-Ask** | Tell an object what to do; don't ask for its state and act on it |
| **CQRS** | Separate queries (return data) from modifiers (change state) |

---

## Output Format

```markdown
## Refactoring Report: [file or description]

### Smells Found
| Smell | Location | Severity | Category |
|-------|----------|----------|----------|
| Long Method | processOrder() | 🔴 High | Bloater |
| Duplicate Code | calculateTax() / calculateDiscount() | 🟡 Medium | Dispensable |

### Refactoring Plan
1. **Extract Method** from `processOrder()` → `validateOrder()`, `applyDiscount()`
2. **Extract Method** for shared tax/discount logic → `applyRate(amount, rate)`
3. **Replace Magic Number** `0.15` → `TAX_RATE`

### Refactored Code
[before/after for each change]

### Principles Applied
- SRP: `processOrder` no longer handles validation
- Tell-Don't-Ask: moved condition logic into Order class
```

---

## Rules for Safe Refactoring

1. **Never add new functionality** while refactoring — one concern at a time.
2. **Run tests before and after** each step.
3. **Refactor in small steps** — one technique per commit if possible.
4. **Don't touch adjacent code** that isn't broken.
5. **Name things by their intent**, not their implementation.
6. If you notice unrelated dead code, **mention it — don't delete it** unless asked.
