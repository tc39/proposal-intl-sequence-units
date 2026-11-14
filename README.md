# Intl Sequence Units

This proposal specifies a mechanism for formatting sequences of measurement units (e.g., "feet and inches" or "meters and centimeters") within `Intl.NumberFormat`.

- Stage: 2
- Champion: @sffc
- [Spec](https://tc39.es/proposal-intl-sequence-units/)
- [Original ECMA-402 issue](https://github.com/tc39/ecma402/issues/398)

Presentations:

- 114th TC39 (May 2026): Stage 1 & 2 - [Slides](https://tc39.es/proposal-intl-sequence-units/slides/stage_1_or_2.html) | [Notes](https://github.com/tc39/notes/blob/main/meetings/2026-05/may-20.md#intl-sequence-units-for-stage-1-or-2)
- 115th TC39 (July 2026): [Slides](https://tc39.es/proposal-intl-sequence-units/slides/time-units.html)
- 116th TC39 (September 2026): Stage 2 Update - [Slides](https://tc39.es/proposal-intl-sequence-units/slides/stage_2_update_2026-09.html)
- 117th TC39 (November 2026): Stage 2 Update - [Slides](https://tc39.es/proposal-intl-sequence-units/slides/stage_2_update_2026-11.html)

## Motivation

Measurement systems frequently employ multiple units in sequence to express a single magnitude. Examples include:

- Person height: "5 feet, 11 inches" or "1 meter, 80 centimeters"
- Mass: "2 pounds, 4 ounces"

Current ECMAScript implementations require manual composition of multiple `Intl.NumberFormat` outputs. This approach introduces risk regarding localized separators, unit ordering, and pluralization agreement, which vary significantly across locales. This proposal addresses these requirements by providing a standardized interface for multi-unit formatting.

For example, to format "5 feet, 11 inches" today, a developer must write:

```javascript
const feetNf = new Intl.NumberFormat("en", { style: "unit", unit: "foot" });
const inchNf = new Intl.NumberFormat("en", { style: "unit", unit: "inch" });
const lf = new Intl.ListFormat("en", { type: "unit" });

lf.format([feetNf.format(5), inchNf.format(11)]);
// "5 feet, 11 inches"
```

While this is possible, it is not ergonomic and is prone to user error:
- It requires creating and managing multiple `Intl.NumberFormat` instances.
- Developers must manually ensure the correct unit ordering.
- It does not automatically handle sign display across the sequence (e.g., only showing the minus sign on the first unit).

## Proposed Solution

`Intl.NumberFormat` is extended to support compound unit identifiers joined by the `-and-` separator (e.g., `foot-and-inch`, `meter-and-centimeter`, `pound-and-ounce`).

When a sequence unit is specified, `format` and `formatToParts` accept an array (or iterable, see [#19](https://github.com/tc39/proposal-intl-sequence-units/issues/19)) of values as input. Each element of the array must correspond in order to a sub-unit identified in the sequence.

### Examples

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// "5 feet, 11 inches"
nf.format([5, 11]); 

// "-5 feet, 11 inches" (Only the first unit renders the minus sign)
nf.format([-5, -11]); 

const massNf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'pound-and-ounce',
  unitDisplay: 'long'
});

// "2 pounds, 4 ounces" (Actual output is locale-dependent)
massNf.format([2, 4]);
```

### Technical Semantics

The formatting procedure for sequence units follows these steps:
1. **Validation**: The sequence must be a valid combination of sanctioned units measuring the same quantity and arranged in descending order of magnitude. The sanctioned sequences are explicitly listed in the specification (e.g., `foot-and-inch`, `kilogram-and-gram`). Note that time units are excluded from sequence units, as they are managed by `Intl.DurationFormat`. Significant digit rounding options (`minimumSignificantDigits`, `maximumSignificantDigits`, or `roundingPriority` values other than `"auto"`, tracked by `[[RoundingType]]`) are disallowed for sequence units and throw a `RangeError` during construction.
2. **Extraction**: Values are retrieved in order from the input array.
3. **Component Formatting**:
    - **Intermediate Units**: Required to be integers, and formatted as integers.
    - **Terminal Unit**: Formatted according to the fraction digit and rounding settings configured on the `Intl.NumberFormat` instance.
4. **Composition**: The resulting component strings are concatenated using `Intl.ListFormat` with `type: "unit"` and the specified `unitDisplay` style to ensure locale-appropriate conjunctions and spacing.

### Error Handling

Inputs to `format` or `formatToParts` must be an array whose length matches the number of sub-units defined in the unit identifier; otherwise, a `RangeError` is thrown. If any element in the sequence is `undefined`, a `TypeError` is thrown immediately.

After all elements have been read and converted to numbers, two final validations occur:
1. **Mixed Signs**: All sub-units must have the same sign. Mixing positive and negative values (e.g., `[5, -11]`) throws a `RangeError`. Zero values (`0` and `-0`) are considered neither positive nor negative for this check and may appear alongside either sign. Because only the first sub-unit renders a minus sign (and negative signs—including on `-0`—are stripped from subsequent sub-units), a leading `-0` is used to display a minus sign when the first sub-unit is zero (e.g., `[-0, -6]` formats as `"-0 feet, 6 inches"`, whereas `[0, -6]` omits the minus sign).
2. **Intermediate Integers**: All intermediate sub-units (all but the final one) must be integers. Providing a non-integer intermediate value (e.g., `[5.5, 6]`) throws a `RangeError`.

```javascript
// Throws RangeError: invalid unit sequence (not in a sanctioned group or order)
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'meter-and-foot' });

// Throws RangeError: significant digit options are disallowed with sequence units
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'foot-and-inch', maximumSignificantDigits: 2 });

const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// Throws RangeError: array length does not match the unit sequence
nf.format([5]); 

// Throws RangeError: sub-units have mixed signs
nf.format([5, -11]);

// Throws RangeError: intermediate sub-unit is not an integer
nf.format([5.5, 6]);
```

### Integration with Intl Unit Protocol

This proposal is compatible with the [Intl Unit Protocol](https://github.com/tc39/proposal-intl-unit-protocol). When utilizing the protocol, the `value` property accepts the array of sub-unit magnitudes:

```javascript
const nf = new Intl.NumberFormat('en-US');

nf.format({ 
  unit: "foot-and-inch", 
  value: [6, 4] 
});
```

### Integration with Amount Proposal

This proposal aligns with the [Amount](https://github.com/tc39/proposal-intl-amount) proposal. To support sequence units, an `Amount` instance accepts an array of sub-unit magnitudes (e.g., `new Amount([2, 30], 'hour-and-minute')`) rather than a scalar numeric value.

## Prior Art

The formatting requirements for sequence units are defined in Unicode Technical Standard #35 (LDML), Section [Unit Sequences (Mixed Units)](https://unicode.org/reports/tr35/tr35-general.html#Unit_Sequences). TR35 specifies the use of localized `listPattern` data to compose these sequences, which this proposal implements for ECMAScript.

## Future Directions

### Additional units

Some units commonly used in sequences, such as `arcminute` and `arcsecond`, are not currently sanctioned in ECMA-402. Formatting angular measurements like "5° 30′ 12″" (which would conceptually use `degree-and-arcminute-and-arcsecond`) would require a separate proposal to add `arcminute` and `arcsecond` to the list of sanctioned single unit identifiers before they can be utilized as sequence units.

## Alternatives Considered

### Scalar Input with Automated Partitioning

The possibility of accepting a single scalar value (e.g., `6.5`) and automatically partitioning it into sequence units (e.g., `"6 feet, 6 inches"`) was evaluated and rejected for the following reasons:

- **Input Ambiguity**: Scalar inputs do not inherently define which sub-unit of a sequence they represent (e.g., in `foot-and-inch`, it is unclear if `6.5` refers to feet or inches).
- **Incomplete Conversion Data**: The engine may not possess conversion factors for all possible unit combinations, particularly for custom or future units. This precludes reliable automated partitioning.
- **Precision and Rounding**: Automated partitioning via floating-point arithmetic can introduce errors. Requiring pre-partitioned input ensures the output accurately reflects the developer's data.
