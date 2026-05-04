# Intl Sequence Units

This proposal introduces the ability to format sequences of measurement units, such as "feet and inches" or "degrees and minutes", natively using `Intl.NumberFormat`.

**Stage**: 0  
**Champion/Author**: Shane F Carr

## Spec

You can browse the [ecmarkup output](https://tc39-transfer.github.io/proposal-intl-sequence-units/)
or browse the [source](https://github.com/tc39-transfer/proposal-intl-sequence-units/blob/HEAD/spec.emu).

## Motivation

Many measurement systems, particularly the US Customary System, frequently express measurements using multiple units in sequence. Common examples include:
- A person's height: "5 feet, 11 inches"
- An angle: "5 degrees, 30 minutes"
- A weight: "2 pounds, 4 ounces"

Currently, formatting these requires developers to instantiate multiple `Intl.NumberFormat` instances and manually combine their outputs. This is not only cumbersome but also error-prone because the separators, order of units, and pluralization rules might differ across locales.

## Proposed Solution

This proposal extends `Intl.NumberFormat` to accept compound units separated by `-and-` (e.g., `foot-and-inch`, `degree-and-minute`).

When using a sequence unit, the `format` and `formatToParts` methods accept a plain JavaScript object instead of a single number. This object should contain properties corresponding to the individual sub-units in the sequence.

### Examples

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// "5 feet, 11 inches"
nf.format({ foot: 5, inch: 11 }); 

const angleNf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'degree-and-minute-and-second',
  unitDisplay: 'narrow'
});

// "5° 30′ 12″" (exact output depends on locale/narrow format for angle)
angleNf.format({ degree: 5, minute: 30, second: 12 });
```

### How it works

Under the hood, when formatting a sequence:
1. It validates that all sub-units separated by `-and-` are sanctioned unit identifiers.
2. It extracts the corresponding values from the passed object.
3. It formats each value using the specified unit. For intermediate units, it uses default rounding (up to 3 fraction digits) to ensure that if non-integer values are provided, the precision is preserved (following a "Garbage In, Garbage Out" philosophy). The configured rounding and fraction settings of the `Intl.NumberFormat` instance are applied only to the final unit in the sequence.
4. It combines the resulting formatted strings into a cohesive list using `Intl.ListFormat` under the hood, passing the appropriate `type: "unit"` and the number formatter's `unitDisplay` style, effectively handling locale-aware conjunctions and spacing correctly.

### Error Handling

When formatting a sequence unit, the object passed to `format` or `formatToParts` must contain a numeric value (or something coercible to a numeric value) for every sub-unit specified in the sequence. If any required field is missing or undefined, the method will throw a `TypeError`.

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// Throws a TypeError because 'inch' is missing
nf.format({ foot: 5 }); 
```

### Integration with Intl Unit Protocol

This proposal is designed to compose cleanly with the upcoming [Intl Unit Protocol](https://github.com/tc39/proposal-intl-unit-protocol) proposal. Under the Intl Unit Protocol, developers can pass both the value and the unit together in a single object to the `.format()` method, which is particularly useful when units are determined dynamically at runtime. 

When sequence units are used alongside the Intl Unit Protocol, the `value` field simply takes the object containing the sub-units, maintaining consistent behavior:

```javascript
const nf = new Intl.NumberFormat('en-US');

// "6 feet, 4 inches"
nf.format({ 
  unit: "foot-and-inch", 
  value: { foot: 6, inch: 4 } 
});
```

### Integration with Amount Proposal

This proposal also naturally extends to the [Amount](https://github.com/tc39/proposal-intl-amount) proposal, which aims to provide a robust primitive for encapsulating a numeric value along with its unit and precision. 

To represent a sequence unit, an `Amount` instance will need to internally store a structured value field (the object mapping sub-units to their respective scalar values) rather than a single scalar number. Because this proposal establishes that sequence units are fundamentally backed by objects, `Amount` can seamlessly adopt this same structural pattern to represent complex, multi-part measurements.

## Prior Art

The semantics and formatting patterns for sequence units are heavily inspired by Unicode Technical Standard #35 (LDML), specifically the section on [Unit Sequences (Mixed Units)](https://unicode.org/reports/tr35/tr35-general.html#Unit_Sequences). TR35 defines these as "composed sequences" (e.g., 5° 30′ or 3 ft 2 in) and specifies that the appropriate localized `listPattern` should be used to compose the units into a single formatted string. This proposal seeks to surface this standard behavior directly to ECMAScript developers.

## Alternatives Considered

### Formatting a scalar with auto-conversion

An alternative approach would be to allow passing a single scalar value to `format`, and have the formatter automatically convert and partition the value into the sequence units. For example, calling `nf.format(6.5)` with the unit `foot-and-inch` could theoretically convert the decimal and automatically yield `"6 feet, 6 inches"`.

This approach was not chosen for a few key reasons:

- **Input Ambiguity**: It is not immediately obvious what unit the scalar input represents. In the example `foot-and-inch`, is `6.5` measured in feet or inches? This would require introducing a concept of a "base" input unit, which adds surface area to the API.
- **Missing Conversion Data**: We cannot represent arbitrary custom sequence units using scalar values because the engine might not have the necessary data to perform the conversion (e.g., knowing how to convert a scalar into a custom sequence of units).
- **Rounding Errors**: Converting a scalar value into a sequence of units often involves floating-point arithmetic, which can introduce subtle rounding errors. By requiring the user to provide pre-partitioned values, we avoid these inaccuracies and ensure the output exactly matches the developer's intent.