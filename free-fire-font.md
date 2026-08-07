# Free Fire Nickname Character Counter

A tiny, dependency-free function that counts Free Fire nickname characters
the way the game actually counts them — not the way `string.length` does.

## The problem

Free Fire's nickname field caps at **12 characters**. Simple enough — until
you try to validate that in code:

```js
"Ghost".length          // 5  ✅ correct
"👻Ghost".length         // 7  ❌ wrong — emoji is 2 UTF-16 code units
"꧁Ghost꧂".length        // 7  ✅ happens to be correct here
"𝓖𝓱𝓸𝓼𝓽".length          // 10 ❌ wrong — bold script letters are astral (SMP), 2 units each
```

JavaScript's `.length` counts **UTF-16 code units**, not visible characters.
Anything outside the Basic Multilingual Plane (bold/script/Fraktur styles,
many emoji) reports double. Any nickname validator built on plain `.length`
will silently reject valid names or accept invalid ones.

## What Free Fire actually does

Tested in-game across multiple symbol types (『 』, ⚜, ❧) and confirmed:
**one visible symbol = one character**, regardless of which Unicode block
it comes from. Free Fire counts by **codepoint**, not by UTF-16 unit.
Full test writeup: https://fontsgeneratorpro.com/free-fire-fonts/

## The fix

```js
function freeFireCharCount(str) {
  return [...str].length; // spreads by codepoint, not UTF-16 unit
}

freeFireCharCount("👻Ghost");   // 6  ✅
freeFireCharCount("𝓖𝓱𝓸𝓼𝓽");    // 5  ✅
```

Python equivalent (Python 3 strings are already codepoint-based, so this is
mostly a non-issue there — but here's an explicit version for parity):

```python
def free_fire_char_count(s: str) -> int:
    return len(s)  # Python 3 str is codepoint-based already
```

## Known limitation

This covers ordinary decorative symbols (brackets, stars, flourishes) — the
cases actually tested. **Stacked/combining-mark styles** (glitch, underline
via U+0332/U+0334/U+0336) were not part of the original in-game test and may
follow different counting behavior. Don't treat this library as authoritative
for those until independently verified.

## Try it interactively

No install needed — the [Free Fire Font Generator](https://fontsgeneratorpro.com/free-fire-fonts/)
has this same counting logic built into a live tool with 200+ styles.

## License
MIT
