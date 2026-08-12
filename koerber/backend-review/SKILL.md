---
name: backend-review
description: Review NestJS/TypeScript backend changes for the load-bearing conventions that CI can't catch — tests follow the self-contained-data + AAA structure, repositories return domain entities (not raw Prisma rows), the compiler is never forced with `as`, GraphQL inputs are validated, and fields exposed via `@ResolveField` are not also duplicated as `@Field` on the DTO. Use when reviewing or writing anything under `backend/` — a `*.repository.ts`, `*.resolver.ts`, `*.input.ts`, `*.dto.ts`, `*.service.ts`, or `*.spec.ts` — e.g. "review this resolver", "is this repository okay", or before committing backend changes.
---

# Backend review

Review changes in `backend/` (NestJS + Prisma + Apollo GraphQL, Clean/Hexagonal + DDD) against the five rules below. Each rule is something the type-checker and linter do **not** enforce, so a human/agent review is the only gate. Flag every violation with `file:line`, the tier, the rule broken, and the concrete fix. The authoritative sources are `.github/instructions/backend.instructions.md` and `typescript.instructions.md`; this skill is the review lens over them.

## Rule 1 — Tests are self-contained and strictly AAA

Defer to the **`backend-test-authoring`** skill for the full ruleset; this is the review checklist over it. Backend tests run against a **real Postgres** truncated after every test, so nothing survives between `it` blocks.

- **Self-contained data:** a test (or its `beforeEach`/`beforeAll`) seeds the whole FK chain up to the leaf row it needs — project → dataset → model training → pipeline run → prediction. Flag any reliance on rows "that happen to be there": `prisma.x.findFirstOrThrow(...)` fishing for a parent, a module-level `const id = randomUUID()` reused across tests with no row behind it, or a raw `randomUUID()` passed as a FK.
- **Factories, not literals:** every `seeder.pushX(...)` receives a `TestXFactory.makeX(...)` value with only the test-relevant overrides — never a hand-assembled object literal, never a hardcoded id (snapshots excepted, with a comment).
- **No spec-local seeding helpers:** reusable graph seeding lives in a `TestSeeder` `push…WithParents` method (like `pushSetWithParents`), or inline in a `beforeEach`. A free-standing `const seedX = async () => {…}` composed helper in the spec is a violation.
- **Shared parents in hooks:** DB rows in `beforeEach` (truncation wipes them); only immutable non-DB setup (compiling the Nest app, grabbing injected services) belongs in `beforeAll`.
- **Strict AAA:** three blocks in order, ideally comment-labelled; exactly **one** Act (one service call or one HTTP request); no seeding/acting in Assert. Assertions use `expect.objectContaining(...)` / `expect.arrayContaining(...)` — never chain into a response by index/property (`body.data.items[0].id`).
- **Mocks & misc:** external services mocked via `test/mocks` (not `jest.fn()`/`jest.mock`); no direct `Date`/time assertions; `*.spec.ts` colocated next to the unit; a new module is registered in `test/aut.ts`; a missing factory is added under `test/factories/` and exported from the barrel.

## Rule 2 — Repositories return entities, never raw Prisma rows

Persistence is a detail. A repository is a port implementation: it wraps Prisma and hands the service a **domain entity**, so the Prisma row shape never leaks past the repository boundary.

- Every public repository method returns a domain `*Entity` (or `Entity | null`, or `Entity[]`), mapped via the entity's factory (`ReleasedModelEntity.fromPrisma(row)`), **not** the object Prisma returned.
- **Flag** a method whose return type is a Prisma model type, `Prisma.XGetPayload<…>`, or an inline row shape — and any `return row` / `return this.prisma.x.findMany(...)` that hands the DAO straight out.
- The Prisma client type is a **DAO** used *inside* the repository only. Prisma calls must not appear in services, resolvers, or controllers (that's a separate layering break — flag it too).
- Mapping direction is fixed: `row → Entity` happens in the repository; `Entity → DTO` happens later in the DTO's `map()` (Rule 5). A repository returning a DTO, or a service receiving a raw row, is inverted — flag it.
- Writes take an entity in and return the persisted entity out (re-read/`include` as needed, then map). Don't return `void` where the caller needs the generated id/timestamps, and don't return the raw create result.

## Rule 3 — Never force the compiler with `as`

Strict mode is mandatory and a type error fails CI, so an `as` cast is a silent override of the one gate that would have caught a bug. The type system should be *satisfied*, not *told to be quiet*.

- **Flag every `as`** that changes or asserts a type: `x as Foo`, `x as unknown as Foo` (double-cast, the worst offender — it launders any type into any other), `as any`, and non-null `!` assertions on values that weren't just validated.
- **Prefer, in order:** a **type guard** (`item instanceof ClassificationEntity`, a `isFoo(x): x is Foo` predicate) → a **discriminated union** + exhaustive `switch` with a `never` default → **generics** so the type flows through → fixing the source type so the cast is unnecessary.
- **`satisfies` is the sanctioned tool** where an `as` is tempting for shape-checking: `Object.assign(new Dto(), { … } satisfies Dto)` validates the literal against the type **without widening or silencing** — that's the DTO-`map()` idiom in this repo, and it is *not* a violation.
- **Narrow allowances (still comment them):** `as const` for literal narrowing; the GraphQL/Prisma enum bridge and other framework seams that genuinely can't be expressed otherwise. Every surviving `as` needs a one-line comment naming what forced it — an uncommented cast is a finding.
- No implicit or explicit `any`: use `unknown` + narrowing. `as any` to reach a property is always a violation — type the value or guard it.

## Rule 4 — GraphQL inputs are validated

Every external input crosses a trust boundary. GraphQL's type layer only guarantees *shape*, not *validity* (a `String` can still be a 10 MB blob or an injection payload), so each input field carries `class-validator` decorators and the pipeline runs `ValidationPipe`.

- Every `@InputType()` / `@ArgsType()` field has the validation constraints its domain implies — `@IsUUID()` on ids, `@IsNotEmpty()` + `@MaxLength(n)` + a `@Matches(PRINTABLE_TEXT_REGEX)`-style pattern on free text, `@IsEnum`, `@Min`/`@Max`, `@IsEmail`, etc. **A bare `@Field()`/`@GraphField()` with no validator on a user-supplied value is a finding.**
- **Nested inputs must be validated recursively:** an input containing another input needs `@ValidateNested()` **and** `@Type(() => ChildInput)` — without `@Type`, class-transformer leaves a plain object and the child's decorators never run. Flag `@ValidateNested()` missing its `@Type`, and arrays of inputs missing `{ each: true }`.
- **Optional ≠ unvalidated:** an optional field still validates *when present* (`@IsOptional()` alongside the constraints). `?:` without `@IsOptional()`, or an optional field that drops its other validators, is a finding.
- Prefer the repo's `@GraphField`/`@OptionalGraphField` helpers for the GraphQL wiring, but they don't add validation — the `class-validator` decorators must still be there.
- Trust the DB for persistence concerns (generated ids, timestamps, DB constraints); trust `class-validator` for input concerns. Don't hand-roll validation logic inside the resolver/service that a decorator expresses declaratively.

## Rule 5 — `@ResolveField`s are not duplicated on the DTO

A relation is resolved **exactly one way**: either eagerly as a mapped `@Field` on the DTO, or lazily via a `@ResolveField` on the resolver — never both. Declaring both makes the schema ambiguous, doubles the fetch, and lets the two paths drift.

- If the resolver has a `@ResolveField(() => XDto) x(@Parent() …)`, the DTO must **not** also declare `@Field(() => XDto) x` for the same name. Flag the duplicate `@Field` — the resolver owns that field; the DTO carries only the **context** needed to resolve it.
- **Context fields carry ids, not resolved objects.** The `map()` populates plain, non-`@Field` context properties (`setIds: UUID[]`, `machineIds: UUID[]`, `createdBy`) that the `@ResolveField`s consume via `@Parent()`/loaders. A DTO field that is both `@Field`-exposed *and* re-fetched by a resolver is the smell this rule targets.
- **Every `@ResolveField` needs RBAC when a matching permission exists** — `middleware: [RBACField({ requiredPermissions: [...] })]` on the field, not a class-level guard standing in for it. A resolve-field exposing protected data without `RBACField` is a **must-fix**.
- **DTO mapping is single-factory:** every DTO exposes exactly one `static map(entity)` (never `fromEntity`/`fromRecord`/etc.), and entity→DTO conversion happens **only** there. `Object.assign(new Dto(), …)` *inside* `map()` is the idiom; `Object.assign` or field-by-field mapping **in a resolver/controller** is a violation — push it into `map()`.
- Prefer DataLoaders for `@ResolveField` relations (avoid the N+1); a resolve-field issuing a direct per-parent query in a list context is a performance finding.

## Scope & severity

**Diff-scoped:** review only changed lines/files. A pre-existing violation in a *touched* file gets one collapsed mention ("pre-existing, not from this change — optional cleanup"), never mixed into the main findings. Untouched files are off-limits. Never ask a PR to fix violations it didn't introduce.

**Two tiers:**

- **Must-fix (blocking):** a repository leaking a raw Prisma row (Rule 2); an `as`/`as unknown as`/`as any` that launders a type or a non-null assertion on unvalidated data (Rule 3); an unvalidated user-supplied input or a `@ValidateNested` missing its `@Type` (Rule 4); a `@ResolveField` exposing protected data without `RBACField`, or a relation resolved both as `@Field` and `@ResolveField` (Rule 5); a test depending on implicit/pre-existing data (Rule 1). These are correctness, security, or data-integrity risks.
- **Should-fix (advisory):** hand-built seed literals or spec-local seeding helpers, AAA blur, index/property assertions (Rule 1); a commented-but-avoidable cast (Rule 3); an optional field missing `@IsOptional()` (Rule 4); `Object.assign`/manual mapping in a resolver instead of `map()`, or an N+1-prone resolve-field (Rule 5).

## Output format

For each finding: `file:line`, the tier, which rule was broken, and the fix (the guard/factory/decorator to use, or how to restructure). Must-fix first, then should-fix, then the collapsed pre-existing note. If the code is clean, say so — don't invent findings.
