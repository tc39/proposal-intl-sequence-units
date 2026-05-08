---
marp: true
theme: default
paginate: true
header: 'Intl.SequenceUnits Proposal'
footer: 'TC39 Proposal - Stage [X]'
---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->

# **Intl.SequenceUnits**
## Formatting sequences of units in JavaScript

---

## The Motivation

Currently, `Intl.UnitFormat` can format a single unit (e.g., "5 hours").
However, there is no standard way to format a **sequence** of units:

- "1 hour, 30 minutes, and 5 seconds"
- "5 feet, 10 inches"
- "3 kilograms, 500 grams"

### The Problem
Developers currently have to manually join multiple `Intl.UnitFormat` outputs using `Intl.ListFormat`, which:
1. Is verbose.
2. Might not handle locale-specific unit sequence rules correctly.

---

## Proposed API

```javascript
const su = new Intl.SequenceUnits("en-US", {
  style: "long", // "short", "narrow"
  listStyle: "wide", // "conjunction", "disjunction", "unit"
});

su.format([
  { value: 1, unit: "hour" },
  { value: 30, unit: "minute" }
]); 
// Output: "1 hour and 30 minutes"
```

---

## Use Cases

### Duration & Time
- "2 hours, 15 minutes"
- "3 days, 4 hours"

### Measurements
- "6 feet, 2 inches"
- "10 stones, 4 pounds"

### Physical Quantities
- "5 kilograms, 200 grams"

---

## Customization

The proposal aims to support different styles and list joining behaviors:

- **Long:** "1 hour, 30 minutes"
- **Short:** "1 hr, 30 min"
- **Narrow:** "1h 30m"

And different list formats:
- **Conjunction:** "1 foot and 2 inches"
- **Unit (typical for measurements):** "1 foot 2 inches"

---

<!-- _class: lead -->

# Current Status & Next Steps

- [ ] Stage 1: Proposal
- [ ] Stage 2: Draft
- [ ] Stage 3: Candidate
- [ ] Stage 4: Finished
