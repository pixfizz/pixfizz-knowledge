# AI switches in the studios: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## What this is
Several studios can use the platform AI image tools: cut a person out of their background, turn a photo into a cartoon, color-correct a photo. **Every AI feature is a switch in the tool's mount, off by default** (or offered only when you list it). A mount without the switch behaves exactly as before and loads no AI files. Platform-level rule, from Nov 2026.

How AI behaves in every studio
- AI tools need a **signed-in shopper**. The studio signs them in on the page, in the tool; it never sends them away.
- AI uses **AI tokens** on your site. A site with AI off: the studio hides its AI choice. The platform cannot be asked in advance whether AI is on, so on such a site the AI choice may show until the first attempt fails; the studio then removes it and remembers that for the site.
- Every AI result is saved once to the shopper's **My Creations** gallery, so they can reuse it.

| Studio | AI switch in the mount | Off by default? | Status (24 Nov 2026) |
|---|---|---|---|
| Fan Face | `auto` in `facefan_cut_modes` ("Cut around the head") | Yes: only when listed | 2.0.0 built, not released (`fan-face.md`) |
| Golf Ball Studio | `gb_ai` (`on` / `off`): cut out, cartoon, logo background remover; `gb_enhance` (`on` / `off`): color correction | No: the example mount has `on`; set `off` to turn it off | 1.0.0 built (`golf-ball-studio.md`) |
| Starring You | child snippet `admin/checklist/starring-you` = `TRUE` | Yes | prototype (`starring-you.md`) |
| Creative Studio (Cartoon Me) | not released yet | | in build |

Whether a lab may charge shoppers for an AI result is not decided yet. None of these settings adds a price.

*Verified by reading `shared/px-ai/README.md` (0.2.0) and the tool snippets named above, 24 Nov 2026.*

## Changelog

- 2026-11-24: Page created.
