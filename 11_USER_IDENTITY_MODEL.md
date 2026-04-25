# 11 — User Identity Model

**Authority Scope:** Identity lifecycle rules only.

_Last updated: 2026-02-26_

---

# 02 — User Identity Model (Locked Canon)

## Core rule
A user **always exists**.

## Anonymous user
- Session-based identity stored in browser.
- Owns cart/projects/galleries during browsing.

## Registered user
- Persisted user record with credentials.
- Can be logged out or logged in.

## Logged-in state
- Registered user with an authenticated session.
- On login, anonymous user data is passed/transferred to the existing user.

## Guest user
- Guest checkout creates a persisted **guest user** flagged as guest.
- Guest cannot login or reset password.
- Guest can receive emails and owns orders/projects/galleries.
- Admin can merge guest into registered user, transferring data.

## Admin / Super Admin
Roles/permissions on user accounts.
