# Aurvex Login Page - QA Bug Report

**Project:** Aurvex Enterprise Platform - Authentication Module  
**Tested by:** Dmitrii Rzhanykh  
**Date:** May 2026  
**Environment:** Chrome 124, Desktop, Windows 11  
**Test type:** Manual exploratory testing

---

## Summary

During exploratory testing of the Aurvex login page, **4 issues** were identified spanning validation logic, security exposure, keyboard accessibility, and error handling. None are critical in isolation, but together they indicate gaps in frontend validation design and UX consistency.

| # | Title | Severity | Category |
|---|-------|----------|----------|
| BUG-01 | Empty username bypasses client-side validation | Medium | Validation |
| BUG-02 | Password field briefly exposes plaintext | High | Security |
| BUG-03 | Enter key does not submit form from password field | Medium | Accessibility |
| BUG-04 | Identical error response for all authentication failures | Low | UX / Error Handling |

---

## BUG-01 - Empty username bypasses client-side validation

**Severity:** Medium  
**Category:** Validation  
**Status:** Open

### Description

The username field displays an inline validation error on blur when left empty. However, the Sign In button performs its own independent validation that only checks whether the password field is non-empty, ignoring the username validation state. As a result, a user can submit the form with an empty username and trigger an authentication request.

### Steps to Reproduce

1. Open the login page.
2. Click on the Username field, then click away without entering anything.
3. Observe the inline error: "This field is required" appears under the field.
4. Enter any value in the Password field.
5. Click **Sign in**.

### Expected Result

Form submission is blocked. The username inline error remains visible. No authentication request is made.

### Actual Result

Form submits successfully. An authentication request is made with an empty username string. The app returns a generic authentication error instead of a client-side validation message.

### Root Cause Observed

Two separate validation layers exist with inconsistent logic:

| Validation layer | Behavior |
|------------------|----------|
| Blur handler | Validates username presence |
| Submit handler | Only validates that password is not empty |

The submit handler does not read or respect the result of username blur validation.

### Impact

- Unnecessary authentication requests for clearly invalid input
- Inconsistent UX: inline error appears but does not prevent submission
- Empty username input may mask deeper validation or backend handling issues

---

## BUG-02 - Password field briefly exposes plaintext

**Severity:** High  
**Category:** Security / Input Handling  
**Status:** Open

### Description

When the user enters a password, the input temporarily switches from masked password display to plaintext, making the entered password characters visible. The field then silently reverts back to `type="password"` after a short delay.

### Steps to Reproduce

1. Open the login page.
2. Click into the Password field.
3. Type any password.
4. Observe the password field while typing.

### Expected Result

Password characters should remain masked. The input type should stay as `password` unless the user explicitly clicks the show/hide toggle.

### Actual Result

The password field briefly exposes entered characters in plaintext before reverting automatically.

### Root Cause Observed

Password input handling temporarily enables visible text without explicit user intent. The revert is controlled by a timeout rather than a user action.

### Impact

- Password is visible to anyone with line-of-sight to the screen
- Risk is higher in public spaces, shared environments, or during screen sharing
- User is not clearly informed that the exposure occurred
- Sensitive input is exposed without explicit consent

---

## BUG-03 - Enter key does not submit form from password field

**Severity:** Medium  
**Category:** Accessibility / Keyboard Navigation  
**Status:** Open

### Description

Pressing Enter while focused on the password field does not submit the form. The form can only be submitted by clicking the Sign in button with a mouse or pointer device.

### Steps to Reproduce

1. Enter a value in the Username field.
2. Enter a value in the Password field.
3. Press **Enter** while the cursor is inside the Password field.
4. Observe the result.

### Expected Result

Pressing Enter in the password field submits the login form, matching standard browser and accessibility behavior for login forms.

### Actual Result

Enter key input is suppressed. No form submission, error message, loading state, or visual feedback appears. The user must click **Sign in** manually.

### Root Cause Observed

A `keydown` event handler on the password field calls `preventDefault()` on Enter without triggering form submission as a fallback.

### Impact

- Breaks standard login UX expectation
- Creates friction for keyboard users
- May fail WCAG 2.1 SC 2.1.1 Keyboard expectations if login cannot be completed using keyboard alone
- Creates inconsistent and confusing interaction behavior

---

## BUG-04 - Identical error response for all authentication failures

**Severity:** Low  
**Category:** UX / Error Handling  
**Status:** Open

### Description

All authentication failure scenarios return the exact same visible error message and error code, regardless of the actual failure reason. Wrong password, wrong username, and empty username all result in the same generic authentication failure path.

### Steps to Reproduce

**Scenario A - Wrong password**

1. Username: `admin`, Password: `wrongpassword`
2. Click **Sign in**.
3. Note the error message and code.

**Scenario B - Wrong username**

1. Username: `notauser`, Password: `password123`
2. Click **Sign in**.
3. Note the error message and code.

**Scenario C - Empty username**

1. Username: empty, Password: `wrongpassword`
2. Click **Sign in**.
3. Note the error message and code.

### Expected Result

While exact authentication failure reasons should not expose sensitive information, the error handling should at minimum:

- Distinguish between invalid input and incorrect credentials
- Provide useful recovery guidance
- Use distinct internal error handling paths for logging and debugging

### Actual Result

All tested failure scenarios return the same generic authentication failure behavior.

### Root Cause Observed

A single error path handles all failure cases. The same error code is used for different failure types instead of separating client-side validation failures from authentication failures.

### Impact

- Users cannot tell whether they made an input mistake or entered incorrect credentials
- Support burden can increase because users receive limited recovery guidance
- Static error handling makes debugging and analytics less useful
- Note: not distinguishing wrong username vs wrong password can be correct for security, but empty input should still be handled separately by client-side validation

---

## Test Cases

### TC-01 - Username field validation on submit

| Field | Value |
|-------|-------|
| Test Case ID | TC-01 |
| Related Bug | BUG-01 |
| Priority | High |

| Step | Action | Expected | Actual | Pass/Fail |
|------|--------|----------|--------|-----------|
| 1 | Leave Username empty, enter any password, click Sign in | Form blocked, "This field is required" shown | Form submits, server error returned | Fail |
| 2 | Enter username, clear it after blur, click Sign in | Form blocked | Form submits | Fail |
| 3 | Enter valid username and password, click Sign in | Successful login | Successful login | Pass |

---

### TC-02 - Password field masking

| Field | Value |
|-------|-------|
| Test Case ID | TC-02 |
| Related Bug | BUG-02 |
| Priority | High |

| Step | Action | Expected | Actual | Pass/Fail |
|------|--------|----------|--------|-----------|
| 1 | Type password and observe field | Characters remain masked | Characters briefly visible | Fail |
| 2 | Type password, click show/hide toggle | Characters toggle on user action only | Works correctly | Pass |
| 3 | Repeat password entry multiple times | Characters always remain masked unless toggled | Characters are exposed during input | Fail |

---

### TC-03 - Keyboard form submission

| Field | Value |
|-------|-------|
| Test Case ID | TC-03 |
| Related Bug | BUG-03 |
| Priority | Medium |

| Step | Action | Expected | Actual | Pass/Fail |
|------|--------|----------|--------|-----------|
| 1 | Focus Password, press Enter | Form submits | Nothing happens | Fail |
| 2 | Fill both fields, press Enter from Password field | Form submits | Nothing happens | Fail |
| 3 | Fill both fields, click Sign in with mouse | Form submits | Form submits | Pass |

---

### TC-04 - Error message differentiation

| Field | Value |
|-------|-------|
| Test Case ID | TC-04 |
| Related Bug | BUG-04 |
| Priority | Low |

| Step | Action | Expected | Actual | Pass/Fail |
|------|--------|----------|--------|-----------|
| 1 | Submit with wrong password | Secure credential error shown | Generic auth error shown | Partial |
| 2 | Submit with wrong username | Secure credential error shown | Same generic auth error shown | Partial |
| 3 | Submit with empty username | Client-side validation error | Same generic auth error shown | Fail |

---

## Notes

- Testing was performed on a frontend demo.
- All bugs are reproducible consistently across multiple attempts.
- BUG-02 is the highest priority fix due to potential security exposure in shared environments.
- BUG-03 should be prioritized for keyboard accessibility and standard login UX compliance.
