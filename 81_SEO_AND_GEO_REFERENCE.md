# 81 — SEO and GEO Reference

**Authority Scope:** How AI-powered search works, Generative Engine Optimization (GEO) practice, and what the Pixfizz platform and Shopper template currently do for AI search visibility. Industry concepts are general knowledge. Platform and template capabilities are tagged explicitly.

_Last updated: 2026-06-18_

---

## Purpose

Reference for advising customers and staff on search visibility in the AI era. Covers three things:

1. How AI search works and how it differs from traditional keyword search (general).
2. The GEO playbook — how to get cited by AI answer engines (general).
3. What Pixfizz and Shopper actually provide for this today, and the current gaps (platform truth).

For onboarding-time SEO setup (URL mapping, 301 redirects, sitemap submission, pre-launch description checks) see `80_ONBOARDING.md` § SEO Migration and § Content Completeness Before Launch. This file does not repeat that material.

**Layer tags used below:** `[PLATFORM]` = always true of Pixfizz CMS. `[SHOPPER]` = Shopper template behavior, may not apply to Custom or Shopify paths. `[GENERAL]` = industry knowledge, not Pixfizz-specific. `[GAP]` = not currently provided. `[CONFIRM]` = needs Matjaz/team confirmation.

**On statistics:** Figures in this file are directional and drawn from third-party 2025–2026 research. The precise numbers vary widely between sources. Verify any specific figure against its named source before using it in a customer-facing context (webinar, slide, email).

---

## Part A — How AI Search Works (General)

### The shift from links to answers

Search has moved from returning a list of ten links to returning a single AI-written answer at the top of the page. The AI does the reading and shortlisting for the customer. Being seen no longer means being clicked.

- **AI Overviews** are AI-generated summary blocks at the top of Google results, condensing multiple sources into one answer with citation links.
- **AI Mode** is Google's conversational search experience, where users ask complex, follow-up questions and can input images or voice without leaving Google.
- Third-party answer engines that matter: ChatGPT, Perplexity, Gemini, Claude, Copilot.

### How an answer is assembled

- **Query fan-out:** the engine breaks one customer question into multiple background sub-queries and searches each separately. (e.g. "best paper for wedding photos" fans out into "archival photo paper," "matte vs glossy," "local high-resolution photo labs").
- **Synthesis (RAG):** it retrieves passages from many sources and stitches them into one custom answer. It is not matching your keyword, it is assembling a recommendation.
- Implication: you are no longer competing for one ranking, you are competing to be one of the trusted sources the answer is built from.

### Ranking vs being cited (important nuance)

The old goal was to rank in the list of links. The new goal is to be cited or recommended inside the AI answer. The new metric is **citation rate** (how often AI names you), not just rank. But the relationship to classic ranking differs by engine:

- **Google AI Overviews still lean heavily on top-ranked pages.** The majority of URLs cited in AI Overviews also rank in Google's top 10 (Ahrefs, 2025). Classic SEO still feeds Google's AI answers.
- **ChatGPT and Perplexity play by different rules.** A large share of their citations come from pages that do not rank in Google's top 10 (Ahrefs, 2025).
- Correct framing: strong SEO fundamentals plus AI-specific signals. Not one or the other. The blanket claim "ranking number one no longer matters" is overstated for Google and only broadly true for the chat engines.

### Scale (directional, verify before quoting)

- AI Overviews now appear on roughly a quarter of US searches, up from roughly half that a year earlier (Conductor, 2026).
- With an AI Overview present, click-through to the top result drops sharply (roughly 8% vs 15% without, Pew Research, 2025).
- Zero-click: more than 68% of US Google searches ended without a click in early 2026 (Jan–April), up from about 60% in 2024 (Adsroid, June 2026, citing market-intelligence panel data). Note this counts clicks to Google's own properties (Maps, YouTube) as non-clicks. The older "58.5%" SparkToro figure is 2024 and now outdated; use the 68% early-2026 figure, framed as "reported."
- Bots now generate the majority of web traffic: 57.5% of HTTP requests versus 42.5% human, the first machine majority, driven mainly by agentic AI (Cloudflare Radar, Matthew Prince, June 3, 2026). The through-line is "crawl without credit" — bots crawl heavily but send little referral traffic. Strong wake-up statistic. Caveat: do not pair it with the "ChatGPT-User/ClaudeBot crawl 3.6x more than Googlebot" claim — Cloudflare's own data still has Googlebot as the largest crawler by reach.
- AI-referred visitors tend to convert better than standard organic visitors because they arrive pre-qualified by the AI's recommendation. Reported multipliers vary too widely across sources to state a single figure; if a number is used, attribute it (e.g. Seer/Semrush) and frame as "reported," and do not show multiple conflicting figures together.

---

## Part B — Glossary (General)

### AEO (Answer Engine Optimization)
Structuring content so AI answer platforms select and cite it. Focuses on clear, extractable answers rather than keyword rankings. Often used interchangeably with GEO and AEO/LLMO.

### AI Mode
Google's conversational search experience powered by Gemini, allowing complex questions and follow-ups in the search interface, assembling custom answers from multiple background searches.

### AI Overviews
AI-generated summary blocks at the top of Google results that condense multiple sources into one answer with citation links.

### Citation rate
How frequently an AI engine references or links a specific brand as a source. Because AI pulls from different sources than classic search, tracking citation rate across platforms is the core AI-visibility metric.

### E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness)
Credibility signals Google and AI systems use to judge whether a source is safe to recommend. AI leans on external citations, established reputation, and expert authorship, especially for sensitive topics.

### GEO (Generative Engine Optimization)
The broad practice of structuring content and online presence so LLM-based engines cite or recommend you. Encompasses statistics, quotations, clear formatting, structured data, and authority signals.

### GPTBot
OpenAI's web crawler that gathers content for its models. Frequently blocked by publishers via robots.txt. For a business that wants to be recommended, it should be allowed, not blocked.

### llms.txt
A markdown file in the site root that describes site structure to AI agents. `[GENERAL]` Google has stated it does **not** directly affect visibility in AI search features. Treat as optional and low priority, not a required tactic.

### LLMO (Large Language Model Optimization)
Synonym for GEO. Managing content so LLMs understand and reference your brand.

### OKF (Open Knowledge Format)
`[GENERAL]` An open spec from Google Cloud (v0.1, published June 12, 2026) for representing organizational knowledge as a directory of markdown files with YAML frontmatter, one concept per file, linked as a graph. **Important: OKF is NOT a web-visibility or SEO tactic.** It is an internal knowledge bundle that your own AI agents read, not a public page format you publish for the web to crawl, and Google has stated plainly it is not a ranking or visibility mechanism. Do not present OKF to customers as a way to get cited by AI search. It is, however, relevant to Pixfizz internally — our own knowledge repo is essentially this pattern.

### Query fan-out
An engine breaking one prompt into multiple simultaneous sub-queries, then synthesizing the retrieved passages into one answer.

### SEO (Search Engine Optimization)
Traditional practice of optimizing to rank higher in standard results via on-page optimization, backlinks, and keywords. Wins clicks from a list of links rather than being the generated answer.

### Structured data / schema
Machine-readable markup (e.g. JSON-LD) that tells engines your entities explicitly (product name, price, availability, brand, business identity). Not strictly required to appear in AI search, but it helps AI extract facts accurately and keeps you eligible for rich results.

### WebMCP (Web Model Context Protocol)
`[GENERAL]` An emerging browser standard co-developed by Google and Microsoft via the W3C, first announced February 10, 2026, and in a public origin trial in Chrome as of June 2026. It lets a website expose its features to AI agents as structured, callable tools (a "Tool Contract") rather than agents guessing via screenshots. Two paths: a Declarative API (HTML attributes on forms) and an Imperative API (JavaScript). Model-agnostic. The "action layer" of AI readiness: schema gets a product understood, GEO gets a brand cited, WebMCP gets a product acted on and purchased by an agent. Status: early and Chrome-only; treat as "watch and early-adopt," not a do-it-now requirement.

### Zero-click search
When a user gets a complete answer on the results page without clicking through. Dominant in AI search, which is why citations and brand mentions matter more than raw click traffic.

---

## Part C — The GEO Playbook: The Five Signals (General)

A memorable framing for advising customers: AI looks for five signals. Be crawlable, be structured, be quotable, be trusted, be present locally.

### Signal 1 — Be crawlable
- If AI bots cannot read your site, you cannot be cited.
- Keep standard, clean HTML. Do not block AI crawlers (GPTBot, ClaudeBot, etc.) in robots.txt if you want to be recommended.
- Fast-loading, error-free pages are retrieved more reliably.

### Signal 2 — Be structured
- Schema gives AI clean facts without it having to interpret your layout: product name, price, availability, brand.
- Deploy where supported: Product, FAQPage, Article, LocalBusiness JSON-LD. Ensure structured data matches the visible text.
- Every product and page needs a populated description. Blank descriptions degrade feeds and trip structured-data validation.

### Signal 3 — Be quotable
- Open each section with a direct 40–60 word answer to a real question before the supporting detail. AI pulls heavily from the top of a page.
- Use FAQ-style question headings with short, factual answers that mirror how people prompt AI.
- Add specific numbers and cite sources. The original Princeton GEO research (KDD '24) found that adding statistics, citing sources, and adding quotations measurably lift AI visibility, while keyword stuffing performs worse than baseline (roughly 10% worse).
- Write for a human reading the answer aloud, not for an algorithm.

### Signal 4 — Be trusted
- AI treats reviews as a gate, not just a ranking factor. Locations ChatGPT recommends average above four stars (around 4.3, SOCi 2026); very low review response rates correlate with being effectively invisible to AI recommendations.
- Ask customers to describe the specific problem you solved, not just leave a star rating. AI lifts specific phrases from review text to justify a recommendation. Respond to reviews.
- Keep Name, Address, Phone (NAP) identical across your site, Google, and directories. Inconsistency erodes the AI's confidence to recommend you.
- Third-party mentions and earned media measurably lift citations. A meaningful share of AI citations come from sources other than your own site.

### Signal 5 — Be present locally
- Treat the Google Business Profile (GBP) as a data feed for AI. Fill every field, use the full business description, list every service and attribute.
- Add fresh photos and posts regularly. Stale profiles lose visibility.
- Never keyword-stuff the business name; it gets penalized and flagged as untrustworthy.

### Local nuance worth knowing
- Local-intent queries trigger an AI Overview far less often than general queries (around 8%, Ahrefs 2025). For local, the AI Overview on the results page is a smaller threat than whether you appear in the AI's *recommendation* at all.
- AI is currently far stingier than the classic map pack: ChatGPT names only ~1.2% of local business locations vs ~36% surfaced in Google's local 3-pack (SOCi 2026). For local discovery, a complete GBP and the map pack still do most of the heavy lifting; the chat engines and Google Ask Maps are the emerging battleground.

---

## Part D — Pixfizz and Shopper: What's Provided Today

This is the platform-truth section. Map each signal to what the platform and template actually do, and flag the gaps.

### Crawlable
- `[PLATFORM]` Built-in crawler at **Admin → Website Crawls** (`/admin/website_crawls`) generates `sitemap.xml` and `product-feed.json`. The `/site/sitemap` path does not exist; do not assume it.
- `[SHOPPER]` `no-index` checklist key set to `TRUE` adds a site-wide noindex. Confirm it is OFF on a live store.
- For sitemap submission to Google Search Console and 301 redirect configuration during migration, see `80_ONBOARDING.md` § SEO Migration.

### Structured data
- `[PLATFORM]/[SHOPPER]` **Product schema** is supported. Enable site-wide via the `schema_loop_all_products` checklist key (`TRUE` = include all products in schema). See `50_SHOPPER_TEMPLATE_REFERENCE.md` § SEO & Metadata.
- `[SHOPPER]` **LocalBusiness JSON-LD: placeholders exist** in the Shopper template. They are present but should not be assumed populated or emitted correctly by default. `[CONFIRM]` whether they are active out of the box, whether they include geo-coordinates and `sameAs`, and what configuration is required.
- `[GAP]` **Review / AggregateRating schema: not currently emitted.** There is on-page review display (Google rating/review fields, `Google_Summary` custom field with `rating` and `review_count`) but no review structured-data output. This is a build opportunity (see Pending Confirmation).
- `[CONFIRM]` **FAQPage, HowTo, Article JSON-LD**: output status unconfirmed. Do not tell customers Pixfizz emits these until confirmed.
- `[SHOPPER]` Per-object SEO meta fields exist: `meta_title` and `meta_description` on products, collections, subcollections, custom pages, blog posts, and services (see `51_CUSTOM_FIELDS_REFERENCE.md`). Site-level title/description via `update-website-title`, `update-website-description`, and the `seo-tdks` checklist key.
- `[SHOPPER]`/`[GENERAL]` **High-fidelity product attributes**: the current Product schema is a sound baseline but emits only a fraction of what Google now recommends. See **Part F** for the current baseline, Google's required-vs-recommended set, and prioritized Shopper upgrades. A standalone **Schema Builder** tool (`pixfizz-schema-builder.html`) generates paste-ready JSON-LD for a site-wide Organization/policies block and for a full single Product block.

### Quotable content
- `[SHOPPER]` Content surfaces available: blog (`blog`, `custom-blog-page`, `custom-blog-post` keys), custom pages, and a custom FAQ page (`custom-faq-page` key). These are where answer-first and FAQ-style content lives.
- Note the distinction: a custom FAQ *page* exists as a content surface; whether FAQPage *schema* is emitted from it is `[CONFIRM]` / likely a `[GAP]`.
- `[PLATFORM]` Populated product and page descriptions are required for feed and structured-data quality. See `80_ONBOARDING.md` § Content Completeness Before Launch.

### Trust and local
- `[SHOPPER]` Review display exists via Google rating fields. Review *generation* programs and GBP optimization are operational/marketing tasks for the client, not platform features.
- Product feed for Google Shopping / Meta is produced by the built-in crawler (`product-feed.json`). Google Merchant Center connection is a client-side setup task.

### Quick capability summary

| Signal | Pixfizz/Shopper status |
|---|---|
| Crawlable (sitemap, product feed) | Provided `[PLATFORM]` |
| Product schema | Provided, enable via `schema_loop_all_products` `[SHOPPER]` |
| Per-object SEO meta fields | Provided `[SHOPPER]` |
| LocalBusiness schema | Placeholders exist, not assumed active `[SHOPPER]` `[CONFIRM]` |
| Review / AggregateRating schema | Not emitted `[GAP]` |
| Return policy / shipping schema | Not emitted; best authored at Organization level, see Part F `[GAP]` |
| Organization / Brand / `sameAs` schema | Status unconfirmed; Schema Builder can author it `[CONFIRM]` |
| Product attribute fidelity (material, specs, variants) | Baseline only; upgrade tiers in Part F `[SHOPPER]` |
| FAQPage / HowTo / Article schema | Unconfirmed `[CONFIRM]` |
| Blog / FAQ / custom page surfaces | Provided `[SHOPPER]` |
| 301 redirects, Search Console submission | Provided, see `80_ONBOARDING.md` |
| llms.txt | Not provided; low priority per Google `[GAP]` |

---

## Part E — Measurement (General)

- Shift the scoreboard from rank and clicks to **citation rate** and **brand mention volume** across ChatGPT, Perplexity, Gemini, and Google AI surfaces.
- **Baseline test (repeatable):** run your 10–20 most important buyer queries through ChatGPT, Perplexity, and Google AI Mode. Note where you appear, how you are described, and which sources are cited. Re-run periodically; citation behavior shifts quickly and varies wildly between engines.
- **What AI already knows about you:** prompt an engine directly with "Tell me what you know about [business name]" to see its current understanding of your services, area, and reputation.
- Filter analytics referral traffic by AI sources (chatgpt.com, perplexity.ai, etc.) for the click-through slice, but expect most AI value to be zero-click.

### Platform measurement tools (current state, 2026)

- **Google Search Console — Search Generative AI performance reports** (launched June 3, 2026): a dedicated view showing how often your URLs appear in AI Overviews and AI Mode (and generative Discover), broken down by page, country, device, and date. Limitations: impressions only, no click or query data, not in the API yet (UI/CSV export only), and rolling out to a subset of properties first (initially UK). Use it to measure AI visibility, not traffic. This is the first time AI-surface visibility is separable from standard organic.
- **Google Search Console — AI controls toggle** (effective from June 17, 2026): lets a site opt its content out of AI Overviews, AI Mode, and generative Discover. Opting out forfeits impressions and any clicks from those features. Google states it is not a ranking signal for normal organic results. Almost no one should use this; relevant only as a thing that exists.
- **Bing Webmaster Tools — AI Performance report** (launched February 2026): tracks citations, the number of times your content is used to build a Copilot or Bing AI answer. Arguably more useful than GSC's impressions-only view because it counts actual citations. Worth pairing, since serious GEO measurement is cross-engine, not Google-only.

---

## Part F — Product Schema: High-Fidelity Attributes (Reference)

"High-fidelity" schema means the richest, most complete, and verifiably accurate Product markup the catalogue can support, rather than the minimum that triggers a rich result. It matters more now because AI shopping surfaces extract specific attributes (materials, dimensions, ratings, return and shipping terms) to answer specific buyer questions, and because richer Product, Brand, and Organization markup raises entity-confidence signals over time, not just snippet appearance. The catch: markup emits whatever the catalogue contains, including invalid identifiers and stale prices. Fidelity means complete **and** correct. Emitting a malformed attribute is worse than omitting it. Source: Google Search Central, Product / merchant-listing / product-snippet structured-data docs (2026); supporting industry analysis (2026), all directional.

### What Shopper emits today (baseline) `[SHOPPER]`

The auto-generated product JSON-LD already covers the Google-required trio and a good recommended set:

- `@type` Product with `name`, `description`, `sku`, `mpn` (falls back to `sku`), and a conditional `gtin` (emitted only when present).
- `image` array: live-preview WebP plus a fallback image.
- `brand` (Brand), `category` (from `google_category`), and `product_type` (from a custom field).
- `offers` (Offer) with `price` (or `from_pricing`), `priceCurrency`, `url`, `itemCondition` (`NewCondition`), `priceValidUntil`, and an `availability` mapping (`InStock` / `OutOfStock` / `InStoreOnly` when shipping is unavailable / `OnlineOnly` when pickup is unavailable).

This meets Google's required fields for a merchant listing (`name`, `image`, and an `offer` with `price` + `priceCurrency`). The opportunity is in the recommended layer below.

### Google's required vs recommended set `[GENERAL]`

- **Required (merchant listing):** `name`, `image`, and `offers` with `price` and `priceCurrency`. Merchant listings require an `Offer` (not `AggregateOffer`).
- **Recommended on Product:** `aggregateRating`, `review`, `brand.name`, `description`, the most specific `gtin8|gtin12|gtin13|gtin14` plus `mpn`/`sku`, `color`, `material`, `pattern`, `size` (or `SizeSpecification`), `audience` (`PeopleAudience`: suggested gender/age), `hasCertification` (energy labels etc., mainly EU), `isVariantOf` / `inProductGroupWithID` (variants), and `subjectOf` (a `3DModel`).
- **Recommended on Offer:** `availability`, `itemCondition`, `url`, `hasMerchantReturnPolicy`, `shippingDetails` (`OfferShippingDetails`), and `priceValidUntil`. Complex pricing uses `priceSpecification` with `UnitPriceSpecification` — `referenceQuantity` for per-unit pricing, `priceType: StrikethroughPrice` for sale prices, and `validForMemberTier` for loyalty/member prices.
- **Hard rules:** structured data must be present in the server HTML (not added by JavaScript after load), must match what the shopper sees, and identifiers must be valid. Each offer on a page needs a `sku` or `gtin` that matches the corresponding `product-feed.json` entry.

### Upgrade priorities for Shopper

**Tier 1 — high value, broadly applicable**
- `[GAP]` **`aggregateRating` + `review`.** Not emitted today, though `Google_Summary` already holds `rating` and `review_count`. This is the single biggest visible win (star ratings) and the "be trusted" GEO signal in concrete form. Only publish ratings that are visible on the page.
- `[GAP]` **`hasMerchantReturnPolicy` + `shippingDetails`.** Google expanded merchant-listing eligibility specifically around these. Best authored once at the Organization level, with per-offer overrides only where a product differs. Required return fields: `applicableCountry` and `returnPolicyCategory` (and `merchantReturnDays` when the window is finite).
- `[SHOPPER]` **`material`, `color`, `pattern`, `size`.** Cheap, factual attributes that AI extracts to match queries; relevant to apparel, gifting, and print verticals.

**Tier 2 — fit for the Pixfizz product model**
- `[GENERAL]`/`[SHOPPER]` **Product variants** (`ProductGroup` / `isVariantOf` / `inProductGroupWithID`) map onto Pixfizz option types, but this is a real design decision given personalization (see cautions).
- `[GENERAL]`/`[SHOPPER]` **`UnitPriceSpecification` + `referenceQuantity`** is the correct home for per-unit / tiered print pricing, rather than a single flat `price`.
- `[SHOPPER]` **`additionalProperty` (`PropertyValue`)** is the catch-all for specs with no dedicated field (paper stock, GSM, finish, page count, turnaround). This is where AI answers find the specifics.

**Tier 3 — entity grounding and GEO**
- `[CONFIRM]` **Organization + Brand + `sameAs`** ground the brand as a known entity; pairs with the LocalBusiness work already flagged.
- `[GAP]` **FAQPage** on product and Q&A pages: the "be quotable" signal in schema form.

### Pixfizz-specific cautions

- `[CONFIRM]` **Server-side rendering is mandatory for merchant listings.** Confirm the Shopper schema snippet renders in the initial server HTML, not via JavaScript after load. The same constraint forbids per-customer page changes (e.g. IP-based pricing).
- `[PLATFORM]` **Match visible content and the feed.** Schema values must match what the shopper sees, and each offer's `sku`/`gtin` must match `product-feed.json`.
- `[SHOPPER]` **GTIN validity.** Keep the conditional emission; an invalid GTIN actively hurts, and personalized/bespoke items legitimately have none. Decide how `identifier_exists`-style logic should behave for items with no GTIN.
- `[SHOPPER]` **`from_pricing` under-represents ranges.** A single `price` from a "from" value reads as a fixed price. For option-driven price ranges, decide deliberately between variant markup, `UnitPriceSpecification`, or an explicit starting-price framing. Note `AggregateOffer` is not for variants.
- `[GENERAL]` **Loyalty alignment.** The Offer pricing model now supports member pricing (`validForMemberTier`) and loyalty-program markup. Given the loyalty program on the roadmap, design the pricing schema so it does not need reworking later.

### Two correctness flags in the current snippet `[SHOPPER]`

- **`priceValidUntil` is computed inside a `{% dynamic %}` block.** Given the known platform behaviour where dynamic blocks can fail and render nothing, a failure here would emit `"priceValidUntil":` with no value and break the JSON for the entire product. Harden it or move the calculation out of the dynamic block.
- **`@context` uses `http://schema.org`.** Google accepts it, but `https://` is the current convention and matches Google's own examples.

### The Schema Builder tool

- A standalone HTML generator (`pixfizz-schema-builder.html`) produces paste-ready JSON-LD for (a) a site-wide **Organization + Brand + `sameAs` + return policy** block and (b) a full single **Product** block with the high-fidelity attributes above. It includes live validation hints and a copy button.
- Intended uses: author the Organization/policies block once per site (this is static data Shopper does not auto-build); test the target Product shape in the Rich Results Test; and hand-build product schema for the Shopify path or static products.
- It is an authoring aid, not a platform feature. The in-platform auto-generation of these formats (below) remains the larger opportunity.

---

## Pending Confirmation (for Matjaz / team)

- Shopper LocalBusiness JSON-LD: are the placeholders populated and emitted by default, and do they include geo-coordinates and `sameAs`? What configuration is required?
- FAQPage / HowTo / Article JSON-LD: does the platform or Shopper emit any of these today?
- Review / AggregateRating schema: confirmed not emitted; candidate build (auto-generate from existing review/`Google_Summary` data).
- llms.txt: low priority (Google states no direct AI-search visibility impact); decide whether to support at platform level as an agent-navigation aid only.
- Product schema rendering: does the Shopper product schema snippet render server-side (in the initial HTML), or could any of it depend on JavaScript? Merchant listings require server-side markup.
- `priceValidUntil` is built inside a `{% dynamic %}` block — confirm it is hardened so a silent dynamic-block failure cannot emit broken Product JSON.
- Return-policy and shipping values: where do they live (existing custom fields or new), and should they be Organization-level defaults or per-product overrides?
- Organization / Brand / `sameAs` schema: is any of this emitted today, or only the LocalBusiness placeholders?

## Knowledge Gaps / Build Opportunities (not yet built — do not present as live)

- Auto-generation of ideal structured-data formats (Product, LocalBusiness, Review/AggregateRating, FAQPage) from existing Shopper data, as a Shopper feature to improve AI visibility. Should extend to the high-fidelity Product layer in Part F (ratings, return/shipping policy, `additionalProperty`, variants, unit pricing). Under discussion; document only once live.
- Schema Builder tool (`pixfizz-schema-builder.html`): a built, standalone authoring aid that generates the Organization/policies block and a full Product block for paste and testing. Not a platform feature and not wired into Shopper; treat as an internal/merchant utility, distinct from the in-platform auto-generation opportunity above.
- WebMCP in Shopper: exposing storefront actions (search, configure product, add to cart) to browser AI agents via the Web Model Context Protocol. Emerging standard, Chrome origin trial only as of mid-2026. On the radar for Shopper; document only once built.

---

## Changelog

- 2026-06-15: Initial version. Created from webinar prep research (how AI search works, GEO playbook, glossary) and platform facts confirmed in-session: Shopper has LocalBusiness JSON-LD placeholders, review schema is not currently emitted, product schema via `schema_loop_all_products`. Cross-references 80 (SEO migration), 50 (checklist keys), 51 (custom fields). Source: claude-chat.
- 2026-06-18: Added verified 2026 stats (68% zero-click early 2026 per Adsroid; Cloudflare 57.5% bot-majority traffic, June 3 2026), with sourcing and caveats. Added glossary entries for WebMCP (real, emerging, Chrome origin trial) and OKF (real but internal-knowledge format, explicitly not an AI-visibility tactic — corrects an earlier framing). Added platform measurement tools: GSC Search Generative AI performance reports and AI controls toggle (June 2026), and Bing Webmaster Tools AI Performance/citation report (Feb 2026). Added WebMCP-in-Shopper to build opportunities. Source: claude-chat (web-verified).
- 2026-06-18: Added Part F (Product Schema: High-Fidelity Attributes), researched against Google Search Central's product / merchant-listing / product-snippet structured-data docs. Documents the current Shopper baseline, Google's required-vs-recommended set, prioritized Shopper upgrade tiers (Tier 1 ratings + return/shipping + attributes; Tier 2 variants, unit pricing, `additionalProperty`; Tier 3 Organization/Brand/`sameAs`, FAQPage), Pixfizz cautions (server-side rendering, match-visible-content, GTIN validity, `from_pricing`, loyalty/`validForMemberTier`), and two correctness flags on the current snippet (`priceValidUntil` inside a `{% dynamic %}` block; `http` vs `https` context). Added the standalone Schema Builder authoring tool. Updated the Quick capability summary, Pending Confirmation, and Build Opportunities to match. Source: claude-chat (web-verified against developers.google.com).
