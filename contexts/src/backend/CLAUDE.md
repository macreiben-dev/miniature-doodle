# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands run from the solution root (`src/backend/`).

```sh
# Build
dotnet build

# Run the API (listens on http://localhost:5000 by default)
dotnet run --project src/PasswordKata.Api

# Run all tests
dotnet test

# Run a single test by name
dotnet test --filter "FullyQualifiedName~ValidPasswordShouldPass"
```

## Architecture

This is a .NET 8 minimal-API solution with two projects:

- **`src/PasswordKata.Api`** — ASP.NET Core web API. Single endpoint: `POST /api/auth/register` (in `AuthController`). `PasswordValidatorService` is registered as a singleton and injected via `IPasswordValidatorService`. The in-memory `_registeredUsers` set lives on the controller (static field) — it resets on restart and is intentionally not production-grade.
- **`tests/PasswordKata.Tests`** — xUnit + FluentAssertions. Tests instantiate `PasswordValidatorService` directly (no mocking needed). The test file is a candidate scaffold with TODO stubs for the kata.

CORS is locked to `http://localhost:4200` (Angular frontend assumed).

## Intentional Bugs

The codebase contains five documented bugs introduced for the QA kata. Each is annotated with a `// BUG Bn:` comment:

| ID | Location | Description |
|----|----------|-------------|
| B1 | `PasswordValidatorService.Validate` | Uses `>` instead of `>=` for min-length check — exactly 8-char passwords incorrectly fail |
| B2 | `HasExcessiveConsecutiveChars` | Counter starts at `0` instead of `1` — exactly 3 consecutive identical chars incorrectly pass |
| B3 | `PasswordValidatorService.Validate` | Username containment check is case-sensitive — `"JohnPass1!"` passes when username is `"john"` |
| B4 | `PasswordValidatorService.Validate` | Max-length threshold is `128` instead of the specified `64` |
| B5 | `AuthController.Register` | No null/empty guard on `request.Password` — throws `NullReferenceException` instead of returning 400 |

Do **not** fix these bugs unless explicitly asked; they are the subject of the QA exercise.
