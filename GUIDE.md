# UkAddressFormat

Give `paf.format` a record as Royal Mail's Postcode Address File holds it, each field as the file writes it: an
empty or blank field counts as left out, and a building number of `0` means none. Store the record's fields as
they came, and the `UkAddress` it gives where you show or post the address: `postal_address` for a label or an
envelope (`text.format` writes it in Great Britain's order), `line_1` to `line_3` for a form with three address
lines, and `premise`, `number` and `unit` for search or display.

`paf.format` fails `PostcodeOutsideGb` when the postcode isn't in Great Britain's form, which a PAF record's never
is; treat it as a record that didn't come from PAF. It fails `PostTownUnreadable` only for a post town with a
line break or another control character in it.

Sort the addresses of one postcode or street with `paf.sorted` before offering them in a picker, so `2` comes
before `10` and `10` before `10A`.
