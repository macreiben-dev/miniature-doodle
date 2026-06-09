# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn start            # ng serve — dev server at http://localhost:4200
yarn run build        # production build
yarn run e2e          # Playwright tests (auto-starts ng serve if not running)
npx playwright test --headed   # watch the browser while tests run
npx playwright test --ui       # interactive Playwright UI mode
```

The Playwright config (`playwright.config.ts`) sets `reuseExistingServer: true`, so you can keep `npm start` running in one terminal and run `npm run e2e` in another without a second server spin-up.

## Architecture

Single-page Angular 17 standalone app. There is no routing — `AppComponent` is a thin shell that renders `RegisterComponent` directly.

**Data flow:**
1. `RegisterComponent` owns a reactive form (`FormGroup`) with three controls: `username`, `password`, `confirmPassword`.
2. Two custom validators run at form level: `passwordRulesValidator` (per-field, 8 rules) and `passwordsMatchValidator` (cross-field group validator).
3. On submit, `AuthService.register()` POSTs to `http://localhost:5000/api/auth/register` and returns an observable of `RegisterResponse | null`.
4. The template drives all error display via `*ngIf` on `control.errors` and `hasPasswordError(code)`.

**Known intentional bugs** (this is a QA kata — do not fix these unless asked):
- **F1** (`register.component.ts:85`): `valueChanges` only watches `confirmPassword`; changing `password` after filling confirm doesn't re-trigger mismatch validation.
- **F2** (`register.component.ts:112`): `onSubmit` guards on `registerForm.pristine` instead of `registerForm.invalid`, so an invalid-but-touched form still fires the API call.
- **F3** (`auth.service.ts:27`): `catchError(() => of(null))` swallows HTTP 4xx errors; API error messages from the response body are never surfaced.
- **F4** (`register.component.ts:48`): `passwordStrength` only checks length, ignoring the other rules — a length-8 all-lowercase password shows "Medium".

## Template note

Angular 17 treats bare `@` as a control-flow keyword. Any `@` in template text must be written as `&#64;` (already applied to the special-character hint in `register.component.html`).
