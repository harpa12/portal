# ExperimentalStableMemory

Byte-level access to (virtual) *stable memory*.

**WARNING**: As its name suggests, this library is **experimental**, subject to change and may be replaced by safer alternatives in later versions of Motoko. Use at your own risk and discretion.

This is a lightweight abstraction over IC *stable memory* and supports persisting raw binary data across Motoko upgrades. Use of this module is fully compatible with Motoko’s use of *stable variables*, whose persistence mechanism also uses (real) IC stable memory internally, but does not interfere with this API.

Memory is allocated, using 'grow(pages)`` , sequentially and on demand, in units of 64KiB pages, starting with 0 allocated pages. New pages are zero initialized. Growth is capped by a soft limit on page count controlled by compile-time flag `--max-stable-pages <n> `` (the default is 65536, or 4GiB).

Each `load` operation loads from byte address `offset` in little-endian format using the natural bit-width of the type in question. The operation traps if attempting to read beyond the current stable memory size.

Each `store` operation stores to byte address `offset` in little-endian format using the natural bit-width of the type in question. The operation traps if attempting to write beyond the current stable memory size.

Text values can be handled by using `Text.decodeUtf8` and `Text.encodeUtf8`, in conjunction with `loadBlob` and `storeBlob`.

The current page allocation and page contents is preserved across upgrades.

NB: The IC’s actual stable memory size (`ic0.stable_size`) may exceed the page size reported by Motoko function `size()`. This (and the cap on growth) are to accommodate Motoko’s stable variables. Applications that plan to use Motoko stable variables sparingly or not at all can increase `--max-stable-pages` as desired, approaching the IC maximum (currently 8GiB). All applications should reserve at least one page for stable variable data, even when no stable variables are used.

## size

``` motoko
let size : () -> (pages : Nat64)
```

Current size of the stable memory, in pages. Each page is 64KiB (65536 bytes). Initially `0`. Preserved across upgrades, together with contents of allocated stable memory.

## grow

``` motoko
let grow : (new_pages : Nat64) -> (oldpages : Nat64)
```

Grow current `size` of stable memory by `pagecount` pages. Each page is 64KiB (65536 bytes). Returns previous `size` when able to grow. Returns `0xFFFF_FFFF_FFFF_FFFF` if remaining pages insufficient. Every new page is zero-initialized, containing byte 0 at every offset. Function `grow` is capped by a soft limit on `size` controlled by compile-time flag `--max-stable-pages <n>` (the default is 65536, or 4GiB).

## loadNat32

``` motoko
let loadNat32 : (offset : Nat64) -> Nat32
```

## storeNat32

``` motoko
let storeNat32 : (offset : Nat64, value : Nat32) -> ()
```

## loadNat8

``` motoko
let loadNat8 : (offset : Nat64) -> Nat8
```

## storeNat8

``` motoko
let storeNat8 : (offset : Nat64, value : Nat8) -> ()
```

## loadNat16

``` motoko
let loadNat16 : (offset : Nat64) -> Nat16
```

## storeNat16

``` motoko
let storeNat16 : (offset : Nat64, value : Nat16) -> ()
```

## loadNat64

``` motoko
let loadNat64 : (offset : Nat64) -> Nat64
```

## storeNat64

``` motoko
let storeNat64 : (offset : Nat64, value : Nat64) -> ()
```

## loadInt32

``` motoko
let loadInt32 : (offset : Nat64) -> Int32
```

## storeInt32

``` motoko
let storeInt32 : (offset : Nat64, value : Int32) -> ()
```

## loadInt8

``` motoko
let loadInt8 : (offset : Nat64) -> Int8
```

## storeInt8

``` motoko
let storeInt8 : (offset : Nat64, value : Int8) -> ()
```

## loadInt16

``` motoko
let loadInt16 : (offset : Nat64) -> Int16
```

## storeInt16

``` motoko
let storeInt16 : (offset : Nat64, value : Int16) -> ()
```

## loadInt64

``` motoko
let loadInt64 : (offset : Nat64) -> Int64
```

## storeInt64

``` motoko
let storeInt64 : (offset : Nat64, value : Int64) -> ()
```

## loadFloat

``` motoko
let loadFloat : (offset : Nat64) -> Float
```

## storeFloat

``` motoko
let storeFloat : (offset : Nat64, value : Float) -> ()
```

## loadBlob

``` motoko
let loadBlob : (offset : Nat64, size : Nat) -> Blob
```

Load `size` bytes starting from `offset` as a `Blob`. Traps on out-of-bounds access.

## storeBlob

``` motoko
let storeBlob : (offset : Nat64, value : Blob) -> ()
```

Write bytes of `blob` beginning at `offset`. Traps on out-of-bounds access.
