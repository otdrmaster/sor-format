# The `.sor` file format (Bellcore / Telcordia OTDR data)

This document describes the layout of `.sor` files, the standard file format
for optical time domain reflectometer (OTDR) traces. It covers two versions:

- **Version 1**, Bellcore 1.x (format revision 100).
- **Version 2**, Telcordia SR-4731 issue 2 (format revision 200).

It covers only what the standard defines. Instruments add proprietary data of
their own. That data is not described here.

![Block layout of a .sor file](sor-blocks.svg)

## 1. Overview

- All integers are **little-endian**.
- Signed and unsigned integers are 2 or 4 bytes wide. The tables below use
  `u16`, `i16`, `u32`, `i32`.
- Text fields are either **null-terminated strings** (`cstr`) or **fixed-width
  character fields** (`char[n]`, no terminator). The standard does not declare
  a character encoding. Treat strings as bytes and decode them with care.
- A file is a sequence of **blocks**. The first block, `Map`, is a directory
  that names every other block with its revision and size, in file order.
- Fixed revision numbers are stored in hundredths: `100` means 1.00, `200`
  means 2.00.

### Version 1 and version 2

| | Version 1 (Bellcore 1.x) | Version 2 (SR-4731 issue 2) |
|---|---|---|
| File starts with | the map revision (`u16`) | the string `Map\0` |
| Map revision field | `100` | `200` |
| Block payload starts with | the first field | the block's own name as a `cstr`, e.g. `GenParams\0` |
| `GenParams` | no fiber type, no user offset distance | adds both |
| `FxdParams` | no distance-form offsets, no averaging time, no trace type | adds them (see §4) |
| `KeyEvents` record | no marker positions | five marker positions per event |

In version 2 the block size given in the map **includes** the name prefix.

## 2. Map block

| Field | Type | Notes |
|---|---|---|
| Marker | `cstr` | `Map\0`. Version 2 only. |
| Map revision | `u16` | `100` for version 1, `200` for version 2. |
| Map block size | `u32` | Size of the map block itself in bytes, from the start of the file (including the `Map\0` marker in version 2) to the end of the last directory entry. This is not the file size. |
| Number of blocks | `u16` | Counts the map block itself, so the directory below has this value minus one entries. |

Then one directory entry per block, in the order the blocks appear in the
file:

| Field | Type | Notes |
|---|---|---|
| Block name | `cstr` | e.g. `GenParams`, `DataPts`. |
| Block revision | `u16` | Revision of that block, in hundredths. |
| Block size | `u32` | Size of the block in bytes. In version 2 this includes the block's name prefix. |

Blocks follow the map directly, in directory order. For a well-formed file:

```
file length = map block size + sum of all block sizes in the directory
```

The standard blocks are `GenParams`, `SupParams`, `FxdParams`, `KeyEvents`,
`LnkParams`, `DataPts` and `Cksum`. Instrument makers may add proprietary
blocks. They appear in the directory like any other block. A reader that does
not recognize a block name must skip it using the size from the directory.

## 3. GenParams: general parameters

What was measured and by whom. Fields in file order.

| Field | Type | Units / scale | Notes |
|---|---|---|---|
| Language code | `char[2]` | | Two-letter code, e.g. `EN`. |
| Cable ID | `cstr` | | |
| Fiber ID | `cstr` | | |
| Fiber type | `u16` | | **v2 only.** ITU-T recommendation number, e.g. `652` for G.652. |
| Nominal wavelength | `u16` | nm | The wavelength as selected, e.g. `1550`. Whole nanometers. |
| Originating location (A) | `cstr` | | |
| Terminating location (B) | `cstr` | | |
| Cable code | `cstr` | | |
| Build condition | `char[2]` | | `BC` as built, `CC` as current, `RC` as repaired, `OT` other. Some files carry `NC` for new condition. |
| User offset | `i32` | 100 ps (1e-10 s) | Length of the launch lead from the front panel, as time. |
| User offset distance | `i32` | 0.1 distance unit | **v2 only.** The same offset as a distance, in tenths of the unit named in `FxdParams`. |
| Operator | `cstr` | | |
| Comment | `cstr` | | Free text entered by the user. |

Any string may be empty (a single `\0`).

## 4. SupParams: supplier parameters

The instrument that produced the trace. Every field is a `cstr`, in this
order:

| Field | Type | Notes |
|---|---|---|
| Supplier name | `cstr` | Instrument maker. |
| Mainframe ID | `cstr` | Mainframe model. |
| Mainframe serial number | `cstr` | |
| Optical module ID | `cstr` | |
| Optical module serial number | `cstr` | |
| Software revision | `cstr` | |
| Other | `cstr` | Free text. |

The layout is the same in both versions.

## 5. FxdParams: fixed parameters

How the trace was acquired. This block drives the distance axis. Fields in
file order.

| Field | Type | Units / scale | Notes |
|---|---|---|---|
| Date and time | `u32` | seconds since 1970-01-01 00:00:00 | Unix time. |
| Distance units | `char[2]` | | `mt` meters, `km` kilometers, `ft` feet, `kf` kilofeet, `mi` miles. Display units only. |
| Actual wavelength | `u16` | 0.1 nm | e.g. `15500` = 1550.0 nm. The wavelength actually emitted, which can differ from the nominal one. |
| Acquisition offset | `i32` | 100 ps (1e-10 s) | Time from the front panel to the first data point. Can be negative. |
| Acquisition offset distance | `i32` | 0.1 distance unit | **v2 only.** The same offset stated as a distance. |
| Number of pulse widths | `u16` | | N, the length of the three arrays that follow. Usually 1. |
| Pulse widths | `u16[N]` | ns | |
| Sample spacing | `u32[N]` | 10 fs (1e-14 s, i.e. 1e-8 µs) | Time between consecutive data points, one entry per pulse width. |
| Number of data points | `u32[N]` | | Data points for each pulse width. The sum equals the total in `DataPts`. |
| Group index of refraction | `u32` | × 100000 | e.g. `146820` = 1.46820. |
| Backscatter coefficient | `u16` | 0.1 dB | Stored as a positive magnitude of a negative value: `800` = −80.0 dB. |
| Number of averages | `u32` | | |
| Averaging time | `u16` | 0.1 s | **v2 only.** May be given instead of the number of averages. |
| Acquisition range | `u32` | 100 ps (1e-10 s) | Range the instrument was set to acquire over. |
| Acquisition range distance | `i32` | 0.1 distance unit | **v2 only.** The range stated as a distance. |
| Front panel offset | `i32` | 100 ps (1e-10 s) | Time between the instrument's optics and its front panel connector. |
| Noise floor level | `u16` | 0.001 dB | Stored as a positive magnitude: `10200` = −10.200 dB. |
| Noise floor scale factor | `i16` | | Scale for the noise floor level, normally 1. |
| Power offset first point | `u16` | 0.001 dB | Attenuation the instrument applied, if any. Not an absolute power reference. |
| Loss threshold | `u16` | 0.001 dB | Smallest event loss the instrument reports. |
| Reflectance threshold | `u16` | 0.001 dB | Stored as a positive magnitude: `55000` = −55.000 dB. |
| End-of-fiber threshold | `u16` | 0.001 dB | Loss step treated as the fiber end. |
| Trace type | `char[2]` | | **v2 only.** `ST` standard, `RT` reverse (measured from the far end), `DT` difference of two traces, `RF` reference. |
| Window coordinates | `i32[4]` | | **v2 only.** Display window of the instrument. Not needed to read the trace. |

**Version 1** has no acquisition offset distance, averaging time, acquisition
range distance, trace type or window coordinates. The remaining fields keep
their order.

## 6. KeyEvents: the event table

A count followed by one record per event, then a fixed trailer that describes
the whole span.

| Field | Type | Notes |
|---|---|---|
| Number of events | `u16` | |

Per event:

| Field | Type | Units / scale | Notes |
|---|---|---|---|
| Event number | `u16` | | Normally 1, 2, 3 … |
| Event propagation time | `u32` | 100 ps (1e-10 s) | Position of the event. Convert to distance as in §8. |
| Attenuation coefficient of lead-in fiber | `i16` | 0.001 dB/km | Slope of the fiber section before the event. Signed. |
| Event loss | `i16` | 0.001 dB | Signed. A negative value is a gain, which is a real reading at a splice between different fibers. |
| Event reflectance | `i32` | 0.001 dB | Normally negative, e.g. `-45000` = −45.000 dB. |
| Event code | `char[6]` | | See below. |
| Loss measurement technique | `char[2]` | | `2P` two-point, `LS` least squares, `OT` other. |
| Marker locations | `u32[5]` | 100 ps (1e-10 s) | **v2 only.** Points used to measure the event, see below. |
| Comment | `cstr` | | Free text. Often empty. |

The five marker locations, in order:

1. Near side of the measurement, for any technique.
2. Near side for least squares; the edge of the event for two-point and other.
3. Far side for least squares; empty otherwise.
4. Far side for least squares; empty otherwise.
5. The point where reflectance is calculated.

### Event code

Six ASCII characters.

| Position | Value | Meaning |
|---|---|---|
| 1 | `0` | Non-reflective event (fusion splice, bend). |
| 1 | `1` | Reflective event (connector, mechanical splice, break). |
| 1 | `2` | Saturated reflective event: the reflection overloaded the receiver. |
| 2 | `A` | Event added by the user. |
| 2 | `M` | Event moved by the user. |
| 2 | `E` | End of fiber. |
| 2 | `F` | Event found by the instrument's automatic analysis. |
| 2 | `O` | Event outside the measured range. |
| 2 | `D` | End of fiber modified by the user. |
| 2 | other | Unknown. Carry the character through unchanged. |
| 3 to 6 | | Landmark number, or `9999` when the event is not tied to a landmark. |

### Trailer

The 22 bytes that follow the last event record:

| Field | Type | Units / scale | Notes |
|---|---|---|---|
| End-to-end loss | `i32` | 0.001 dB | Total loss of the span between the two loss markers. Signed. |
| End-to-end loss start | `u32` | 100 ps (1e-10 s) | Position of the start marker. |
| End-to-end loss finish | `u32` | 100 ps (1e-10 s) | Position of the end marker. |
| Optical return loss (ORL) | `u16` | 0.001 dB | ORL of the span. |
| ORL start | `u32` | 100 ps (1e-10 s) | |
| ORL finish | `u32` | 100 ps (1e-10 s) | |

## 7. LnkParams: link parameters

The standard defines an optional `LnkParams` block that describes the link:
landmarks along the route and their positions. It is optional and is not
described here. Skip it by its size from the map.

## 8. DataPts: the trace

| Field | Type | Notes |
|---|---|---|
| Number of data points | `u32` | Total over all groups. Equals the sum of the point counts in `FxdParams`. |
| Number of scale factors | `i16` | Number of groups that follow. Usually 1. |

Then for each group:

| Field | Type | Notes |
|---|---|---|
| Data points in this group | `u32` | M |
| Scale factor | `u16` | Commonly `1000`. |
| Data points | `u16[M]` | Raw samples. |

### From samples to dB

Each sample is an attenuation value that grows with distance. With scale
factor `S`:

```
dB per count = S × 1e-6          (S = 1000 gives 0.001 dB per count)
level[i] (dB) = −raw[i] × S × 1e-6
```

Plotted as `level`, the trace descends with distance.
The zero of this scale is relative. A `.sor` file does not give an absolute
power reference, so the axis is dB, not dBm.

### From sample index to distance

Distances come from `FxdParams`. With `c = 299 792 458 m/s`, `n` = group index
of refraction, and `Δt` = sample spacing in seconds (stored value × 1e-14):

```
meters per sample   dx = c × Δt / n
first sample at     x0 = c × (acquisition offset × 1e-10) / n
sample i at         x  = x0 + i × dx
```

Event positions, markers and the acquisition range use the same conversion
with their own unit, 1e-10 s:

```
distance (m) = c × (stored time × 1e-10) / n
```

Note the two time units: sample spacing is in 1e-14 s, positions are in
1e-10 s. Mixing them up puts every event at the very start of the trace.

Changing the index of refraction rescales the distance axis without changing
any measured value.

## 9. Cksum

The last block of the file.

| Field | Type | Notes |
|---|---|---|
| Name prefix | `cstr` | `Cksum\0`. Version 2 only. |
| Checksum | `u16` | |

The standard specifies a CRC-16 CCITT checksum over every byte of the file
that precedes the stored value, including the `Cksum\0` prefix in version 2.
The parameters in common use are polynomial `0x1021`, initial value `0xFFFF`,
no bit reflection and no final XOR (catalogued as CRC-16/CCITT-FALSE, check
value `0x29B1` for the ASCII string `123456789`).

## 10. Reading a `.sor` file safely

- **Walk the map, not a fixed list.** Read each block at the offset the map
  implies and advance by the size the map gives, whether or not you recognize
  the block.
- **Skip unknown blocks by length.** Proprietary blocks are normal. Never fail
  on a block name you do not know.
- **Do not depend on block order** beyond what the map says, and do not
  require every standard block to be present.
- **Do not require the revision to equal 200 exactly.** Treat the revision as
  a number. Decide the layout of each block from the block itself: a version 2
  block starts with its own name.
- **Expect empty strings.** Any `cstr` can be a lone `\0`.
- **Check sizes against the file.** A block whose size runs past the end of
  the file is damaged. Report it rather than reading beyond the buffer.
- **Keep the original bytes** if you intend to write the file back. Re-encode
  only the blocks you change, then rebuild the map sizes and the checksum.

## 11. References

- Telcordia Technologies, **SR-4731, Issue 2**, *Optical Time Domain
  Reflectometer (OTDR) Data Format*. Available from Telcordia (Ericsson).
- Telcordia Technologies, **GR-196-CORE**, *Generic Requirements for Optical
  Time Domain Reflectometer (OTDR) Type Equipment*.
- Field names and units were cross-checked against two open-source readers,
  [pyOTDR](https://github.com/sid5432/pyOTDR) and
  [otdrs](https://github.com/JamesHarrison/otdrs).
