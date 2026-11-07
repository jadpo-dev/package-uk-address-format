# @jadpo/uk-address-format

Lays out a UK address from Royal Mail's Postcode Address File (PAF) as Royal Mail's Clear Addressing
guidelines say, and sorts a street's addresses in their numbered order. A Jadpo library package, called
`UkAddressFormat.v1` in your app.

```jadpo
var address: UkAddressFormat.UkAddress = attempt UkAddressFormat.paf.format(UkAddressFormat.PafAddress {
    organisation_name: "S D Alcott Florists"
    building_name: "Flower House 189A"
    thoroughfare: "Pye Green Road"
    post_town: "CANNOCK"
    postcode: "WS11 5SB"
})
// address.line_1 "S D Alcott Florists", line_2 "Flower House", line_3 "189A Pye Green Road",
// premise "Flower House, 189A", number "189A"; address.postal_address is a PostalAddress in GB
```

- `paf.format(record)` gives a `UkAddress`: `line_1` to `line_3` (every line after the second joined onto the
  third with `, `), `premise`, `number` and `unit`, and `postal_address`, a standard `PostalAddress` in Great
  Britain holding those lines, the post town in capitals and the postcode. It fails `PostcodeOutsideGb` when the
  postcode isn't in Great Britain's form, and `PostTownUnreadable` when the post town can't be one line.
- `paf.sorted(records)` puts a street's records in order: by building number, then building name, sub-building
  name, organisation, department and PO Box, with runs of digits compared as numbers.

An address is personal data: `PostalAddress` and `PostalCode` count as contact details, and every other field of
`PafAddress` and `UkAddress` is marked `PersonalData.contact`.

## Where the rules come from

The rules are a port of [uk-clear-addressing](https://github.com/ideal-postcodes/uk-clear-addressing) 4.2.1 by
Ideal Postcodes (MIT), which implements the rules in Royal Mail's PAF Programmers' Guide: the Exception Rule for
building names such as `12A`, `1-2` or `A`, splitting a name that ends in a range, the unit prefixes (`Unit`,
`Rear of`, ...) that stop that split, PO Boxes, dependent thoroughfares and localities, and the three-line
folding. The sort's natural comparison is a port of
[@cablanchard/koelle-sort](https://github.com/cblanc/koelle-sort) 0.1.0 (MIT), which uk-clear-addressing uses.
Both licences are in [NOTICE](NOTICE).

The tests port uk-clear-addressing's whole test suite: all 64 address fixtures in its `test/data/formatting.json`,
the cases of `test/rules.unit.ts`, `test/rules.utils.unit.ts` and `test/utils.unit.ts`, the formatting cases of
`test/address.unit.ts`, and the five streets of `test/data/*.sorted.json`. Many of the fixture addresses are Royal
Mail's own published examples from its Programmers' Guide; uk-clear-addressing ships them under its MIT licence
with no separate note.

The behaviour is the original's, quirks included. Among them: a building number or name that would go before the
street is lost when the record has no street or locality (the original writes it to index `-1` of its array); a
building number of `0` puts the sub-building and building names on one line; a field that is only white space
counts as empty; the unit prefixes need no word break after them (`Shopping` starts with `Shop`); and two sort
keys that differ only in case end the comparison as equal. Each regular expression is written to match exactly
what the JavaScript one does (its `.`, its `\s` and its `i` flag).

What it doesn't copy, on purpose:

- The record's other attributes and views (`udprn`, `umprn`, coordinates, counties, `district`, `ward`,
  `country`, `postcode_type`, `su_organisation_indicator`, `delivery_point_suffix`, `raw()`, `toJSON()`,
  `extractInteger`, `extractFloat`): none of them takes part in laying out an address.
- `cache`: a Jadpo value never changes, so there is nothing to cache.
- `postcode_inward`, `postcode_outward` and `formatPostcode`: the postcode is a `PostalCode`, which holds its one
  canonical form.
- Rule 7's branch for a building number of `0`, which the original notes can't be reached.
- Empty strings: an empty line, premise, number or unit is `none`.

What differs, because Jadpo isn't JavaScript:

- `postal_address` is new, so a postcode outside Great Britain's form, or a post town with a line break, now fails
  rather than coming back as written.
- The sort compares text by code point where JavaScript compares UTF-16 units; the two differ only between a
  character beyond U+FFFF and one from U+E000 to U+FFFF.
- The sort reads a run of digits as an exact decimal to 12 places, where JavaScript reads a double: they differ
  for numbers past 2^53 or with more than 12 places, and a run of over 26 digits before its point is compared as
  text.
- The sort walks a key a character at a time by recursion, so a key over about 990 characters stops at Jadpo's
  recursion limit.

## Install

It lives in its own repository so each release is this repository at a tag, which is how jadpo.dev's package
registry publishes every package. Once the registry opens:

```sh
jadpo package add @jadpo/uk-address-format
```

Licensed under Apache 2.0.
