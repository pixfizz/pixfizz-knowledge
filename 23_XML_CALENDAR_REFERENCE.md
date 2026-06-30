# 23 — XML Calendar & Planner Template Reference

**Authority Scope:** XML definition structure for calendar and planner products — date sequence vocabulary, set parameters, and annotated real-world examples. Platform-level — not Shopper-specific.

_Last updated: 2026-06-30_

---

## What this file covers

Calendar and planner products use an extended XML vocabulary beyond the standard page parameters documented in `19_XML_TEMPLATE_REFERENCE.md`. This file covers:

- `<definition>` root attributes
- `<set>` parameters
- The dates system — full element reference
- `<foreachdate>` — iteration over date sequences
- Named date sequence patterns (month grid, weekly, daily, etc.)
- Three annotated real-world examples

For standard page parameters (`width`, `height`, `bleed`, `margin`, `snap`, `hinge`, `gutter`, `output-name`), see `19_XML_TEMPLATE_REFERENCE.md`.

---

## 1. `<definition>` Attributes

The root element of every XML template definition.

| Attribute | Description |
|---|---|
| `unit` | Unit system for all dimension values. Common values: `inch`, `mm`. |
| `dpi` | Target resolution for production output. |
| `output` | Output file format. Common value: `pdf`. |
| `minimum-dpi` | Minimum acceptable image resolution. Used for image quality warnings in the Design Tool. |
| `add` | Minimum number of pages a user can add to the project. |
| `max` | Maximum total page count for the product. |

**Sample:**
```xml
<definition unit="inch" dpi="200" output="pdf" minimum-dpi="36" add="2" max="241">
```

---

## 2. `<set>` Parameters

A `<set>` groups one or more `<page>` elements that are treated as a unit. Sets can be repeated via `foreachdate`.

| Attribute | Values | Description |
|---|---|---|
| `count` | `false` / omit | `count="false"` excludes this set from the page count defined in `<definition>`. Commonly used for covers. Omit to include in count. |
| `grow` | `true` / omit | `grow="true"` marks this set as the one used when the end user adds new pages to a project. Typically used for interior pages in a book or planner. Only one set should have this. |
| `fulfillment` | `false` / omit | `fulfillment="false"` excludes this set from production artwork generation. Omit to include. |
| `editor` | `false` / omit | `editor="false"` hides this set from the end user in the Design Tool. Omit to show normally. |
| `preview` | `true` / omit | `preview="true"` designates this set as the project preview — the thumbnail shown to users in cart and saved projects. Not to be confused with a design preview. |
| `foreachdate` | Named date sequence | Repeats the set once for each date in the named sequence. See section 5. |

**Sample — hidden preview set:**
```xml
<set fulfillment="false" editor="false" preview="true">
	<page type="preview" bleed="0" width="10" height="10" />
</set>
```

---

## 3. Page Attributes Specific to Calendars

Two `<page>` attributes are specific to calendar/planner products:

| Attribute | Description |
|---|---|
| `dates` | Space-separated list of named date sequences to make available on this page. The design tool uses these to populate calendar grid elements. |
| `autofill` | **Legacy — no longer does anything. Ignore if seen in existing templates.** |

**Sample:**
```xml
<page type="calendar-month" dates="calendar-month previous-calendar-month next-calendar-month" bleed="0.125" width="11.25" height="8.75"/>
```

---

## 4. Dates Vocabulary

Date sequences are defined at the top of the `<definition>` block using a dedicated vocabulary. They are named and then referenced by pages and `foreachdate` iterations.

### `<dates name="...">`
Defines a named date sequence. The sequence is built by the child elements inside it.

### `<date />`
Adds a single date to the sequence. Accepts offset attributes: `day`, `month`, `year`.

```xml
<date day="+1" />
```

### `<defdate>`
Calculates a date and stores it under a name for later reference. Does not add it to the sequence — it is a named anchor only.

| Attribute | Description |
|---|---|
| `name` | The reference name to store the date under. |
| `day` | Day offset (e.g. `-1`, `+41`). |
| `month` | Month offset (e.g. `-1`, `+1`). |
| `year` | Year offset (e.g. `+1`). |
| `weekday` | Jump to the nearest occurrence of a weekday: `SU`, `MO`, `TU`, `WE`, `TH`, `FR`, `SA`. |

```xml
<defdate name="last-day-of-previous-month" day="-1" />
```

### `<dateshift>`
Shifts the current date context before executing child elements. All child date generation happens relative to the shifted position.

| Attribute | Description |
|---|---|
| `day` | Shift by N days. |
| `month` | Shift by N months. |
| `year` | Shift by N years. |
| `weekday` | Shift to the nearest occurrence of a weekday (`SU`, `MO`, etc.). |
| `to` | Jump to a previously stored `defdate` reference name. |

```xml
<dateshift to="sunday-of-first-week">
	<dategen freq="daily" count="42" />
</dateshift>
```

### `<dategen>`
Generates a sequence of dates and adds them to the current sequence.

| Attribute | Description |
|---|---|
| `freq` | Frequency: `daily`, `weekly`, `monthly`. |
| `count` | Number of dates to generate. |
| `until` | Generate dates up to and including this named `defdate` reference. |
| `interval` | Step interval (e.g. `interval="2"` = every 2 days). Default: `1`. |
| `tag` | Marks generated dates with a label. Used to identify out-of-range dates for styling. |

```xml
<dategen freq="daily" count="42" />
<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range" />
```

### `<deldategen>`
Removes dates from the current sequence. Accepts the same attributes as `<dategen>`.

### `<datesfrom name="...">`
Reuses another named date sequence inline. Avoids duplication when multiple sequences share the same base logic.

```xml
<dates name="calendar-year-month3">
	<dateshift month="3" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
```

### Tags
The `tag` attribute on `<dategen>` marks dates with a label. The most common use is `out-of-range` or `out-of-current-month` — marking dates that fall outside the current month so the design tool can style them differently (e.g. greyed out).

---

## 5. `<foreachdate>`

Two forms — both iterate over a named date sequence.

### As a `<set>` attribute
Repeats the set once per date in the sequence.

```xml
<set foreachdate="by-month">
	<page type="image" bleed="0.125" width="11.25" height="8.75"/>
	<page type="calendar-month" dates="calendar-month" bleed="0.125" width="11.25" height="8.75"/>
</set>
```

### As a wrapping block
Repeats everything inside it (multiple sets) once per date in the sequence. Used when a repeating unit contains more than one set.

```xml
<foreachdate name="by-month-3">
	<set>
		<page type="monthly-left" dates="calendar-month" width="7.4375" height="10.5625" />
		<page type="monthly-right" dates="calendar-month" width="7.4375" height="10.5625" />
	</set>
	<set foreachdate="by-day-monthly">
		<page type="daily-left" dates="calendar-day" width="7.4375" height="10.5625"/>
		<page type="daily-right" dates="calendar-day" width="7.4375" height="10.5625"/>
	</set>
</foreachdate>
```

`<foreachdate>` blocks can be nested — the inner `foreachdate="by-day-monthly"` on the set above generates a daily spread for every day within each iterated month.

---

## 6. Named Date Sequence Patterns

### Monthly grid — Sunday start
Generates 42 dates (6 weeks) starting from the Sunday of the first week. Marks dates outside the current month with `out-of-range`.

```xml
<dates name="calendar-month">
	<dateshift day="-6">
		<defdate weekday="SU" name="sunday-of-first-week"/>
	</dateshift>
	<dateshift to="sunday-of-first-week">
		<dategen freq="daily" count="42" />
	</dateshift>
	<defdate day="-1" name="last-day-of-previous-month"/>
	<dateshift to="sunday-of-first-week">
		<defdate day="+41" name="last-generated-date"/>
	</dateshift>
	<dateshift to="sunday-of-first-week">
		<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range"/>
	</dateshift>
	<dateshift month="+1">
		<dategen freq="daily" until="last-generated-date" tag="out-of-range"/>
	</dateshift>
</dates>
```

### Monthly grid — Monday start
Same logic but anchors to `weekday="MO"` instead of `weekday="SU"`.

```xml
<dates name="calendar-month">
	<defdate weekday="MO" name="monday-of-first-week"/>
	<dateshift to="monday-of-first-week">
		<dategen freq="daily" count="42" />
	</dateshift>
	<defdate day="-1" name="last-day-of-previous-month"/>
	<dateshift to="monday-of-first-week">
		<defdate day="+41" name="last-generated-date"/>
	</dateshift>
	<dateshift to="monday-of-first-week">
		<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range"/>
	</dateshift>
	<dateshift month="+1">
		<dategen freq="daily" until="last-generated-date" tag="out-of-range"/>
	</dateshift>
</dates>
```

### 12-month iteration sequence
Used with `foreachdate` to repeat a set or block once per month across 12 months.

```xml
<dates name="by-month">
	<dateshift day="1">
		<dategen freq="monthly" count="12" />
	</dateshift>
</dates>
```

### Previous and next month grids
Used when a calendar page displays the previous and next month in small mini-calendars alongside the current month.

```xml
<dates name="previous-calendar-month">
	<defdate name="end-of-pre-previous-month" month="-1" day="-1" />
	<defdate name="beginning-of-current-month" />
	<dateshift month="-1" day="-6">
		<dateshift weekday="SU">
			<defdate name="final-date" day="+37" />
			<dategen freq="daily" until="final-date" />
			<dategen freq="daily" until="end-of-pre-previous-month" tag="out-of-current-month" />
			<dateshift to="beginning-of-current-month">
				<dategen freq="daily" until="final-date" tag="out-of-current-month" />
			</dateshift>
		</dateshift>
	</dateshift>
</dates>

<dates name="next-calendar-month">
	<defdate name="end-of-current-month" month="+1" day="-1" />
	<defdate name="beginning-of-next-next-month" month="+2" />
	<dateshift month="+1" day="-6">
		<dateshift weekday="SU">
			<defdate name="final-date" day="+37" />
			<dategen freq="daily" until="final-date" />
			<dategen freq="daily" until="end-of-current-month" tag="out-of-current-month" />
			<dateshift to="beginning-of-next-next-month">
				<dategen freq="daily" until="final-date" tag="out-of-current-month" />
			</dateshift>
		</dateshift>
	</dateshift>
</dates>
```

### Year-at-a-glance — 24 named month sequences
For products that display many months simultaneously. Each named sequence reuses `calendar-month` via `<datesfrom>`, shifted to the target month.

```xml
<dates name="year-at-a-glance">
	<date/>
</dates>
<dates name="calendar-year-month1">
	<dateshift month="1" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
<!-- ... repeat for month2 through month12 ... -->
<!-- For a second year, use year="+1": -->
<dates name="calendar-year-month13">
	<dateshift year="+1" month="1" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
```

The `year-at-a-glance` sequence is a single date (`<date/>`). When used with `<set foreachdate="year-at-a-glance">`, it causes the set to run exactly once. The page inside it receives all 12 (or 24) month sequences via its `dates` attribute.

---

## 7. Annotated Examples

### Example A — Standard 12-month wall calendar

The most common calendar type. Cover + 12 × (image page + calendar grid page) + back cover.

```xml
<definition output="pdf" unit="inch" dpi="300">

	<dates name="calendar-month">
		<!-- Sunday-start monthly grid — see section 6 -->
	</dates>

	<dates name="by-month">
		<dateshift day="1">
			<dategen freq="monthly" count="12" />
		</dateshift>
	</dates>

	<dates name="previous-calendar-month">
		<!-- See section 6 -->
	</dates>

	<dates name="next-calendar-month">
		<!-- See section 6 -->
	</dates>

	<set>
		<page type="cover" bleed="0.125" width="11.25" height="8.75"/>
	</set>

	<set foreachdate="by-month">
		<page type="image" bleed="0.125" width="11.25" height="8.75"/>
		<page type="calendar-month" dates="calendar-month previous-calendar-month next-calendar-month" bleed="0.125" width="11.25" height="8.75"/>
	</set>

	<set>
		<page type="back-cover" bleed="0.125" width="11.25" height="8.75"/>
	</set>

</definition>
```

**Key points:**
- No `count="false"` on the cover set — cover is counted here.
- `by-month` generates 12 dates so the `foreachdate` set repeats 12 times, producing 24 interior pages total.
- `previous-calendar-month` and `next-calendar-month` give the calendar page mini-calendars for the surrounding months.

---

### Example B — Year-at-a-glance product

A single-page product displaying all 12 months of a year simultaneously.

```xml
<definition unit="inch" dpi="300" output="pdf">

	<dates name="year-at-a-glance">
		<date/>
	</dates>

	<!-- 12 named month sequences, each reusing calendar-month via datesfrom -->
	<dates name="calendar-year-month1">
		<dateshift month="1" day="1"><datesfrom name="calendar-month"/></dateshift>
	</dates>
	<!-- ... month2 through month12 ... -->

	<dates name="calendar-month">
		<!-- Sunday-start monthly grid — see section 6 -->
	</dates>

	<layers>
		<layer name="artwork" visibility="on"/>
		<layer name="background" visibility="on"/>
		<layer name="photo" visibility="on"/>
	</layers>

	<!-- Hidden preview set — not fulfilled, not shown in editor -->
	<set fulfillment="false" editor="false" preview="true">
		<page type="preview" bleed="0" width="10" height="10" />
	</set>

	<set foreachdate="year-at-a-glance">
		<page type="product"
			dates="calendar-year-month1 calendar-year-month2 ... calendar-year-month12"
			bleed="0" width="11" height="8.5" />
	</set>

</definition>
```

**Key points:**
- `year-at-a-glance` is a single date — `foreachdate` runs the set exactly once.
- All 12 month sequences are passed to the single page via the `dates` attribute.
- The hidden preview set uses `fulfillment="false" editor="false" preview="true"` — a common pattern for products that need a custom project thumbnail without it appearing in the editor or production output.
- Layers defined here are assigned to elements in the admin design tool.

---

### Example C — 3-month planner (complex nested structure)

A planner product with fixed front/back matter, a year-at-a-glance spread, and a 3-month repeating block containing monthly and daily pages.

```xml
<definition unit="inch" dpi="200" output="pdf" minimum-dpi="36" add="2" max="241">

	<!-- Date sequences: year-at-a-glance, 24 named month sequences,
	     calendar-month, previous/next month, by-month-3,
	     by-day-monthly, calendar-day — see section 6 -->

	<!-- Cover — not counted -->
	<set count="false">
		<page type="cover" width="10.3" height="12" />
	</set>

	<!-- Fixed front matter -->
	<set>
		<page type="profile" width="7.4375" height="10.5625" />
	</set>
	<set>
		<page type="logo" width="7.4375" height="10.5625"/>
		<page type="intro" width="7.4375" height="10.5625"/>
	</set>

	<!-- Year-at-a-glance — runs once, receives all 24 month sequences -->
	<set foreachdate="year-at-a-glance">
		<page type="holidays2" width="7.4375" height="10.5625" />
		<page type="at-a-glance"
			dates="calendar-year-month1 ... calendar-year-month24"
			width="7.4375" height="10.5625" />
	</set>

	<!-- Growable contacts section -->
	<set grow="true">
		<page type="contacts-left" width="7.4375" height="10.5625" />
		<page type="contacts-right" width="7.4375" height="10.5625" />
	</set>

	<!-- Vision board spread -->
	<set>
		<page type="vision-board-left" width="7.4375" height="10.5625" bleed="0.125" margin="0.125" snap="0,0.125,0.125"/>
		<page type="vision-board-right" width="7.4375" height="10.5625" bleed="0.125" margin="0.125" snap="0,0.125,0.125"/>
	</set>

	<!-- 3-month repeating block — repeats 3 times -->
	<foreachdate name="by-month-3">

		<set>
			<page type="intentions-left" width="7.4375" height="10.5625"/>
			<page type="intentions-right" width="7.4375" height="10.5625"/>
		</set>

		<set>
			<page type="monthly-left" dates="calendar-month previous-calendar-month next-calendar-month" width="7.4375" height="10.5625" />
			<page type="monthly-right" dates="calendar-month zodiac-quote-date" width="7.4375" height="10.5625" />
		</set>

		<set>
			<page type="health-left" width="7.4375" height="10.5625" />
			<page type="health-right" width="7.4375" height="10.5625" />
		</set>

		<!-- Daily spreads — one set per day in the month -->
		<set foreachdate="by-day-monthly">
			<page type="daily-left" dates="calendar-day" bleed="0" width="7.4375" height="10.5625"/>
			<page type="daily-right" dates="calendar-day" bleed="0" width="7.4375" height="10.5625"/>
		</set>

	</foreachdate>

	<!-- Fixed back matter -->
	<set><page type="passwords-left" width="7.4375" height="10.5625" /><page type="passwords-right" width="7.4375" height="10.5625" /></set>
	<set><page type="mood-left" width="7.4375" height="10.5625" /><page type="mood-right" width="7.4375" height="10.5625" /></set>
	<set><page type="notes-left" width="7.4375" height="10.5625" /><page type="notes-right" width="7.4375" height="10.5625" /></set>

</definition>
```

**Key points:**
- `<foreachdate name="by-month-3">` wraps multiple sets — all repeat 3 times.
- `foreachdate="by-day-monthly"` is nested inside the outer `foreachdate` block. It generates a daily spread for every day of each of the 3 months.
- `grow="true"` on the contacts set allows users to add more contact pages.
- `count="false"` on the cover excludes it from the page count.
- The 6-month and 12-month planner variants in the original definition are commented out — they represent alternative structures (6-month daily, 12-month weekly) that can be swapped in by uncommenting and commenting out the 3-month block.

---

## 8. Legacy Attributes

The following attributes appear in older template definitions and should be ignored — they no longer do anything:

| Attribute | Notes |
|---|---|
| `autofill` | Was intended to auto-populate image zones. No longer functional. |
| `reorder` | No longer functional. |

---

## Calendar Transformation Triggers

Calendar transformations (the admin configuration that maps calendar dates to outputs/designs) can be triggered on:

- a specific date or date range, and
- **a specific week number in the year** (added June 2026).

Week-number triggering fires a transformation on a given week (for example week 1 or week 52) rather than a fixed calendar date.

## Changelog
- 2026-04-03: Created from platform documentation and annotated real-world examples provided by AdeB. Covers definition attributes, set parameters, full dates vocabulary, foreachdate, named sequence patterns, and three annotated examples.
- 2026-06-30: Documented week-number triggering for calendar transformations. Source: notion-dashboard (2026-06-22).
