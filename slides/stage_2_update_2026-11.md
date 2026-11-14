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

## Proposal Recap

<div class="columns">
<div>

Extends `Intl.NumberFormat` to format sanctioned `-and-` unit sequences.

**September 2026 Updates:**
- **#16:** Switched input from object to array
- **#14:** Disallowed significant digit options (`RangeError`)
- **#8:** Excluded time sequence units from `Intl.NumberFormat`

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

## Issue #20: How to Read the List of Units?

With `format()` and `formatToParts()` accepting `[5, 11]`, there are three conventions in ECMA-262 / ECMA-402 for reading list inputs:

1. **Iterator protocol** (`GetIterator` + `IteratorToList`)
   - Precedent: `Intl.ListFormat` (`StringListFromIterable`)
2. **Array-like** (`LengthOfArrayLike` + indexed `Get`)
   - Reads `.length` and properties `"0"` .. `length - 1`
3. **Strict Array** (`IsArray` + indexed `Get`)
   - Rejects non-`Array` objects with `TypeError`

---

## TG2 Recommendation: Iterator Protocol (#20)

<div class="columns">
<div>

**`ToSequenceUnitList ( unit, value )`:**
```
1. If _value_ is not an Object,
   throw a *TypeError* exception.
2. Let _sequence_ be
   StringSplitToList(_unit_, *"-and-"*).
3. Let _iteratorRecord_ be
   ? GetIterator(_value_, ~sync~).
4. Let _iterableValues_ be
   ? IteratorToList(_iteratorRecord_).
5. If length of _iterableValues_ ≠
   length of _sequence_, throw *RangeError*.
...
```

</div>
<div>

**Why Option 1 (Iterator)?**
- Directly aligns with `Intl.ListFormat`, which composes sequence unit output
- Works seamlessly with `Array`, TypedArrays, sets, and custom iterables
- Leading `Object` check prevents accidentally iterating primitive strings (e.g. `nf.format("56")` throws `TypeError`)

</div>
</div>

---

## Summary & Next Steps

- **Issue #20:** Highlighting TG2's choice of the **iterator protocol** (`GetIterator` + `IteratorToList` on `Object` inputs) for plenary feedback
- **Draft Spec:** Complete and up to date at [tc39.es/proposal-intl-sequence-units](https://tc39.es/proposal-intl-sequence-units/)
- **Stage 2.7 Reviewers:** Eemeli Aro (EAO) and Dan Minor (DLM)
- **Next Time:** Requesting Stage 2.7 advancement
