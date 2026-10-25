# Designing classes for a new feature

When: You are about to write the classes for a new feature and want them to hold up once other code depends on them.

How to use: Open this when you are about to write the classes for a new feature, and keep it open until the code is reviewed. Answer the two questions first; they switch on the equality and threading checks. Tick a check when its 'How to verify' line shows what it describes. Musts come first in each moment.

Before you start: Will instances be map or set keys, or compared by value? (any that apply) Yes, keys or value comparison / None of these. Checks marked "only if" apply only to that answer.
Before you start: Will instances be shared between threads? (any that apply) Yes, across threads / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write any class

You know what the feature does and are naming the things it handles.

Must:
- [ ] Decide value or entity first (source 1)
  Why: A **value object** is defined by its contents, like money; an entity keeps its identity as fields change, like an account.
  How to verify: Each planned class carries a note: value (equal by fields, usually unchangeable) or entity (equal by identity or ID).
- [ ] Inherit only for true is-a
  Why: Hold the other object as a field and forward only the calls you need; this is **composition**.
  How to verify: Each planned subclass has a one-line note saying it is a kind of its parent, so every public parent method applies to it.

Don't forget these too:
- [ ] Give each class one job (source 2)
  Why: Describe it in one sentence without 'and'; the **Single Responsibility Principle** allows each class one reason to change.
  How to verify: Read each class's one-line description: it names one responsibility and one kind of change that would touch it.
- [ ] Name classes after domain things
  Why: Use the words the spec and its users use; names like Manager, Helper or Util hide a vague job.
  How to verify: Read the class list to someone who knows the feature: they recognise every name without explanation.
- [ ] Sketch the calling code first
  Why: A few lines written from the caller's side show which methods the class really needs, and nothing more.
  How to verify: The sketch exists, and the class's planned public methods are exactly the ones it calls.
- [ ] Make classes unchangeable by default
  Why: Set fields once in the constructor and add mutation only where the feature needs state to change over time.
  How to verify: Count setters and reassignable fields: each one has a reason tied to the feature's behaviour.
- [ ] Put interfaces at real seams
  Why: Add a small interface where you expect several implementations or need a test stand-in, not on every class.
  How to verify: Each interface has two or more implementations, or a named test double, or a stated planned second use.
- [ ] Reuse what the codebase already has
  Why: Search for existing classes that cover part of the job, and call or extend them before writing your own version.
  How to verify: Search the codebase for the feature's key nouns: each match is either reused or has a noted reason it does not fit.
- [ ] Follow the codebase's existing conventions (source 3)
  Why: Copy how neighbouring code names, structures and wires its classes; consistency wins unless it would **worsen code health**.
  How to verify: Set a new class beside a similar existing one: naming, layout and dependency style match.
- [ ] Keep business rules apart from infrastructure (source 4)
  Why: Domain classes hold the rules. Database, HTTP, file and UI code sit in separate classes, and **dependencies point inward** toward the rules.
  How to verify: Run the domain classes' tests with no database, web server or UI available: they all pass.
- [ ] Pick one error strategy
  Why: Choose exceptions, result types or error codes for the whole feature, matching what the surrounding code already uses.
  How to verify: Read the public methods' failure paths: they all report errors the same way.

Stop if:
- If a class seems to cross a module or service boundary: ask your tech lead first

Move on when each class has a one-sentence job and is marked value or entity.

## 2. While shaping each class (every time)

Fields and methods are taking shape inside one class.

Must:
- [ ] Make every field private (source 5)
  Why: Expose behaviour, not data, and give each member the **lowest access level** that still lets the software work.
  How to verify: Scan the field declarations: none are public or widely visible without a written reason.
- [ ] Copy collections in and out
  Why: Store a copy of a list passed in, and return a **read-only view** or a copy, never the internal list itself.
  How to verify: Change a list after passing it in, and one returned by a getter: the object's state stays the same.

Don't forget these too:
- [ ] Write down each class's invariants (source 6)
  Why: An **invariant** is a rule that always holds, like end after start; every public method must leave it true.
  How to verify: Each invariant appears in a comment or assertion, and every method that changes state preserves it.
- [ ] Put behaviour next to its data (source 7)
  Why: Tell the object what to do instead of pulling its fields out; logic kept in services leaves an **anemic domain model**.
  How to verify: Look for callers that read two or more fields and compute with them: that logic now lives in the class.
- [ ] Keep the public surface small
  Why: Start every method private; each public one is a promise other code will hold you to.
  How to verify: List the public methods: each is used by the feature or by a caller you can name.
- [ ] Use small types, not bare primitives (source 8)
  Why: These **domain primitives**, like EmailAddress or Money, validate once at creation and stop callers swapping two plain strings or numbers.
  How to verify: Read the method signatures: no run of same-typed parameters a caller could swap without the compiler noticing.
- [ ] Make absent values explicit
  Why: Return an empty collection or an optional type instead of null, so callers cannot forget the empty case.
  How to verify: Read the getters and finders: none return null to mean nothing found.
- [ ] Replace boolean flag parameters (source 9)
  Why: Split one method into two clearly named methods, or take a named option, so every call reads on its own.
  How to verify: Read the public method signatures: none takes a boolean that switches what the method does.
- [ ] Let failures surface, never swallow them (source 10)
  Why: Catch an error only where you can fully handle it; otherwise pass it on with its context.
  How to verify: Read every catch block: none returns a default value or logs and carries on without handling the failure.

Move on when every field is private and every public method keeps the class's invariants true.

## 3. When writing the constructors

You are deciding how each object comes into being.

Must:
- [ ] Reject invalid arguments up front (source 11)
  Why: Take every required part as an argument and validate it before assigning fields, so a half-valid object never exists.
  How to verify: Construct with missing, empty and out-of-range values: each fails at once with an error naming the argument.
- [ ] Avoid overridable calls in constructors (source 12)
  Why: An override runs before the subclass sets its own fields, so it sees null; call only private, final or static methods.
  How to verify: Read each constructor: every method it calls cannot be overridden by a subclass.

Don't forget these too:
- [ ] Register listeners after construction
  Why: Register from a **factory method** once the object is fully built, so nobody sees it half-built.
  How to verify: In every constructor, the object is not passed to any listener, registry or thread.
- [ ] Pass dependencies into the constructor
  Why: Take collaborators like clocks, repositories and clients as parameters, so tests can swap in a fixed clock or a fake.
  How to verify: The class body creates no service-like objects and reads no globals; they all arrive as constructor parameters.
- [ ] Use a builder for many parameters
  Why: Once there are several optional or same-typed parameters, a builder or parameter object stops callers mixing them up.
  How to verify: No constructor has a long run of parameters, and none has adjacent ones of the same type and similar meaning.
- [ ] Name each way of building
  Why: A static method like fromCents or forGuest tells the reader what each variant means, which overloaded constructors cannot.
  How to verify: Where a class has two or more creation paths, each has a named factory method.

Stop if:
- If a constructor needs many collaborators: ask a reviewer whether the class does too much

Move on when no constructor can produce an invalid object or run subclass code early.

## 4. When instances are compared or shared

Objects will be map keys, compared by value, or used from more than one thread.

Must:
- [ ] Define equality and hashing together (only if: Yes, keys or value comparison)
  Why: Use the same fields in both: **equal objects** must produce equal hash codes.
  How to verify: Build two equal instances separately and put one in a hash set: the set reports it contains the other.
- [ ] State each class's thread safety (only if: Yes, across threads)
  Why: Pick one: unchangeable, internally locked, or confined to one thread, and say so on the class.
  How to verify: Each shared class has a doc line saying unchangeable, thread-safe, or not thread-safe.

Don't forget these too:
- [ ] Keep hashed fields unchangeable (only if: Yes, keys or value comparison)
  Why: An object whose hashed fields change after insertion gets lost inside its map or set.
  How to verify: Every field used in equality and hashing is set once and never reassigned.
- [ ] Settle equality across subclasses (only if: Yes, keys or value comparison)
  Why: Subclasses that add fields break symmetric equality; make value classes final or compare the exact class.
  How to verify: Compare a parent and a subclass instance both ways: the answers agree, or the class cannot be subclassed.
- [ ] Keep ordering consistent with equality (source 13) (only if: Yes, keys or value comparison)
  Why: Sorted sets use the comparison, not equality: add two unequal items that compare as zero and the second add returns false.
  How to verify: For sample pairs, comparison returns zero exactly when equality returns true.
- [ ] Share only unchangeable objects freely (only if: Yes, across threads)
  Why: Fields set once, no setters and copied collections make an object safe to pass between threads without locks.
  How to verify: Objects handed across threads are unchangeable, or their class documents its locking.

Stop if:
- If several shared objects must change together atomically: ask a concurrency-experienced reviewer before merging

Move on when equality and hashing are tested together and every shared class states its thread safety.

## 5. Before other code depends on it

The classes work and the feature is close to review.

Must:
- [ ] Seal classes not built for extension
  Why: Document the methods subclasses may override, and how, before opening a class.
  How to verify: Each class open to subclassing documents its extension points; every other class is final or sealed.

Don't forget these too:
- [ ] Test through the public methods
  Why: Tests that use only the public surface let you change internals later without rewriting them.
  How to verify: Read the tests: none reach into private fields or force state through reflection.
- [ ] Test invalid input and invariants
  Why: Cover the rejected constructions and each method that changes state, not just the happy path.
  How to verify: Each invariant and each constructor rejection has at least one test that fails when it breaks.
- [ ] Document what public methods promise
  Why: Say what each needs, returns and throws, and whether it can return nothing, in the codebase's usual doc-comment style.
  How to verify: Read each public method's doc comment: it states the inputs it rejects, its result, its **failure cases** and any empty result.
- [ ] Have a teammate read the API
  Why: Ask them to use the classes from public methods alone; their questions mark unclear names and missing methods.
  How to verify: A reviewer has written calling code or comments from the caller's side, and each question is resolved.
- [ ] Get security review for sensitive data
  Why: When the classes handle money, permissions, credentials or personal data, ask whoever owns security or privacy to read them.
  How to verify: The review request names the sensitive fields, and every reviewer comment is resolved before merge.

Stop if:
- If a public method others already call must change: stop and agree it with those callers' owners

Move on when tests pass and a reviewer has read the public API from the caller's side.

Not covered: This list stops at the class: service and module boundaries belong to architecture work, and locking designs beyond one class belong to the checklist on running work in parallel safely.

Last reviewed: 2026-10-12

Sources:
1. Martin Fowler: Evans Classification, entities and value objects: https://martinfowler.com/bliki/EvansClassification.html
2. Robert C. Martin: The Single Responsibility Principle: https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html
3. google.github.io: https://google.github.io/eng-practices/review/reviewer/standard.html
4. Robert C. Martin: Clean Architecture, independent of frameworks and UI: https://www.informit.com/articles/article.aspx?p=2832399
5. Bloch, Effective Java: Item 13, minimize accessibility: https://homepages.dcc.ufmg.br/~rodolfo/dcc030-2-11/Effective%20Java%20Item%2013.htm
6. MSDN Magazine: Invariants and Inheritance in Code Contracts: https://learn.microsoft.com/en-us/archive/msdn-magazine/2011/june/msdn-magazine-cutting-edge-invariants-and-inheritance-in-code-contracts
7. Martin Fowler: Anemic Domain Model: https://www.martinfowler.com/bliki/AnemicDomainModel.html
8. Manning, Secure by Design: Domain Primitives chapter: https://www.manning.com/preview/secure-by-design/chapter-5
9. Martin Fowler: Flag Argument: https://martinfowler.com/bliki/FlagArgument.html
10. Microsoft Learn: Best practices for exceptions (.NET): https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
11. Matthias Noback: Objects should be constructed in one go: https://matthiasnoback.nl/2018/07/objects-should-be-constructed-in-one-go/
12. SEI CERT Java: MET05-J, overridable calls in constructors: https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/methods-met/met05-j/
13. Oracle Java docs: Comparable, consistent with equals: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Comparable.html
14. Ploeh blog: On constructor over-injection: https://blog.ploeh.dk/2018/08/27/on-constructor-over-injection/

From CheatGrid: https://www.cheatgrid.com/checklists/computer-science-fundamentals/designing-classes-for-a-new-feature?utm_source=github&utm_medium=listing&utm_campaign=designing-classes-for-a-new-feature
