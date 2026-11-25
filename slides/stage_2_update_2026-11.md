---
marp: true
theme: default
paginate: true
footer: 'Intl Sequence Units - Stage 2 Update'
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
  table {
    font-size: 0.68em;
  }
---

<!-- 
_class: lead 
_backgroundColor: #2c3e50
_color: #ffffff
-->

# Intl Sequence Units
## Stage 2 Update
### Shane F Carr — November 2026 TC39

---

## Recap: Proposal & Sept 2026 Updates

<div class="columns">
<div>

Formats sanctioned `-and-` unit sequences in `Intl.NumberFormat`.

**Sept 2026 Updates:**
- **#16:** Object → Array input
- **#14:** Disallow sigfig options
- **#8:** Exclude time units

</div>
<div>

```javascript
const nf = new Intl.NumberFormat("en-US", {
  style: "unit",
  unit: "foot-and-inch",
});

nf.format([5, 11]);
// "5 feet, 11 inches"

nf.format([-5, -11]);
// "-5 feet, 11 inches"
```

</div>
</div>

---

## Issue #20: Precedents for Reading List Inputs

Four list-reading patterns across ECMA-262 & 402 ([#19 audit](https://github.com/tc39/proposal-intl-sequence-units/issues/19#issuecomment-6875324113)):

| Pattern | Spec Precedents | TypedArray | Iterable | `"56"` |
| :--- | :--- | :---: | :---: | :---: |
| **Object + `GetIterator`** | `Iterator.zip` | ✅ | ✅ | ✅ `TypeError` |
| **General `GetIterator`** | `Intl.ListFormat`, `Promise.all` | ✅ | ✅ | 😕 `"5"`, `"6"` |
| **`LengthOfArrayLike`** | `CanonicalizeLocaleList`, `apply` | ✅ | ❌ | ✅ / varies |
| **`IsArray` + indexed** | `JSON.stringify` *(branch only)* | ❌ | ❌ | ✅ `TypeError` |
| **Hybrid** | `Array.from`, `%TypedArray%.from` | ✅ | ✅ | 😕 `"5"`, `"6"` |

---

## Why Reject Primitives Before `GetIterator`?

```javascript
const ft = new Intl.NumberFormat("en", { style: "unit", unit: "foot" });
ft.format("56"); // "56 feet" (numeric strings coerced)

const fi = new Intl.NumberFormat("en", {
  style: "unit", unit: "foot-and-inch"
});
fi.format("56");
// General GetIterator:  "5 feet, 6 inches" ❌ (iterates chars!)
// Object + GetIterator: TypeError          ✅
```

- Matches `Intl.ListFormat` + `Iterator.zip` (rejecting primitives)
- Leaves primitives free for future scalar extensions ([#12](https://github.com/tc39/proposal-intl-sequence-units/issues/12))

---

## TG2 Recommendation: `ToSequenceUnitList` (#20)

```
1. If value is not an Object, throw a TypeError exception.
2. Let sequence be StringSplitToList(unit, "-and-").
3. Let iteratorRecord be ? GetIterator(value, ~sync~).
4. Let iterableValues be ? IteratorToList(iteratorRecord).
5. If length of iterableValues ≠ length of sequence, throw RangeError.
6. Convert each via ? ToIntlMathematicalValue (undefined → TypeError),
   requiring matching signs & integer non-last values (→ RangeError).
```

- Accepts `Array`, TypedArrays, `Set`, and custom iterables
- *Note: drains via `IteratorToList` (vs. bounded `IteratorClose`)*

---

## Summary & Next Steps

- **Open Question (#20):** Any concerns with using **Object-only `GetIterator`** in `ToSequenceUnitList`?
- **Draft Spec:** Complete at [tc39.es/proposal-intl-sequence-units](https://tc39.es/proposal-intl-sequence-units/)
- **Stage 2.7 Reviewers:** Eemeli Aro (EAO) and Dan Minor (DLM)
- **Next Steps:** Incorporate plenary & reviewer feedback toward Stage 2.7
