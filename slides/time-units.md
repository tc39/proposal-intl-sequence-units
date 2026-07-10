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

**Example sanctioned units:**

- Length: `foot-and-inch`, `meter-and-centimeter`
- Mass: `pound-and-ounce`, `kilogram-and-gram`

Duration units are currently excluded from the list of sanctioned units:

- Time: `hour-and-minute`, `minute-and-second`, `day-and-hour`

*In this presentation I will use "time units" and "duration units" interchangeably*

---

## Time Units in Amount vs Temporal.Duration

Developers can already use Temporal.Duration and Intl.DurationFormat:

```javascript
const formatter = new Intl.DurationFormat("en", { style: "long" });
formatter.format({ hours: 2, minutes: 30 });

const duration = Temporal.Duration.from({ hours: 2, minutes: 30 });
formatter.format(duration);
```

What should happen if they try to use Amount and/or Intl Sequence Units?

```javascript
const formatter = new Intl.NumberFormat("en", { style: "unit", unit: "hour-and-minute" });
formatter.format({ hour: 2, minute: 30 });

const amount = new Amount({ hour: 2, minute: 30 }, "hour-and-minute");
formatter.format(amount);
```

---

## The Two Perspectives in TG2

In recent TG2 discussions (June & July 2026), two distinct architectural viewpoints emerged regarding proposal scoping and platform consistency:

- **Position 1: Exclude Time Units**
  Focus on guiding developers to purpose-built, domain-specific APIs.

- **Position 2: Include Time Units**
  Focus on alignment with CLDR data and ease of use with `Amount`.

---

<!-- _class: lead -->
# Position 1: Exclude Time Units
## Prioritize Domain-Dedicated APIs

---

## Position 1: Why Exclude Time Units? (1/3)

### 1. Duration units have footguns

- `day-and-hour` crosses timezone / DST boundaries
- `month-and-day` depends on calendar month lengths

### 2. Durations are complicated to format

- Digital formatting styles (`2:30:00`)
- Flexible handling of zero-valued fields

> **Note:** Domain-specific functionality impacts both large (month, day) and small (minute, second) duration units.

---

## Position 1: Why Exclude Time Units? (2/3)

### 3. The Web Platform should promote the right way to do things

- We just spent 9 years designing Temporal
- We shouldn't add a worse way to represent and format time units

### 4. Adding duration unit formatting to Intl.NF will make developers expect formatting options there

- Six years ago, we explicitly decided to make a separate Intl.DurationFormat instead of overloading Intl.NumberFormat with duration formatting options and functionality
- If we add duration unit formatting only now, then the lack of duration-specific features in Intl.NumberFormat will be a pain point for years to come

---

## Position 1: Why Exclude Time Units? (3/3)

### 5. Amount is already more general than Intl.NumberFormat

- Amount will allow arbitrary units, like `jupiter-radius`
- Intl will throw when formatting these non-sanctioned units
- So even if Amount supports time units, Intl.NF need not

### 6. Intl is already a subset of CLDR

- CLDR has over 200 units; Intl has 45, each independently motivated
- Intl is already sanctioning a narrow subset of sequence units
- ECMA need not add time sequence units just because they are in CLDR

---

<!-- _class: lead -->
# Position 2: Include Time Units
## Prioritize Flexibility and Consistency with CLDR

---

## Position 2: Why Include Time Units? (1/2)

### 1. Alignment with CLDR and `Amount` Conversion

- Sequence units and unit conversions in `Amount` are defined in terms of CLDR.
- There are usage preferences like "duration-media" that use time units.
- If `Amount.prototype.convertTo` can produce `minute-and-second` objects, it is arbitrary and ergonomic friction if those valid `Amount` objects throw errors when passed to `Intl.NumberFormat`.

---

## Position 2: Why Include Time Units? (2/2)

### 2. Flexibility / Multiple Valid Workflows

- While `DurationFormat` is ideal for dedicated duration handling, it should not be the *only* permitted approach enforced by artificial restrictions.
- When doing unit conversions within a category (e.g., via an `Amount` conversion form), developers should be able to format their results directly without converting back and forth to `Temporal.Duration`.

### 3. Not a Formatting Hazard

- Sequence formatting does not perform calendar or time math; it simply formats quantities together (like feet-and-inches or pounds-and-ounces).
- Formatting a valid sequence like `hour-and-minute` is fundamentally just list formatting with unit styles, which poses no inherent architectural danger.

---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->
# Next Steps

---

## Open Questions for Discussion

**Intl Sequence Units:**

1. Should Intl.NumberFormat include time units as sanctioned sequence units?

**Amount:**

1. Should Amount support single time units?
    - Example: `new Amount(5, "hour")`
2. Should Amount support sequence time units?
    - Example: `new Amount({ hour: 2, minute: 30 }, "hour-and-minute")`
3. Should Amount support time unit conversions?
    - Example: `new Amount(5.5, "hour").convertTo("hour-and-minute")`
