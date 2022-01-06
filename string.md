# String design

## String struct

```rs
struct String {
    id: NonZeroU16, // 1..=65535
    ptr: ptr::Unique<u8>,  // ptr to null terminated string data
    cap: u32,
    len: u32,
}
```

All string usages (Triggers, map title, unit names, ...) need string `id`. String `ptr` and `len` are essential for mutating string. String `ptr` can be calculated by in-game triggers, by adding `STRx` address (fixed to `0x191943C8`) to string offset in table, but reading offsets from STR table is slow so we store `ptr` and `len` in struct. We need `cap` for checking out of bound error, although SC does *not* support mutating string offset, we can't gracefully reallocate string...

We can't avoid [**fucked string**][fucked_string], or **null terminated string with extra data** layout. SC needs null character to recognize where string ends.

String constants won't need string struct but collection of constant string ids could be used. String constants can be malaligned.

## Field details

`id = 0` means `(no text)` so range of id is [1, 65535].

`ptr` should be EPD, EUD-friendly form. `cap` could be EPD length too.

Modifier for `len` FastVar could be `Add`. Two possible `len` value forms:

* `EPD` + `subp`

    Store `subp` in 2 most significant bits, better form to mutate string.

* bytes length

    Better form for length counting? I'm not sure whether this is useful for `UTF-8` encoded CJK strings.

## String struct memory layout

Two possible memory layout. Proposal 1 is better at accessing single field. (`2T 3A` vs `2T 9A`) Proposal 2 is better at accessing all fields. (`5T 12A` vs `2T 9A`)

### Proposal 1. Triggers for each field

```py
# Notation
# _ = must initialize before use
# SetTo, 0 = fixed, must restore to original value if changed
Trigger(None, SetDeaths(_, SetTo, id, 0))
Trigger(None, SetDeaths(_, SetTo, ptr, 0))
Trigger(None, SetDeaths(_, SetTo, cap, 0))
Trigger(None, SetDeaths(_, SetTo, len, 0))
```
Size = 72 * 4 = 288 bytes

### Proposal 2. Packed in Single Trigger

```py
Trigger(None, (
    SetDeaths(_, SetTo, id, 0),
    SetDeaths(_, SetTo, ptr, 0),
    SetDeaths(_, SetTo, cap, 0),
    SetDeaths(_, SetTo, len, 0),
))
```
Size = ? (less than 288 bytes iirc)

## `STRx` layout in .chk

```rs
struct STRx<const COUNT: usize> {
    count: u32,  // Unused in SC
    offset: [u32; COUNT],
    data: [u8],
    string: [String; COUNT],
}
```

All string-related data laid first. String structs are not guaranteed to gather at single section but most do for space efficiency. Trigger locality matters.

## Further reading

[StringBuffer in eudplib][stringbuffer]

[STR section layout in scenario.chk][str_layout]

[fucked_string]: https://www.joelonsoftware.com/2001/12/11/back-to-basics/ "Back to basics"
[stringbuffer]: https://github.com/armoha/eudplib/blob/master/eudplib/eudlib/stringf/strbuffer.py "String buffer in eudplib"
[str_layout]: http://www.staredit.net/wiki/index.php/Scenario.chk#.22STR_.22_-_String_Data "STR section in scenario chk"
