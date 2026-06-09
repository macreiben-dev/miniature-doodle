# Interviewer Guide — Password Registration Kata

> **Keep this file out of the candidate's hands.**
> Share only `README.md` and the `backend/` + `frontend/` folders.

---

## Seeded Bugs

### Backend (`PasswordValidatorService.cs` / `AuthController.cs`)

| ID | Location | Bug | Expected behaviour | Actual behaviour |
|----|----------|-----|--------------------|-----------------|
| **B1** | `PasswordValidatorService.cs` line with min-length check | `password.Length > 8` instead of `>= 8` | An 8-character password passes | An 8-character password fails with "must be at least 8 characters" |
| **B2** | `HasExcessiveConsecutiveChars` | Counter initialised to `0` instead of `1` | `"aaaPass1!"` (3 identical in a row) fails | `"aaaPass1!"` incorrectly passes; only 4+ consecutive are caught |
| **B3** | `PasswordValidatorService.cs` username check | `password.Contains(username)` — case-sensitive | Check is case-insensitive per spec | `"JohnPass1!"` passes when username is `"john"` |
| **B4** | `PasswordValidatorService.cs` max-length check | Checks `> 128` instead of `> 64` | Passwords > 64 chars are rejected | Passwords up to 128 chars are accepted |
| **B5** | `AuthController.cs` — `Register` method | No null/empty guard before calling `Validate` | Empty or null password returns `400` with a validation error | Throws `NullReferenceException` → `500 Internal Server Error` |

### Frontend (`register.component.ts` / `auth.service.ts`)

| ID | Location | Bug | Expected behaviour | Actual behaviour |
|----|----------|-----|--------------------|-----------------|
| **F1** | `ngOnInit` — `valueChanges` subscription | Only subscribes to `confirmPassword`, not `password` | Changing the password after confirm is filled re-triggers mismatch check | No re-validation; mismatch silently goes undetected |
| **F2** | `onSubmit` guard | `if (this.registerForm.pristine)` instead of `if (this.registerForm.invalid)` | Invalid form cannot be submitted | Any touched (non-pristine) invalid form can be submitted |
| **F3** | `auth.service.ts` — `catchError` | HTTP errors are caught and mapped to `null`; errors never reach the component | API validation errors (`400` response body) shown to user | `null` returned; component displays nothing; user sees no feedback |
| **F4** | `passwordStrength` function | Only checks `password.length` | Strength reflects all rule compliance | `"aaaaaaaa"` (8 chars, no upper/digit/special) shows "Medium" |

---

## Scoring Rubric

### Test Plan (25 pts)

| Score | Signal |
|-------|--------|
| 20–25 | Boundary values for every rule (7, 8, 64, 65 chars). Equivalence classes clearly named. Negative cases as thorough as positives. Cross-field cases (mismatch after changing password). Null/empty inputs. Ambiguities flagged. |
| 12–19 | Most happy-path cases covered. Some boundaries identified. Missing a few edge cases or cross-field scenarios. |
| 0–11 | Only happy-path and obvious negatives. No boundary thinking. No questions asked about spec. |

### Bug Finding (35 pts — 4 pts per backend bug, 3 pts per frontend bug + 3 bonus)

A bug report earns full points if it includes:
1. Clear steps to reproduce
2. Expected vs. actual behaviour
3. Severity / impact assessment

| Bugs found | Notes |
|------------|-------|
| 7–9 | Excellent — systematic tester |
| 4–6 | Solid — found the obvious ones, missed some subtle ones |
| 1–3 | Concerning — only found bugs by accident, not by design |

**Bonus (3 pts):** Candidate notices and documents an ambiguity in the spec (e.g. partial username match, which characters count as "special", Unicode edge cases).

### Test Automation (30 pts)

| Score | Signal |
|-------|--------|
| 25–30 | xUnit tests covering boundary values AND the negative cases. At least 2 Playwright e2e tests that actually verify form behaviour (not just page load). Tests are readable, independent, and would catch the seeded bugs. |
| 15–24 | Some automation. Tests run and pass (or fail for the right reasons). May be missing boundary cases or have fragile selectors. |
| 5–14 | Tests exist but don't add much coverage, or tests are broken and candidate didn't debug them. |
| 0–4 | No automation produced. |

### Communication & Process (10 pts)

| Score | Signal |
|-------|--------|
| 8–10 | Proactively asked clarifying questions before testing. Organised findings clearly. Could explain *why* each bug matters (user impact). |
| 4–7 | Asked some questions, reasonable organisation. |
| 0–3 | Dove straight in without questions. Disorganised output. |

---

## Good Candidate Signals

- Asks about the spec *before* writing tests ("what counts as a special character?")
- Tests boundary values explicitly rather than just picking arbitrary numbers
- Writes a failing test first, then finds the bug that explains it
- Classifies bugs by severity (B2 is low-risk, B5 is a 500 error — critical)
- Notices F2 means the form *can be submitted* while invalid, not just that the button looks wrong
- Realises F3 and F2 interact: you can submit an invalid form and get no error message back — the worst combination

## Weak Candidate Signals

- Only tests the happy path
- Files duplicate bugs (e.g. "button doesn't work" and "form submits with errors" as separate reports without realising they're the same root cause)
- Writes tests that can't actually fail (tests that assert on nothing meaningful)
- Never asks a question about the spec
- Confuses "the frontend shows an error" with "the backend validates correctly" — treats them as the same thing

---

## Suggested Debrief Questions

1. *"You found X bugs — which one would you escalate first, and why?"*
   - Looking for severity thinking: B5 (500 error) and F2+F3 together (silent bad submission) are the most dangerous.

2. *"How would you prevent these kinds of bugs from shipping in future?"*
   - Good answers: shift-left review of validation logic, contract tests between front and back, PR checklist for boundary tests.

3. *"What would you do differently if you had another hour?"*
   - Reveals awareness of coverage gaps they already noticed.

4. *"Are there any requirements you found ambiguous?"*
   - If they didn't raise this during the session, this is a second chance. Good candidates will have a list.
