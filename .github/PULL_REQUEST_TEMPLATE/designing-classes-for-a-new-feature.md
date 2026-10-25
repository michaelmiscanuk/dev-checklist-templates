## Designing classes for a new feature

When: You are about to write the classes for a new feature and want them to hold up once other code depends on them.

Before you start: Will instances be map or set keys, or compared by value? (any that apply) Yes, keys or value comparison / None of these. Checks marked "only if" apply only to that answer.
Before you start: Will instances be shared between threads? (any that apply) Yes, across threads / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write any class

You know what the feature does and are naming the things it handles.

Must:
- [ ] Decide value or entity first
- [ ] Inherit only for true is-a

Stop if:
- If a class seems to cross a module or service boundary: ask your tech lead first

Move on when each class has a one-sentence job and is marked value or entity.

### 2. While shaping each class (every time)

Fields and methods are taking shape inside one class.

Must:
- [ ] Make every field private
- [ ] Copy collections in and out

Move on when every field is private and every public method keeps the class's invariants true.

### 3. When writing the constructors

You are deciding how each object comes into being.

Must:
- [ ] Reject invalid arguments up front
- [ ] Avoid overridable calls in constructors

Stop if:
- If a constructor needs many collaborators: ask a reviewer whether the class does too much

Move on when no constructor can produce an invalid object or run subclass code early.

### 4. When instances are compared or shared

Objects will be map keys, compared by value, or used from more than one thread.

Must:
- [ ] Define equality and hashing together (only if: Yes, keys or value comparison)
- [ ] State each class's thread safety (only if: Yes, across threads)

Stop if:
- If several shared objects must change together atomically: ask a concurrency-experienced reviewer before merging

Move on when equality and hashing are tested together and every shared class states its thread safety.

### 5. Before other code depends on it

The classes work and the feature is close to review.

Must:
- [ ] Seal classes not built for extension

Stop if:
- If a public method others already call must change: stop and agree it with those callers' owners

Move on when tests pass and a reviewer has read the public API from the caller's side.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/designing-classes-for-a-new-feature.md). Interactive version: [Designing classes for a new feature on CheatGrid](https://www.cheatgrid.com/checklists/computer-science-fundamentals/designing-classes-for-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=designing-classes-for-a-new-feature).
