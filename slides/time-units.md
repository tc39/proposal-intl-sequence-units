---
marp: true
theme: default
paginate: true
footer: 'TC39 Intl Sequence Units - Issue #8'
---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->

# Should We Support Time Units?
## Architectural Discussion for `Intl.NumberFormat` & `Amount`
### TC39 Plenary / TG1 Discussion — Issue #8

---

## Background: What are Sequence Units?

Measurement systems frequently employ multiple units in sequence to express a single magnitude.

### Existing Sanctioned Sequences (Table 1)
- **Length:** `foot-and-inch`, `meter-and-centimeter`
- **Mass:** `pound-and-ounce`, `kilogram-and-gram`

### What about Time & Angles?
- **Time:** `hour-and-minute`, `minute-and-second`, `day-and-hour`
- **Angles:** `degree-and-arcminute`, `arcminute-and-arcsecond`

Currently, time units are **excluded** from the sanctioned sequence units table in `proposal-intl-sequence-units`.

---

## The Architectural Dilemma

When a developer works with time sequences (like hours and minutes):

1. Should they be able to format them using general sequence formatting in **`Intl.NumberFormat`** and **`Amount`**?
   ```javascript
   const nf = new Intl.NumberFormat("en", { style: "unit", unit: "hour-and-minute" });
   nf.format({ hour: 2, minute: 30 }); // Should this work or throw?
   ```

2. Or should the platform strictly direct them to **`Intl.DurationFormat`** and **`Temporal.Duration`**?
   ```javascript
   const df = new Intl.DurationFormat("en", { style: "long" });
   df.format(Temporal.Duration.from({ hours: 2, minutes: 30 }));
   ```

---

## The Two Perspectives in TG2

In recent TG2 discussions (June & July 2026), two distinct architectural viewpoints emerged regarding proposal scoping and platform consistency:

- **Position 1: Exclude Time Units (Domain Dedication)**
  *Advocated by Shane F. Carr (@sffc)*
  Focus on guiding developers to purpose-built, domain-specific APIs.

- **Position 2: Include Time Units (Data & API Consistency)**
  *Advocated by Richard Gibson (@gibson042) & Eemeli Aro (@eemeli)*
  Focus on alignment with CLDR data and interoperability with `Amount`.

---

<!-- _class: lead -->
# Position 1: Exclude Time Units
## Prioritize Domain-Dedicated APIs

---

## Position 1: Why Exclude Time Units? (1/2)

### 1. Dedicated Domain-Specific API Already Exists
- TC39 invested significant effort designing **`Intl.DurationFormat`** and **`Temporal.Duration`** specifically for time and duration formatting.
- `DurationFormat` provides domain-essential features: balancing, normalization, and digital formatting styles (e.g., `2:30:00`) that general sequence formatting lacks.
- Providing a secondary, less-capable way to format durations in `Intl.NumberFormat` creates developer confusion and API redundancy.

### 2. Hidden Complexities & Pitfalls of Time
- Time units involve inherent domain hazards: combining `day-and-hour` crosses timezone / DST boundaries; `month-and-day` depends on calendar month lengths.
- Even if simple pairs like `minute-and-second` seem harmless, allowing time units in general sequence formatting invites subtle bugs.

---

## Position 1: Why Exclude Time Units? (2/2)

### 3. Clean Alignment Between Data & Formatter APIs
- **`Intl.DurationFormat`** has a 1-to-1 correspondence with **`Temporal.Duration`**.
- **`Intl.NumberFormat`** should align cleanly with **`Amount`**.
- Creating time-based `Amount` sequences that duplicate `Temporal.Duration` functionality blurs architectural boundaries.

### 4. Curated Web Platform Capabilities vs. Raw CLDR Data
- While CLDR (`units.xml`) includes time unit combinations, ECMA-402 is not obligated to blindly expose every combination in CLDR.
- The web platform should curate its surface area to guide developers toward the correct tools (*"We don't want to add bad time into the web platform"*).

---

<!-- _class: lead -->
# Position 2: Include Time Units
## Prioritize Data & API Consistency

---

## Position 2: Why Include Time Units? (1/2)

### 1. Alignment with CLDR and `Amount` Conversion
- Sequence units and unit conversions in `Amount` are defined in terms of CLDR data and usage preferences (which include `minute-and-second` for media duration and person age).
- Because `Amount.prototype.convertTo` can produce `minute-and-second` objects, it is arbitrary and ergonomic friction if those valid `Amount` objects throw errors when passed to `Intl.NumberFormat`.

### 2. Flexibility / Multiple Valid Workflows
- While `DurationFormat` is ideal for dedicated duration handling, it should not be the *only* permitted approach enforced by artificial restrictions.
- When doing unit conversions within a category (e.g., via an `Amount` conversion form), developers should be able to format their results directly without converting back and forth to `Temporal.Duration`.

---

## Position 2: Why Include Time Units? (2/2)

### 3. Not a Formatting Hazard
- Sequence formatting does not perform calendar or time math; it simply formats quantities together (like feet-and-inches or pounds-and-ounces).
- Formatting a valid sequence like `hour-and-minute` is fundamentally just list formatting with unit styles, which poses no inherent architectural danger.

### 4. Dedicated Rendering Needs
- Certain unit sequences require specialized syntax or localization symbols beyond simple list formatting (e.g., degree-arcminute-arcsecond or minute-second notation: `° ' "`).
- Sequence formatting provides the necessary localization machinery for these representations.

---

## Summary Comparison

| Aspect | Position 1: Exclude Time Units | Position 2: Include Time Units |
| :--- | :--- | :--- |
| **Primary Philosophy** | Guide developers to domain-dedicated tools (`DurationFormat`). | Maintain consistency across CLDR data, `Amount`, and formatters. |
| **View on Duplication** | Redundant with `DurationFormat` / `Temporal.Duration`. | Legitimate alternative workflow for general measurement handling. |
| **Handling of Hazards** | Prevent bugs by blocking complex units (TZ / DST / calendar shifts). | Sequence formatting is just presentation; math hazards belong elsewhere. |
| **CLDR Alignment** | ECMA-402 should curate a safe subset of CLDR. | If CLDR & `Amount` support it, `Intl.NumberFormat` should format it. |

---

## Discussion & Questions for TG1

1. **Platform Guidance vs. Developer Flexibility:**
   Should TC39 actively restrict general-purpose APIs to force adoption of domain-specific APIs (e.g., blocking time units in `Intl.NumberFormat` to favor `Intl.DurationFormat`)?

2. **Interoperability between `Amount` and `Intl.NumberFormat`:**
   If `Amount` unit conversion supports CLDR usage preferences (like `minute-and-second`), should `Intl.NumberFormat` be guaranteed to format any valid `Amount` sequence?

3. **Subsetting Strategy:**
   What is the appropriate level of strictness for Table 1 (Sanctioned Sequence Units) as `proposal-intl-sequence-units` progresses toward Stage 3?

---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->

# Thank You!
## Discussion / Q&A
