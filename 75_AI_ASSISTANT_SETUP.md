# 75 — Set Up an AI Assistant for Your Pixfizz Store

**Authority Scope:** How a Pixfizz customer sets up their own AI assistant (Claude) to work on their store: the Pixfizz connector, this knowledge base and the Pixfizz Skills plugin. Customer-facing. Applies to every deployment (Full Pixfizz on Shopper, Full Pixfizz custom, Shopify + Pixfizz, API integration).

_Last updated: 2026-11-25._

The step-by-step version with copy buttons, the project instructions generator and ready-made prompt packs is **aisetup.pixfizz.com**. This file is the same guide in reference form, so an assistant can answer setup questions from it.

---

## The three parts

| Part | What it gives the assistant | Where it is added |
|---|---|---|
| **Pixfizz connector** | Access to your live store: it can look things up and make changes | Claude, Settings, Connectors |
| **Pixfizz knowledge base** | The platform reference (these files), so answers come from how Pixfizz works | A Claude project, as project knowledge |
| **Pixfizz Skills** | Working procedure for specific jobs (pricing, templates, launch, campaigns and more) | Claude desktop app or Claude Code, as a plugin |

Each part works on its own. Together: the connector does the work, the knowledge base supplies the facts, the skills supply the method.

**Naming.** The plugin is called **Pixfizz Skills**. Until late November 2026 it was listed as **Pixfizz Storefront**. It is the same plugin: the install commands, the `/pixfizz:` commands and existing installs are unchanged.

---

## Before you start

- A paid Claude plan (Pro, Max, Team or Enterprise). Projects and plugins need one.
- The Claude desktop app, installed and signed in. The connector and the project can be set up in the browser at claude.ai; the plugin needs the desktop app or Claude Code.
- Your myPixfizz login (`my.pixfizz.com`), your store name and your storefront address.
- Labels inside Claude change from time to time. If a button is named slightly differently, look for the closest match.

---

## 1. Connect Claude to your store (Pixfizz connector)

1. In Claude (desktop app or claude.ai) open **Settings**, then **Connectors**.
2. Click **Add custom connector**. Name it `Pixfizz` and use the address `https://mcp.pixfizz.com`.
3. Click **Add**, then **Connect**, and sign in with your myPixfizz account.
4. On a Team or Enterprise plan an owner adds it once under **Organization settings**, **Connectors**; then each person connects with their own login.

**Test it.** Start a new chat, switch **Pixfizz** on in the chat's tools menu, and ask it to list the stores you can access, give an overview and a health check of your store and show the 5 most recent orders, without changing anything.

**What it does.** Reads and changes store data such as prices, stock, product details, orders, promo codes and snippets, and reports store health. It shows a plan first and changes something only after you say yes. Every change is logged and can be reverted. What the connector can and cannot do in detail: `61_PIXFIZZ_API.md` § Pixfizz MCP connector.

**No stores listed.** The store has to be switched on for the connector in myPixfizz. If myPixfizz shows a banner asking for your Pixfizz API key, follow it first. If that does not help, open a ticket under Support in myPixfizz.

**Rule: connector first, browser last.** A change made through the connector is checked, logged and revertible. A change made by an AI clicking through the admin in a browser is none of those. The assistant should use the browser only for something the connector cannot do, and say so before it does.

---

## 2. Create a Claude project

1. At claude.ai open **Projects**, click **New project** and name it after your store.
2. Switch on **Code execution and file creation** (Pro or Max: **Settings**, **Capabilities**. Team or Enterprise: the owner switches it on under **Organization settings**, together with **Skills**). Without it the skills do not load and Claude cannot open a backup, a CSV or a template export.
3. Keep **Web search** on in the chat's tools menu.

The project also appears in the Claude desktop app, which is the better place for day-to-day work.

---

## 3. Add the knowledge base to the project

The knowledge base is the public repository `https://github.com/pixfizz/pixfizz-knowledge`.

1. Open the project and find its files panel (**Project knowledge** or **Context**).
2. Click **+** or **Add**, choose **GitHub**, sign in to GitHub when asked (a free account is enough).
3. Paste the repository address, select every file and click **Add files**.

- GitHub is added from inside the project, not under Settings, Connectors.
- Connect the original repository. A copy in your own GitHub account, or files you wrote yourself, never receive updates.
- Updates are not automatic. Open Project knowledge and click **Sync now** on the GitHub connection about once a month, or when Pixfizz announces a change.
- No GitHub account: download the repository as a ZIP and upload the markdown files by hand.

---

## 4. Set the project instructions

Use the generator at **aisetup.pixfizz.com** (steps 3 and 4 there): pick your deployment, enter your store name and storefront address, copy the instructions and paste them into the project's **Instructions**, replacing what is there.

The instructions tell the assistant which deployment you are on, to use the connector first, to answer from the knowledge files and name the file it used, and never to invent a feature, setting or formula. An answer with no named source should be treated as a guess.

If the project was set up before you added the connector, copy the instructions again and replace the old ones.

---

## 5. Teach it about your store

In a new conversation inside the project, run the store discovery prompt from aisetup.pixfizz.com (step 5). The assistant reads your storefront and writes a `SITE_CONTEXT.md` for you to correct and upload to Project knowledge.

- On its own it sees only the public storefront. With the connector it also reads products, collections, snippets and store health.
- A website backup attached to that conversation (not to Project knowledge) gives it your pages, snippets and settings as well.
- Update `SITE_CONTEXT.md` when your catalog or setup changes in a way that matters.

---

## 6. Add Pixfizz Skills

**Claude desktop app**

1. Open **Customize**, then **Plugins**.
2. Click **+** or **Add**, then **Add marketplace** (it may say **Add from repository**).
3. Enter `pixfizz/claude-plugins` and confirm.
4. Install **Pixfizz Skills** from the list.
5. Switch on **auto-update** for the Pixfizz marketplace, so new skills arrive on their own.

**Claude Code, in a terminal**

```
/plugin marketplace add pixfizz/claude-plugins
/plugin install pixfizz@pixfizz
```

- The desktop app and Claude Code keep separate plugin lists. If you use both, install in both.
- The `/plugin` commands do not exist in the browser at claude.ai.
- Run `/pixfizz:help` for the current list of skills with an example prompt for each. Skills also load on their own when a question matches.
- Plugin source and changelog: `https://github.com/pixfizz/claude-plugins`.

---

## Keeping it current

| Part | How it updates |
|---|---|
| Connector | Nothing to do on your side. |
| Knowledge base | **Sync now** in Project knowledge, about monthly. |
| Pixfizz Skills | Automatic with auto-update on; otherwise update the plugin in the desktop app. |
| Project instructions | Copy again from aisetup.pixfizz.com when Pixfizz announces a change. |

---

## Common questions

- **Phone or tablet.** Questions and connector work run in the Claude app. Installing the plugin and working on files need the desktop app.
- **Other assistants.** The knowledge base and the instructions pattern carry over to other AI assistants. The plugin is built for Claude.
- **The assistant sends admin links instead of making the change.** Name the connector in the request ("Use the Pixfizz connector to ...") and check the project instructions include the connector rules.
- **An answer does not match your store.** Ask the assistant which knowledge file it used, then raise a ticket in myPixfizz with a screenshot if the file is wrong.
- **Limits.** Test formulas and code before applying them to a live store. Use real product photos rather than AI drawings of your product. The assistant does not replace Pixfizz support for live incidents.

---

## Changelog
- 2026-11-25: New file. Plugin renamed from Pixfizz Storefront to Pixfizz Skills.
