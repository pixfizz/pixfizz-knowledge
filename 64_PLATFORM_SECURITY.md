# 64 — Platform Security

**Authority Scope:** How the Pixfizz platform protects admin access, integrations and stores, and how to answer "is Pixfizz affected by vulnerability X?". Platform-level (Pixfizz Core): applies to Shopper, custom CMS sites and Shopify + Pixfizz admins. Sign-in mechanics live in `18_ADMIN_NAVIGATION.md` § Admin Login: Two-Factor and Passkeys; API keys in `61_PIXFIZZ_API.md` § API keys; service users in `11_USER_IDENTITY_MODEL.md` § Service user. Customer-facing version: help.pixfizz.com, Getting started, "Security at Pixfizz".

_Last updated: 2026-11-25._

---

## Admin sign-in

- Admin URL format: `https://admin.pixfizz.com/site/<site-slug>/admin`. Super admin: `https://admin.pixfizz.com/superadmin`.
- Sign-in is handled on `login.pixfizz.com`. Pages on `login.pixfizz.com` carry a one-time `state` parameter and must never be bookmarked or shared. Bookmark the admin URL instead; it redirects to sign-in when needed.
- After the move to the admin domain, saved password manager entries for old admin addresses can autofill the wrong password. Fix: update the saved entry to the new admin address.
- Admin sessions end after 24 hours of inactivity. *Confirmed by Pixfizz, 2026-11-25.*
- Repeated failed sign-ins and automated sign-in attempts are blocked (brute-force and bot protection). Scripts that sign in with a username and password to the login form will be blocked; use an API key.

## Two-factor authentication (2FA)

- 2FA is required for all admin and super admin accounts (enforced since October 2026).
- Methods: passkey (WebAuthn: Face ID, Touch ID, Windows Hello, security key) or TOTP authenticator app. A user can register both.
- Once a passkey is registered, sign-in requires it unless an authenticator app is also registered, in which case the app is offered as the alternative. A user with only a passkey who moves to a new computer or browser cannot sign in there. Advise every admin to register both methods.
- Lost device: a super admin resets the user's 2FA from the user page in admin. The user then registers a new method at next sign-in.
- TOTP codes fail when the device clock is wrong; set the phone clock to automatic.
- AI browser agents (Claude in Chrome and similar) cannot use the user's passkey. For agent work use the Pixfizz connector with an API key, or have the user sign in in the browser first.

## Email sign-in links

- Sign-in links sent by email open a confirmation page with a Confirm button; the user is signed in only after clicking it. This prevents email security scanners that pre-fetch links from expiring the token.

## Account security page

- Open it from the three dots in the bottom left corner of the admin, then Account security.
- Admin users can change their password, manage passkeys and the authenticator app, and sign out all other sessions ("sign out elsewhere") there.

## API keys

- Use API keys for every API request instead of HTTP Basic auth with a username and password.
- Send the key in the username position of HTTP Basic auth; the password is ignored and may be empty.

```
curl -u pxk_<key>: https://<site>.pixfizz.com/v1/admin/orders.json
```

- API key requests are not subject to the sign-in brute-force protection and are faster (about 0.5 s less per request than username and password).
- Create keys under Admin > Users > Users, on the user the integration acts as, in the API Keys section. Keys start with `pxk_`.
- A key created in a site's admin is scoped to that site.
- A key assigned to a super user in super admin grants access to every site that super user can access. Treat it as highly sensitive.
- Never paste an API key into Liquid, storefront JS, a public repo, a help article or a chat transcript.

## Service users

- A checkbox on the user page in admin and super admin marks a user as a service user.
- Service users can only authenticate with API keys and can never sign in with username and password.
- Mark every account that exists only for an integration (connector, OrderHub, partner system, scripts) as a service user. Staff who already use strong passwords and 2FA may create keys under their own account; actions then appear in the admin under their name.

## Request filtering and injection protection

- A web application firewall layer checks every request against known attack patterns. Matching requests are rejected and repeat sources are blocked temporarily. Patterns are updated as new attacks appear.
- A strict Content Security Policy is enforced in the admin. Inline event handlers and `javascript:` links are not allowed in admin pages; scripts must use delegated handlers.
- Text inputs are length-validated before storage.
- Stored FTP passwords and HTTP header values for fulfillment are masked on screen.

## Infrastructure

- Pixfizz Core runs on Ruby on Rails. No PHP is used anywhere in the stack, so PHP vulnerabilities do not affect Pixfizz sites.
- Servers receive scheduled security updates. Planned maintenance can make FTP unavailable for 5 to 10 minutes and cause brief web disruptions of a few seconds; staff announce these in advance.

## Answering "Is Pixfizz affected by vulnerability X?"

1. Check whether X targets a technology Pixfizz uses. If it targets PHP, WordPress plugins or another stack Pixfizz does not run, answer "not affected" and name the reason.
2. If X targets Ruby, Rails, a library, the OS or a browser, the developer confirms status and any patch. Do not answer "not affected" before that confirmation.
3. Add a dated line to the Security notices table on the help.pixfizz.com Security page.
4. A storefront on Shopify, WordPress or another external platform is outside Pixfizz Core; the customer checks that platform separately.

## Changelog
- 2026-11-25: New file. Admin sign-in and session rules, 2FA summary, email sign-in link confirmation, account security page, API keys and service users summary, request filtering and script policy, infrastructure (no PHP), and the procedure for vulnerability questions.
