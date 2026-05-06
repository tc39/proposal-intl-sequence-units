# Intl Sequence Units

This proposal specifies a mechanism for formatting sequences of measurement units (e.g., "feet and inches" or "degrees and minutes") within `Intl.NumberFormat`.

**Stage**: 0  
**Champion/Author**: Shane F Carr

## Spec

Technical specification (ecmarkup): [ecmarkup output](https://tc39-transfer.github.io/proposal-intl-sequence-units/)  
Source: [spec.emu](https://github.com/tc39-transfer/proposal-intl-sequence-units/blob/HEAD/spec.emu)

## Motivation

Measurement systems frequently employ multiple units in sequence to express a single magnitude. Examples include:
- Anthropometric height: "5 feet, 11 inches"
- Angular measurement: "5 degrees, 30 minutes"
- Mass: "2 pounds, 4 ounces"

Current ECMAScript implementations require manual composition of multiple `Intl.NumberFormat` outputs. This approach introduces risk regarding localized separators, unit ordering, and pluralization agreement, which vary significantly across locales. This proposal addresses these requirements by providing a standardized interface for multi-unit formatting.

Original ECMA-402 issue: [https://github.com/tc39/ecma402/issues/398](https://github.com/tc39/ecma402/issues/398)

## Proposed Solution

`Intl.NumberFormat` is extended to support compound unit identifiers joined by the `-and-` separator (e.g., `foot-and-inch`, `degree-and-minute`).

When a sequence unit is specified, `format` and `formatToParts` accept a JavaScript object as input. Each property of the object must correspond to a sub-unit identified in the sequence.

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

// "5° 30′ 12″" (Actual output is locale-dependent)
angleNf.format({ degree: 5, minute: 30, second: 12 });
```

### Technical Semantics

The formatting procedure for sequence units follows these steps:
1. **Validation**: All components of the `-and-` delimited identifier must be sanctioned unit identifiers.
2. **Extraction**: Values are retrieved from the input object based on the sub-unit keys.
3. **Component Formatting**:
    - **Intermediate Units**: Formatted using default rounding (maximum 3 fraction digits). This preserves precision for non-integer inputs without requiring the engine to perform unit conversion ("Garbage In, Garbage Out").
    - **Terminal Unit**: Formatted according to the rounding and fraction settings configured on the `Intl.NumberFormat` instance.
4. **Composition**: The resulting component strings are concatenated using `Intl.ListFormat` with `type: "unit"` and the specified `unitDisplay` style to ensure locale-appropriate conjunctions and spacing.

### Error Handling

Inputs to `format` or `formatToParts` must contain a value for every sub-unit defined in the unit identifier. If a required property is `undefined` or missing, a `TypeError` is thrown. Additionally, all sub-units must have the same sign; mixing positive and negative values (e.g., `{ foot: 5, inch: -11 }`) will throw a `RangeError`.

```javascript
const nf = new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'foot-and-inch',
});

// Throws TypeError: 'inch' property is missing
nf.format({ foot: 5 }); 

// Throws RangeError: sub-units have mixed signs
nf.format({ foot: 5, inch: -11 });
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

## Alternatives Considered

### Scalar Input with Automated Partitioning

The possibility of accepting a single scalar value (e.g., `6.5`) and automatically partitioning it into sequence units (e.g., `"6 feet, 6 inches"`) was evaluated and rejected for the following reasons:

- **Input Ambiguity**: Scalar inputs do not inherently define which sub-unit of a sequence they represent (e.g., in `foot-and-inch`, it is unclear if `6.5` refers to feet or inches).
- **Incomplete Conversion Data**: The engine may not possess conversion factors for all possible unit combinations, particularly for custom or future units. This precludes reliable automated partitioning.
- **Precision and Rounding**: Automated partitioning via floating-point arithmetic can introduce errors. Requiring pre-partitioned input ensures the output accurately reflects the developer's data.
