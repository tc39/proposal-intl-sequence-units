# Intl Sequence Units

This proposal specifies a mechanism for formatting sequences of measurement units (e.g., "feet and inches" or "meters and centimeters") within `Intl.NumberFormat`.

- Stage: 0
- Champion: @sffc
- [Spec](https://tc39.es/proposal-intl-sequence-units/)
- [Original ECMA-402 issue](https://github.com/tc39/ecma402/issues/398)

Presentations:

- 114th TC39: [Slides](https://tc39.es/proposal-intl-sequence-units/slides/stage_1_or_2.html)

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

When a sequence unit is specified, `format` and `formatToParts` accept a JavaScript object as input. Each property of the object must correspond to a sub-unit identified in the sequence.

### Examples

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// "5 feet, 11 inches"
nf.format({ foot: 5, inch: 11 }); 

// "-5 feet, 11 inches" (Only the first unit renders the minus sign)
nf.format({ foot: -5, inch: -11 }); 

const massNf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'pound-and-ounce',
  unitDisplay: 'long'
});

// "2 pounds, 4 ounces" (Actual output is locale-dependent)
massNf.format({ pound: 2, ounce: 4 });
```

### Technical Semantics

The formatting procedure for sequence units follows these steps:
1. **Validation**: The sequence must be a valid combination of sanctioned units measuring the same quantity and arranged in descending order of magnitude. The sanctioned sequences are explicitly listed in the specification (e.g., `foot-and-inch`, `kilogram-and-gram`). Note that time units are excluded from sequence units, as they are managed by `Intl.DurationFormat`.
2. **Extraction**: Values are retrieved from the input object based on the sub-unit keys.
3. **Component Formatting**:
    - **Intermediate Units**: Required to be integers, and formatted as integers.
    - **Terminal Unit**: Formatted according to the rounding and fraction settings configured on the `Intl.NumberFormat` instance.
4. **Composition**: The resulting component strings are concatenated using `Intl.ListFormat` with `type: "unit"` and the specified `unitDisplay` style to ensure locale-appropriate conjunctions and spacing.

### Error Handling

Inputs to `format` or `formatToParts` must contain a value for every sub-unit defined in the unit identifier. Properties are read in the order they appear in the unit identifier sequence. If a required property is `undefined` or missing, a `TypeError` is thrown immediately. 

After all properties have been read and converted to numbers, two final validations occur:
1. **Mixed Signs**: All sub-units must have the same sign. Mixing positive and negative values (e.g., `{ foot: 5, inch: -11 }`) throws a `RangeError`.
2. **Intermediate Integers**: All intermediate sub-units (all but the final one) must be integers. Providing a non-integer intermediate value (e.g., `{ foot: 5.5, inch: 6 }`) throws a `RangeError`.

```javascript
// Throws RangeError: invalid unit sequence (not in a sanctioned group or order)
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'meter-and-foot' });

const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// Throws TypeError: 'inch' property is missing
nf.format({ foot: 5 }); 

// Throws RangeError: sub-units have mixed signs
nf.format({ foot: 5, inch: -11 });

// Throws RangeError: intermediate sub-unit is not an integer
nf.format({ foot: 5.5, inch: 6 });
```

### Integration with Intl Unit Protocol

This proposal is compatible with the [Intl Unit Protocol](https://github.com/tc39/proposal-intl-unit-protocol). When utilizing the protocol, the `value` property accepts the object mapping sub-units to their respective magnitudes:

```javascript
const nf = new Intl.NumberFormat('en-US');

nf.format({ 
  unit: "foot-and-inch", 
  value: { foot: 6, inch: 4 } 
});
```

### Integration with Amount Proposal

This proposal aligns with the [Amount](https://github.com/tc39/proposal-intl-amount) proposal. To support sequence units, an `Amount` instance encapsulates a structured value (an object mapping sub-units to magnitudes) rather than a scalar numeric value.

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
