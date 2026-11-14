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

## Issue #20: Precedents for Reading List Inputs

Across ECMA-262 and ECMA-402, there are **four** patterns for reading list-like arguments:

1. **Iterator protocol (`GetIterator`)**
   - *Object-only:* `Iterator.zip` *(rejects primitives)*; `Iterator.concat` (`~reject-primitives~`)
   - *General (iterates strings):* `Intl.ListFormat`, `Promise.all`, `Map` / `Set`, `Math.sumPrecise`
2. **Array-like (`LengthOfArrayLike`)**
   - `Function.prototype.apply`, `Reflect.apply`, `CanonicalizeLocaleList` *(ECMA-402)*
3. **`IsArray` + indexed properties (`IsArray` + `LengthOfArrayLike`)**
   - `JSON.stringify` (`replacer`; no built-in throws `TypeError` to require `IsArray`)
4. **Hybrid: Iterable with Array-like fallback**
   - `Array.from`, `Array.fromAsync`, `%TypedArray%.from`

---

## TG2 Recommendation: Object-Only Iterator (#20)

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

**Why Object-Only `GetIterator`?**
- Aligns with `Intl.ListFormat` (iterator-based) and `Iterator.zip` (rejecting primitives)
- Accepts `Array`, TypedArrays, `Set`, generators, and custom iterables
- **Why reject primitives?** Without step 1, `nf.format("56")` on `"foot-and-inch"` would iterate `"5"` and `"6"`, coerce both via `ToIntlMathematicalValue`, and silently return `"5 feet, 6 inches"`!

</div>
</div>

---

## Summary & Next Steps

- **Issue #20:** Highlighting TG2's choice of the **Object-only iterator protocol** (`GetIterator` + `IteratorToList`) for plenary feedback
- **Draft Spec:** Complete and up to date at [tc39.es/proposal-intl-sequence-units](https://tc39.es/proposal-intl-sequence-units/)
- **Stage 2.7 Reviewers:** Eemeli Aro (EAO) and Dan Minor (DLM)
- **Next Time:** Requesting Stage 2.7 advancement
