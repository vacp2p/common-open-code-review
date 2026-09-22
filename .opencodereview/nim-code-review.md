# Shared Nim Code-Review Rules

## Return Values and Errors

- Do not use the implicit `result` variable to return values; use an expression-based return or `return value`.
- Prefer explicit error-signalling types (`bool`, `Opt`, `Result`) over exceptions or status codes. Use `Opt[T]` from `results` rather than other option-like types.
- Do not use bare `except` or catch `CatchableError`; use specific exception types. Do not catch `CancelledError` except to re-raise it. Preserve an original exception message when wrapping or logging it. New or significantly modified exported procedures should declare an explicit `{.raises.}` annotation.

## Async and Concurrency

- For Chronos code, use `Future[T]`/`Future[void]`, give manually created futures an explanatory `init("Owner.operation")` name, and specify their raised exceptions. Prefer `cancelSoon()` for non-blocking cancellation and `cancelAndWait()` when completion must be awaited.
- Do not use untracked `asyncSpawn`; retain a future reference when its lifetime matters. Document why every `AsyncLock` is needed.
- Do not use `sleepAsync` to hide races or wait for a condition. Tests should use a condition-aware helper when available; an unavoidable sleep needs an adjacent reason.

## Types, Data, and APIs

- Prefer strong domain types. Represent durations with `chronos.Duration`, binary data with `byte`/`seq[byte]`, and use signed integers for counts, lengths, and indexes. Avoid `Natural`, pointer-to-int casts, range types, converters, finalizers, and `alloca`.
- Do not expose tuples in public APIs; use a named object with clear fields. Avoid `ref object` unless reference semantics are essential; prefer an explicit `ref MyType` type where callers can choose it.
- Prefer `func` when side effects are avoidable, `openArray` for traversal parameters, and the most restrictive of `const`, `let`, and `var`. Avoid unintended exported symbols.

## Style and Dependencies

- Use camelCase for variables/procedures and PascalCase for types. Use specific imports, avoid `include`, unused imports, explicit `{.inline.}`, and unnecessary macros. Prefer templates to macros where practical.
- Construct values with object literals or `Type.init(...)`; an `init` procedure creates a stack object and `new` returns a heap object. Prefer a valid zero-initialized state.
- Prefer safer ecosystem replacements when applicable: `chronos` over std async, `stew/bitops2`, `stew/endians2`, `results`, `stew/io2`, and `nim-faststreams`. Prefer `stew` when it supplies an equivalent safer utility.
- Do not use `discard` as an empty test body; use an assertion or a reusable, explicit no-op callback. Random generators must be passed in rather than created as default arguments. Emit hexadecimal output in lowercase while accepting either case on input.

## Logging and Review Scope

- Never log credentials, keys, tokens, private data, or sensitive exception text. Match log severity to operational impact; expected peer/input failures generally belong at trace/debug, not warning/error.
- Flag unused newly introduced symbols only when the usage cannot be supplied by an interface, callback, external API, compile-time expansion, or `{.used.}`.
- When code changes are made, ensure modified Nim files are formatted with the repository's configured formatter (commonly `nph`/`nimble format`).
