# Shared Nim Code-Review Rules

These conventions define a shared style for Nim projects using Chronos,
Results, Chronicles, and Stew. They are project conventions, not requirements
of the Nim language. Compiler versions, memory managers, repository layouts,
and test helpers remain project-specific choices.

## Naming and General Style

- Use `camelCase` for variables and procedures and `PascalCase` for types.
- Match identifier spelling and casing exactly to the declaration.
- Remove unused imports.
- Prefer runtime configuration or feature flags for experimental features.
  Use compile-time flags only when code must be excluded from the binary for
  security, compliance, platform, ABI, or substantial size/performance reasons.

## Return Values and Variable Declarations

- Do not use the implicit `result` variable to return values. Use an expression
  or an explicit `return value`.
- Prefer expressions to initialize variables.
- Use the most restrictive declaration that fits: `const`, then `let`, then `var`.

## Asynchronous Code

- Use Chronos (`import chronos`) for asynchronous code.
- Use `async`, `await`, and `Future[T]`; asynchronous procedures return
  `Future[T]` or `Future[void]`.
- Specify the exceptions raised by manually created futures, for example
  `Future[T].Raising([SomeError]).init("Owner.operation")`.
- Give every manually initialized future a descriptive name identifying its
  purpose or creation site.
- Use `cancelSoon()` for non-blocking cancellation and `cancelAndWait()` when
  cancellation must complete before execution continues. Avoid `Future.cancel()`.
- Choose the cancellation operation explicitly according to the required
  lifetime and cleanup behavior.
- Use `asyncSpawn` only when the future reference is explicitly tracked.
- Document why each `AsyncLock` is needed.
- Do not use `sleepAsync` to conceal a race or wait for a condition. In tests,
  use a condition-aware helper such as `checkUntilTimeout` when available.
  If a sleep is necessary, add an adjacent comment explaining why.

## Error Handling and Optional Values

- Prefer explicit error-signalling types (`bool`, `Opt`, and `Result`) over
  exceptions and numeric status codes.
- Use the Results library: `Result[T, E]`, `?`, `valueOr`, `isOk`, and `isErr`.
- Prefer `valueOr` to `tryGet()` when handling a failed result explicitly.
  Do not assume an exception raised by `tryGet()` belongs to the project's
  custom exception hierarchy.
- For optional values, import `results` and use the unqualified `Opt[T]` type.
  Avoid alternatives such as `options.Option`.

### Exceptions

- Add explicit `{.raises.}` annotations to new or significantly modified
  exported procedures and functions.
- Raise and catch specific exception types. Avoid raising or catching
  `CatchableError`, and do not use bare `except` clauses.
- Allow `CancelledError` to propagate. If it must be caught for cleanup,
  re-raise it.
- Name the exception variable `e`, for example `except IOError as e`.
- When logging an exception or raising a replacement exception, preserve the
  original message using `e.msg` or `getCurrentExceptionMsg()`. Redact sensitive
  information before including it; the rule against logging secrets takes
  precedence.
- Do not add cancellation exception messages merely to preserve their text.

## Logging

- Choose levels by operational impact and the action required from the user:
  - `error`: a component or background operation has stopped working and
    requires investigation.
  - `warn`: the software remains usable, but configuration, API use, a callback,
    or a resource constraint needs attention.
  - `info`: a low-frequency normal lifecycle event, such as starting or stopping.
- Use `trace`, or `debug` for a bounded operation summary, for expected failures
  caused by external input, such as malformed input, handshake failures, or
  timeouts. These should not normally produce `warn` or `error` messages.
- For every project-defined type used as a log field, define `shortLog` and
  register `chronicles.formatIt` to delegate to it. Keep representations safe
  and bounded, including strings, byte sequences, messages, and collections.
- Define a `logScope` in modules that emit logs.
- Never log private keys, credentials, tokens, sensitive payloads, or sensitive
  exception text at any level. Omit or redact them.

## Types and Public APIs

- Prefer explicit, well-defined types over loosely typed or primitive
  representations of domain values.
- Represent durations with `chronos.Duration`, rather than integers or floats.
- Use named object types with clear fields instead of tuples in public or
  shared APIs. Limit tuple use to functions internal to one file and invoked
  in one place.
- Avoid converters and range types.
- Avoid defining types as `ref object`. Where reference semantics are needed,
  prefer an explicit `ref MyType`, allowing callers to choose where possible.
- Export only procedures, functions, and variables intended to form the public API.

## Binary Data and Integers

- Use `byte` for binary values and `seq[byte]` for dynamic byte arrays.
- Avoid representing binary data as `string`. Convert binary strings returned
  by library APIs to byte sequences as early as possible.
- Use signed integers for counting, lengths, and array indexing.
- Use explicitly sized unsigned integers for binary formats, bit manipulation,
  hardware access, and similar low-level interfaces.
- Do not cast `pointer` to `int`.
- Avoid `Natural`; implicit conversion from `int` can raise a `Defect`.
- Emit hexadecimal output in lowercase and accept either case as input.

## Memory and Object Construction

- Follow the project's configured memory manager; do not assume a particular one.
- Prefer stack-based and statically sized data in core and low-level libraries.
  Use heap allocation in glue layers where appropriate.
- Do not use finalizers or `alloca`.
- Construct objects explicitly, for example `Widget(x: 42, y: Other(z: 54))`,
  or use a provided `Type.init(...)` or `Type.new(...)` constructor.
- Prefer a valid default zero-initialized state.
- Avoid bare declarations such as `var instance: Type` when an explicit
  initializer can preserve compiler diagnostics.
- Name constructors `init` when they create value objects and `new` when they
  return heap-allocated objects.

## Procedures, Callbacks, and Metaprogramming

- Use `func` when possible and `proc` when side effects cannot conveniently
  be avoided.
- Prefer `openArray` to `seq` for parameters used to traverse elements.
- Avoid explicit `{.inline.}` annotations.
- Annotate procedure type definitions and forward declarations with
  `{.raises: [], gcsafe.}`, or declare the specific exceptions they may raise.
- Prefer simple constructs over macros. Avoid generating public API functions
  with macros.
- Put as much logic as possible into templates, using macros to connect them
  when necessary.

## Imports and Dependencies

- Use specific imports and avoid `include`.
- Use the standard library judiciously. Prefer smaller packages with safer APIs
  when they provide equivalent functionality.
- Prefer Stew utilities when equivalent functionality exists in both Stew and
  the standard library.
- Use these preferred alternatives:

| Functionality | Preferred library or module |
| --- | --- |
| Asynchronous operations | `chronos` |
| Bit operations | `stew/bitops2` |
| Endian conversion | `stew/endians2` |
| Error and optional-value handling | `results` |
| I/O | `stew/io2` |
| SQLite bindings | `nim-sqlite3-abi` |
| Streams | `nim-faststreams` |

## Unused Symbols

- Check variables, parameters, procedures, functions, iterators, templates,
  macros, and constants for unused declarations.
- Identify declarations that are never referenced, routines that are never
  called, templates or macros that are never expanded, unused parameters,
  values that are assigned but never read, and unused outer declarations
  hidden by shadowing.
- Check exported symbols for use within and outside the project. Lack of
  internal references alone does not prove that a public API is unused.
- When reporting an unused symbol, provide its name, file, line number, and
  the evidence that it is unused.
- Exempt symbols marked `{.used.}`, symbols required by interfaces, callbacks,
  or external APIs, and compile-time symbols used through `static`, `when`, or
  macro expansion.

## Tests, Callbacks, and Randomness

- Do not use `discard` as an empty test body. Use `expect` when testing that an
  exception is raised rather than swallowing it in a `try`/`except` block.
- Make callbacks that must never run fail explicitly if invoked. Define
  intentional no-op callbacks once and reuse them.
- Do not use `newRng()` as a default constructor or initializer argument.
  Create and pass random generators explicitly to avoid unnecessary entropy
  consumption.
- Reuse the project's shared random-generator fixture or helper in tests
  where one is available.
