# Starring You: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.1.x, prototype and demo on experience.pixfizz.com (Nov 2026), not on customer sites. Snippet: `product/starring-you`. Mount option: `sy_mount`.

The shopper, and their pet, drawn by AI into a fun scene (a beach chair, a roadster) on a print.

Starring You has **no mount settings**. The mount is only the snippet call: `{% snippet 'product/starring-you' %}`.

| Where | What | Can you change it? |
|---|---|---|
| Child snippet `admin/checklist/starring-you` | The switch. `TRUE` turns the studio on; anything else leaves it off. Off by default. | **Yes**, with Pixfizz: AI needs AI tokens on your site |
| Custom type `starring_you_scene` on your site | The scenes: code, name, categories, needs pet, sample, scene image, prompt, sort, active. Each site has its own scenes; they are not inherited. Set `active` to `false` to hide one. | **Yes** for name, sort and active. **Ask Pixfizz** before changing a prompt |
| Snippet `starring-you/prompts` | Prompt lines shared by every scene | **No** |

*Verified by reading `custom-design-tools/starring-you/build/src/product__starring-you.liquid`, 24 Nov 2026. Prototype: not yet verified on a customer site.*

## Changelog

- 2026-11-24: Page created.
