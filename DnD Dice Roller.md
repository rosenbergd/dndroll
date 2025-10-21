# DnD Dice Roller

The intention of this project is as a learning project for bash, and an opportunity to create a well documented, and feature rich DnD die roller.

## Full Tests

These are great tests to make sure the main regex works as expected, and for testing the integration with the secondary regex:

```
d6
1d%
2x1d100
6xd4
3x1d20
14x3d2
2d8*12+3
2d8*12-3 <-- decrease result by 3
6d12l4h-2 <-- keep lowest 4, then drop top 2 of those (same as the clearly better l2)
2x6d12l-1h3 <-- keep highest 3, then drop lowest of those (or just h2) (repeat this entire thing twice)
2x6d12l-1h-1 <-- drop lowest 1 and highest 1 (repeat this entire thing twice)
2x3d(custom1)+5 <-- roll a custom die (notation in file to be considered, but files should be named the same as the name in brackets)
3x{2x2d8&2x5d6l3}*2+3h10 <-- & sign separates multiple different rolls, each one is the full format similar to the above ones. If you would like to apply a rule to the rolled results, you can surround them with curly braces, then use the notation as normal around them as if the part inside is the `xdy` part
2x{3x{2d6&5d4}*2+1l1}*3+2l-1h-1
2x6d12*3+2l-1h-1 <-- example with all capture groups, including non-capture groups
3d6*3+2*4 <-- this is not going to be allowed because of the order of operations confusion, should +2*4 mean +8, or is each a modifier on the die roll? rather just block it as invalid
```

## Main Regex

`^(?:([1-9][0-9]{,5})x)?((?:[1-9][0-9]{,5})?d(?:[1-9][0-9]{,5}|%|\([a-zA-Z0-9_]{,30}\))|\{[a-zA-Z0-9_\{\}&\*\+\-\%\(\)]{,255}\})(\*[1-9][0-9]{,5})?([+-][1-9][0-9]{,5})?(l-?[1-9][0-9]{,5})?(h-?[1-9][0-9]{,5})?(?: |$)`

**NOTE:** the `\-` in the above needs to be converted to `-` for ERE bash format

- Capture group 1 = multiplier at the start indicating repetitions of the entire pattern (with modifiers and roll filtering) (`1x`, `2x`, `3x`, etc. with the `x` removed through the regex)
- Capture group 2 = `xdy` format handler (`y` can be a custom die `(...)`), `{...&...}` handler (just for splitting)
- Capture group 3 = multiply modifier (`*3`, `*5`, etc.)
- Capture group 4 = skill modifier, etc. (`+2`, `+6`, `-5`, etc.)
- Capture group 5 = keep lowest x rolls / drop lowest x rolls if negative (`l3`, `l1`, `l-7`, etc.)
- Capture group 6 = keep highest x rolls / drop highest x rolls if negative (`h3`, `h-2`, `h10`, etc.)

Since both highest and lowest can be mixed together, they're handled as the largest number first, e.g. if keep highest 5, and keep lowest 3, it will keep the lowest 3 of the top 5 rolls, even though they're specified in reverse order, i.e. `l3h5`.

It's also possible to specify `l-3` or `h-5` for example to drop instead of keep. I.e. `l-3` = drop lowest 3 rolls, and `h-5` = drop highest 5 rolls. These can be applied together, and since they operate on completely different sides, they apply without a care for ordering (can still use highest handled first in case something like `l-3h5` is used, to drop the lowest 3 from the top 5, although that's just a bad notation, but accepted).

Capture group 2 needs further processing to figure out which type of rule it is, recursive `{...&...}`, special dice `(...)`, or regular `xdy`.

## Regex For Group 2

`^(?:([1-9][0-9]{,5})?d([1-9][0-9]{,5}|%|\([a-zA-Z0-9_]{,30}\))|\{([a-zA-Z0-9_\{\}&\*\+\-\%\(\)]{,255})\})$`

**NOTE:** the `\-` in the above needs to be converted to `-` for ERE bash format

- Capture group 1 = number of rolls of the specified die type (`5`, `19`, `1`, etc.)
- Capture group 2 = die type, whether specified as number of sides (`6`, `20`, `100`), percent sign (`%` = `100`), or custom die specifier with filename (`custom1`, `complex_set9`, `myDice`, etc.)
- Capture group 3 = if groups 1 and 2 are not found, group 3 is the recursive group with the curly brackets removed, ready for applying the first regex on it recursively after splitting by `&` sign

Capture group 3 needs to be split by `&`, with each part recursively sent to the first regex handler.

### Regex 2 Tests

These are great tests to make sure the regex for capture group 2 of the main regex works well on its own:

```
d6
1d%
1d100
999999d999999
1d20
3d2
2d8
6d12
3d(custom1)
{2x2d8&2x5d6l3}
```