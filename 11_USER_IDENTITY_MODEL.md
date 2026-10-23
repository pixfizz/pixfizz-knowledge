# 11 — User Identity Model

**Authority Scope:** Identity lifecycle rules only.

_Last updated: 2026-10-22_

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
- **Site admin access is all-or-nothing.** The user page has a single Admin checkbox; there are no admin roles or admin categories, so an account cannot be limited to one area such as Projects. Platform-level (Pixfizz CMS). *Stated in a KB patch, 2026-10-21; not verified by query.*
- **Super admin** (Pixfizz staff) adds what a site admin does not have, such as super accounts, fulfillment settings, billing and page XML access (`18_ADMIN_NAVIGATION.md` § Super Admin). *Stated, not verified.*

## Same email on two sites
- A user is per site. The same email on a different site is a completely separate user. myPixfizz, by contrast, treats an email as globally unique, so single sign-on from the Pixfizz admin into myPixfizz carries the site subdomain (`70_MYPIXFIZZ_OVERVIEW.md` § Single sign-on from the Pixfizz admin). Platform-level. *Stated by the core developer, Slack #development, 2026-10-20.*

## Changelog
- 2026-10-22: Admin / Super Admin: site admin access is all-or-nothing (single Admin checkbox, no roles) and what super admin adds. New Same email on two sites (per-site users, myPixfizz SSO pointer). Source: claude-chat, slack-message.
