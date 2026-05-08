---
marp: true
theme: default
paginate: true
# header: 'TC39 Intl Sequence Units'
footer: 'TC39 Intl Sequence Units'
---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->

# TC39 Intl Sequence Units Proposal

## Shane F Carr, 114th TC39, May 2026

---

## Background

Measurement systems frequently employ multiple units in sequence to express a single magnitude. 

### Examples

- Person height: "5 feet, 11 inches" or "1 meter, 80 centimeters"
- Mass: "2 pounds, 4 ounces"

---

## The Problem

- Currently, developers must manually compose multiple `Intl.NumberFormat` outputs, leading to potential for error.
- In the Amount proposal, we want an all-in-one formatter to reduce the amount of logic available only in `Amount.prototype.toLocaleString`.

---

<!-- _class: lead -->

# Stage 1?

---

## Proposed Solution: Extend `Intl.NumberFormat`

`Intl.NumberFormat` is extended to support compound unit identifiers joined by the `-and-` separator.

When a sequence unit is specified, `format` accepts a JavaScript object mapping sub-units to magnitudes.

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// "5 feet, 11 inches"
nf.format({ foot: 5, inch: 11 });
```

---

## Prior Art: CLDR / UTS #35

**Unicode Technical Standard #35 (LDML)** defines the the unit identifier syntax and behavior in the section for [Unit Sequences (Mixed Units)](https://unicode.org/reports/tr35/tr35-general.html#Unit_Sequences).

UTS #35 specifies the use of localized `listPattern` data to correctly compose mixed unit sequences in different languages. This proposal implements those TR35 guidelines natively for ECMAScript.

---

## Negative Numbers

- All input fields must share the same sign.
- Like Intl.DurationFormat, only the first formatted unit in the sequence renders the negative sign.

```javascript
// To format a negative measure, all fields must have the same sign:
nf.format({ foot: -5, inch: -11 }); 
// "-5 feet, 11 inches"

// RangeError: sub-units have mixed signs
nf.format({ foot: 5, inch: -11 });
```

---

## Comparison with Intl.DurationFormat

Because we have Intl.DurationFormat, time units are out of scope of this proposal.

Some key differences between this proposal and Intl.DurationFormat:

| Feature | `Intl.DurationFormat` | `Intl.NumberFormat` Sequence Units |
|---------|-----------------------|------------------------------------|
| **Digital Style** | Yes: "1:23:45" | N/A |
| **Fields Read** | All duration unit fields | Only the fields in the specific unit ID |

---

## Edge Cases and Error Handling

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit', unit: 'foot-and-inch'
});

// TypeError: 'inch' property is missing
nf.format({ foot: 5 }); 

// OK, but extra fields are ignored
nf.format({ yard: 3, foot: 1, inch: 5 }); // "1ft 5in"

// RangeError: intermediate sub-unit is not an integer
nf.format({ foot: 5.5, inch: 6 });

// RangeError: invalid unit sequence
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'meter-and-foot' });
```

---

## Sanctioned Sequence Unit Groups

For better error handling and more future flexibility, we are restricting the sequence units to the following groups:

- `"mile"`, `"yard"`, `"foot"`, `"inch"`
- `"kilometer"`, `"meter"`, `"centimeter"`, `"millimeter"`
- `"stone"`, `"pound"`, `"ounce"`
- `"kilogram"`, `"gram"`
- `"gallon"`, `"fluid-ounce"`
- `"liter"`, `"milliliter"`

---

## Alternative: Scalar Input with Automatic Conversion

Example: `6.25` instead of `{ foot: 6, inch: 3 }`.

TG2 considered this design and decided against it:

- **Input Ambiguity:** In `foot-and-inch`, does `6.5` refer to feet or inches?
- **Incomplete Conversion Data:** The engine may not have conversion factors for all current, custom, or future unit combinations.
- **Precision and Rounding:** Floating-point arithmetic can introduce errors during automated partitioning.

---

<!-- _class: lead -->

# Stage 2?
