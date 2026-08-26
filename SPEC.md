# Blind Audio Test File Format Specification (`.batest`)

**Version:** 2.2
**Status:** Stable — non-breaking, additive revision of v2.1

This document is the authoritative specification of the `.batest` file
format, an open, ZIP-based container format for storing reproducible
blind audio comparison tests.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119.

> **v1.0 users:** This document describes v2.2, a non-breaking,
> additive revision of v2.0/v2.1 (`formatVersion` remains `2`; see
> [Test Object Schema](#test-object-schema) for what's new). Files
> produced by a v2.0 or v2.1 implementation remain fully valid under
> v2.2. The v1.0 specification (single-test `test.json` structure)
> remains available in this repository's git history via the `v1.0`
> tag/release. See [Migration from v1](#migration-from-v1) for a
> summary of what changed since v1.

## Table of Contents

1. [Overview](#overview)
2. [Design Goals](#design-goals)
3. [File Extension and Container Format](#file-extension-and-container-format)
4. [Directory Structure](#directory-structure)
5. [manifest.json](#manifestjson)
6. [testSet.json](#testsetjson)
   1. [Top-Level Fields](#top-level-fields)
   2. [Test Object Schema](#test-object-schema)
   3. [Field Placement and Overrides](#field-placement-and-overrides)
   4. [backingTrack](#backingtrack)
   5. [loudnessMatching](#loudnessmatching)
   6. [trackLengthMode](#tracklengthmode)
   7. [Track Object Schema](#track-object-schema)
   8. [Format Conversion Rule](#format-conversion-rule)
   9. [testTypeConfig](#testtypeconfig)
7. [Assets](#assets)
8. [Resources](#resources)
9. [Test Results](#test-results)
10. [Data Integrity and Security](#data-integrity-and-security)
    1. [File Integrity](#file-integrity)
    2. [Zip-Slip Protection](#zip-slip-protection)
    3. [Canonical Serialization for Hashing](#canonical-serialization-for-hashing)
11. [Schema Hygiene Convention](#schema-hygiene-convention)
12. [Identifier Stability](#identifier-stability)
13. [Versioning and Compatibility](#versioning-and-compatibility)
14. [Migration from v1](#migration-from-v1)
15. [Guiding Principle](#guiding-principle)

## Overview

A `.batest` file is a ZIP container that fully describes one or more
related blind audio comparison tests, grouped into a **test set**: the
audio tracks being compared, the metadata needed to reproduce the
listening experience, and optional supporting material (cover images,
documentation, external references). A conforming `.batest` file is
self-contained and MUST be playable offline, without any network
access, by any implementation that supports the test type(s) it
contains.

This specification describes the format independently of any single
application or implementation. Test *results* (a listener's actual
responses) are explicitly out of scope — see
[Test Results](#test-results).

## Design Goals

- Open file format
- ZIP-based container
- Cross-platform
- Versioned and forward-compatible
- Fully usable offline
- Modular and extensible

## File Extension and Container Format

The format uses the `.batest` file extension. A `.batest` file MUST be
a valid ZIP archive.

## Directory Structure

```text
MyTest.batest
│
├── manifest.json
├── testSet.json
├── required/
│   ├── track1.flac        (converted from WAV)
│   ├── track2.mp3         (lossy source, kept as-is)
│   └── track3.flac
├── assets/
│   ├── cover.jpg
│   ├── setup.jpg
│   ├── room.png
│   ├── measurements.pdf
│   └── intro.mp4
└── resources/
```

- `manifest.json` and `testSet.json` MUST be present at the root of
  the archive.
- `required/` contains every audio file that is necessary to run any
  test in the set (tracks, and, if present, each test's backing
  track). Implementations MUST treat the files in this directory as
  required for playback.
- `assets/` contains optional supporting files (images, video, PDF
  documents) referenced from `testSet.json`. See [Assets](#assets).
- `resources/` is reserved for future use; external references that
  are not embedded in the container are instead listed in
  `testSet.json`'s `resources` array (see [Resources](#resources)).

## manifest.json

`manifest.json` is a lightweight, redundant summary of the test set.
It allows a file browser or import dialog to display key information
without unzipping and parsing `testSet.json`.

```json
{
  "formatVersion": 2,
  "createdWith": "Blind Audio Test 1.0.0",
  "createdAt": "2026-07-10T18:30:00Z",
  "title": "Example Test",
  "comparisonCategory": "microphones",
  "models": [
    { "manufacturer": "Neumann", "model": "U87" },
    { "manufacturer": "AKG", "model": "C414" }
  ]
}
```

All fields listed above are REQUIRED.

- `formatVersion` is the container/schema version. See
  [Versioning and Compatibility](#versioning-and-compatibility).
- `createdWith` identifies the application and version that produced
  the file.
- `createdAt` is the creation timestamp, in ISO 8601 format.
- `title` is the test set's display title.
- `comparisonCategory` MUST be the resolved, human-readable value: if
  the author selected a free-text "Others" category, the free-text
  value is substituted in directly here. This differs from
  `testSet.json`, where `comparisonCategory` stays the raw category
  value and the free text lives in its own `comparisonCategoryOther`
  field (see [Top-Level Fields](#top-level-fields)) — `manifest.json`'s
  `comparisonCategory` is a display-only derivative, not a copy, of
  `testSet.json`'s.
- `models` lists the distinct manufacturer/model combinations used
  across all tracks in all of the test set's `test[]` objects,
  deduplicated. If a track used a free-text "Others" manufacturer/
  model, that free-text value is used here as well. `models` MUST be
  an empty array if the test set has no manufacturer/model data (e.g.
  a Mixes/Masters or Codecs comparison).

> **v2 change:** `manifest.json` no longer has a `testType` field. A
> test set can now contain multiple `test[]` objects, each with its
> own `testType`, so a single top-level value could no longer
> represent the test set. Determining the test type(s) in a test set
> now requires parsing `testSet.json`'s `test[]` array.

## testSet.json

`testSet.json` is the primary, authoritative description of the test
set.

```json
{
  "testSetId": "UUIDv7",
  "creatorUuid": null,
  "title": "Example Test",
  "description": "...",
  "comparisonCategory": "microphones",
  "content": {
    "type": "vocals",
    "genres": ["indie-pop"],
    "tags": ["soft", "breathy"]
  },
  "test": [
    {
      "testId": 0,
      "testType": "abx",
      "tracks": [],
      "backingTrack": {
        "file": "required/background.flac",
        "gainDb": -12
      },
      "loudnessMatching": {
        "mode": "integrated-lufs",
        "reference": "loudest",
        "version": 1
      },
      "trackLengthMode": "shortest",
      "testTypeConfig": {
        "abx": { "testRounds": 8 }
      }
    }
  ]
}
```

`description` and `assets` are omitted from the example above; see
[Schema Hygiene Convention](#schema-hygiene-convention) and
[Assets](#assets) for when each is present versus omitted.

### Top-Level Fields

- `testSetId` is the test set's unique identifier (a UUIDv7). REQUIRED.
  See [Identifier Stability](#identifier-stability) for normative
  requirements on this field.
- `creatorUuid` is an OPTIONAL identifier for the person or account
  that created the test set (e.g. for attribution once publishing/
  sharing is supported). This field MUST always be present in
  `testSet.json`; it is `null` when not set (e.g. an anonymous local
  export). Unlike most other optional fields in this format,
  `creatorUuid` is represented by an explicit `null`, not by omitting
  the key — see [Schema Hygiene Convention](#schema-hygiene-convention).
- `title` is the test set's display title. REQUIRED.
- `description` is an OPTIONAL free-text description of the whole test
  set. The key is omitted entirely when no description was set.
- `comparisonCategory` is the raw category value chosen when creating
  the test set (e.g. `"microphones"` or `"others"`). REQUIRED.
- `comparisonCategoryOther` holds the free-text specification when
  `"others"` was selected for `comparisonCategory`. Omitted otherwise.
- `comparisonSubcategory` is only relevant for categories that define
  subcategories (currently only `"effects"`). Omitted for all other
  categories, and also omitted when the category defines subcategories
  but none was selected.
- `comparisonSubcategoryOther` holds the free-text specification when
  `"others"` was selected for `comparisonSubcategory`. Omitted
  otherwise.
- `content` — OPTIONAL, test-set-wide content description. See
  [Field Placement and Overrides](#field-placement-and-overrides).
- `test` — the array of test objects that make up the test set.
  REQUIRED. MUST contain at least one item. See
  [Test Object Schema](#test-object-schema).
- `playback` and `randomization` are open objects reserved for future
  fields; this specification does not yet define any fields for them.
  Both are OPTIONAL and omitted entirely when they hold no content.
- `assets` — OPTIONAL, test-set-wide assets. See
  [Assets](#assets) and [Field Placement and Overrides](#field-placement-and-overrides).
- `resources` — see [Resources](#resources).

`recording` is **not** a valid field at the `testSet.json` root level
in v2 — it is only meaningful at the test-object and track-object
level. See [Field Placement and Overrides](#field-placement-and-overrides).

### Test Object Schema

Each entry in `testSet.json`'s `test` array describes one runnable
test within the test set (for example, a test set could bundle an
A/B/X test followed by a separate Rating test over the same source
material).

```json
{
  "testId": 0,
  "testType": "abx",
  "title": "Preamp A vs. Preamp B",
  "content": {},
  "recording": {},
  "soundSource": "vocals",
  "tracks": [],
  "backingTrack": null,
  "loudnessMatching": {
    "mode": "integrated-lufs",
    "reference": "loudest",
    "version": 1
  },
  "trackLengthMode": "shortest",
  "assets": {},
  "testTypeConfig": {
    "abx": { "testRounds": 8 }
  }
}
```

`content`, `recording`, and `assets` are shown above only to indicate
where they are positioned in the object; each is OPTIONAL and omitted
entirely when empty, per the
[Schema Hygiene Convention](#schema-hygiene-convention) — see
[Field Placement and Overrides](#field-placement-and-overrides).

- `testId` — the test object's zero-based integer index within the
  `test` array. REQUIRED. `testId` MUST correspond to the object's
  position in `test` (the first test object has `testId: 0`, the
  second `testId: 1`, and so on, incrementing sequentially), and MUST
  be unique within the test set.
- `testType` — identifies this test object's test type. REQUIRED.
  MUST be one of the values defined in [testTypeConfig](#testtypeconfig)
  (`"ab"`, `"abx"`, `"abx-then-ab"`, `"ranking"`, `"rating"`), and MUST
  match `testTypeConfig`'s single key on the same test object.
- `title` — an OPTIONAL display title for this individual test, distinct
  from `testSet.json`'s root-level `title` (which names the whole test
  set). The key is omitted entirely when not set; implementations MAY
  fall back to a generated label (e.g. `"Test 1"`, based on `testId`)
  for display when it is absent. *(New in v2.1 — see the note below.)*
- `content` — OPTIONAL. See
  [Field Placement and Overrides](#field-placement-and-overrides).
- `recording` — OPTIONAL. See
  [Field Placement and Overrides](#field-placement-and-overrides).
- `soundSource` — an OPTIONAL raw sound-source-category value for this
  individual test (e.g. `"vocals"`, `"acoustic-guitar"`, `"other"`),
  identifying what kind of source material this specific test's tracks
  are drawn from. Unlike `comparisonCategory`, which applies to the
  whole test set, `soundSource` is scoped to a single test object,
  since different tests within the same test set can compare different
  kinds of source material (e.g. a mixed test set with one vocal test
  and one instrumental test). The key is omitted entirely when not set.
  *(New in v2.2 — see the note below.)*
- `soundSourceOther` — holds the free-text specification when the
  sentinel value `"other"` was selected for `soundSource`. Omitted
  otherwise. **Note the sentinel here is exactly `"other"`, singular —
  not `"others"` as used by `comparisonCategory`/`comparisonCategoryOther`
  at the `testSet.json` root.** This is an intentional but easy-to-miss
  inconsistency between the two fields; do not assume the same sentinel
  string applies to both. *(New in v2.2.)*
- `soundSourceSubtype` — an OPTIONAL, more specific classification
  within `soundSource` (e.g. a `soundSource` of `"vocals"` might have a
  `soundSourceSubtype` of `"lead-vocal"` or `"backing-vocal"`). Not
  every `soundSource` value defines subtypes; the key is omitted
  entirely both when the chosen `soundSource` has no subtypes and when
  it does but none was selected. Independent sentinel from
  `soundSource`'s. *(New in v2.2.)*
- `soundSourceSubtypeOther` — holds the free-text specification when
  the sentinel value `"other"` was selected for `soundSourceSubtype`,
  independently of whatever was selected (or not) for `soundSource`
  itself. Omitted otherwise. *(New in v2.2.)*
- `tracks` — the array of track objects being compared in this test.
  REQUIRED. See [Track Object Schema](#track-object-schema).
- `backingTrack` — see [backingTrack](#backingtrack). REQUIRED (always
  present; `null` when unused).
- `loudnessMatching` — see [loudnessMatching](#loudnessmatching).
- `trackLengthMode` — see [trackLengthMode](#tracklengthmode).
- `assets` — OPTIONAL, test-specific assets. See
  [Assets](#assets) and [Field Placement and Overrides](#field-placement-and-overrides).
- `testTypeConfig` — see [testTypeConfig](#testtypeconfig). REQUIRED.

> **v2.1 change:** each test object gained a new OPTIONAL `title`
> field, letting an individual test within a test set have its own
> display title, independent of the test set's root-level `title`.
> This is a non-breaking, additive change — `formatVersion` stays `2`
> — and files written by a v2.0 implementation (which omit this field)
> remain fully valid; implementations reading such a file fall back to
> a generated per-test label as before.

> **v2.2 change:** each test object gained four new OPTIONAL fields —
> `soundSource`, `soundSourceOther`, `soundSourceSubtype`, and
> `soundSourceSubtypeOther` — identifying the kind of source material
> that individual test's tracks are drawn from, following the same
> raw-value-plus-free-text-fallback pattern already used by
> `comparisonCategory`/`comparisonCategoryOther` at the `testSet.json`
> root, but scoped per test object rather than per test set (see above
> for why). This is a non-breaking, additive change — `formatVersion`
> stays `2`. Although the reference `Blind Audio Test` application
> currently always sets `soundSource` when creating a test, this
> specification deliberately keeps all four fields OPTIONAL (rather
> than adding `soundSource` to the test object's required fields):
> making a newly introduced field required at the schema level would
> retroactively invalidate every `.batest` file already produced under
> v2.0/v2.1, which is exactly what "non-breaking, additive" rules out.
> Files written by a v2.0 or v2.1 implementation (which omit all four
> fields) remain fully valid under v2.2, and implementations reading a
> test object without `soundSource` MUST treat it as "not specified"
> rather than erroring.

### Field Placement and Overrides

v2 introduces multiple valid nesting levels for `content`, `recording`,
and `assets`. This is new relative to v1, where `content` and
`recording` existed only at the (single) test level and the track
level. Each field's valid levels are:

| Field       | testSet.json root | test object | track object |
|-------------|:------------------:|:-----------:|:-------------:|
| `content`   | ✅                  | ✅           | ✅             |
| `recording` | ❌                  | ✅           | ✅             |
| `assets`    | ✅                  | ✅           | ❌             |

All occurrences are OPTIONAL and, per the
[Schema Hygiene Convention](#schema-hygiene-convention), the key is
omitted entirely at any level where there is nothing to add — never
written as `null` or an empty object.

**`content` and `recording` override rule:** where the same field is
present at more than one applicable level for a given track, the more
specific level MUST take precedence over the less specific one,
**per field**, not per object. For `content`, precedence from least to
most specific is: `testSet.json` root → test object → track object.
For `recording`, precedence is: test object → track object (there is
no root-level `recording`). A consuming application MUST merge values
field-by-field, with the most specific level's value winning when
present, falling back to the next-less-specific level otherwise.
Array-typed sub-fields (e.g. `content.tags`, `content.genres`,
`recording.signalChain`) are NOT merged element-by-element: if a more
specific level provides its own array for a given sub-field, that
array replaces the less specific level's array for that sub-field
entirely, rather than being concatenated with it.

*Example.* A microphone comparison test set with these fragments:

```json
// testSet.json root
"content": { "type": "vocals", "tags": ["soft"] }
```

```json
// test object (test[0])
"content": { "tags": ["breathy"] },
"recording": {
  "environment": "studio",
  "signalChain": [
    { "type": "preamp", "manufacturer": "Millennia", "model": "HV-3C" }
  ]
}
```

```json
// track object (test[0].tracks[0])
"recording": {
  "signalChain": [
    { "type": "microphone", "manufacturer": "Neumann", "model": "U 87 Ai" }
  ]
}
```

For this track, the effective merged values are:

- `content.type` = `"vocals"` (only set at root, so root's value
  applies).
- `content.tags` = `["breathy"]` (set at test level, which is more
  specific than root; there is no track-level `content.tags` to
  override it further).
- `recording.environment` = `"studio"` (only set at test level; there
  is no root-level `recording` at all).
- `recording.signalChain` = `[{ "type": "microphone", "manufacturer":
  "Neumann", "model": "U 87 Ai" }]` (the track-level array replaces the
  test-level array entirely, rather than appending to it — the
  test-level preamp entry is not part of the effective chain for
  display purposes at track level; it remains available to
  implementations that want to reconstruct the full chain by reading
  both levels directly).

**`assets`:** unlike `content` and `recording`, this specification
does not define an override or merge rule between `testSet.json`
root-level `assets` and test-object-level `assets` — both, where
present, are simply the assets relevant to that scope (test-set-wide
documentation vs. test-specific documentation). How an implementation
presents assets from both levels together is application-specific.

### backingTrack

`backingTrack` is an OPTIONAL, single audio track that plays
continuously in the background throughout a test (e.g. a full music
production over which the individually compared elements — instrument
takes, effects, etc. — are layered). It is not part of the blind
comparison itself: it plays unchanged, in parallel, for every track
being evaluated in that test.

This field MUST always be present on a test object; it is `null` when
that test does not use a backing track (see
[Schema Hygiene Convention](#schema-hygiene-convention)).

```json
"backingTrack": {
  "file": "required/background.flac",
  "gainDb": -12
}
```

- `file` — path to the backing track's audio file inside `required/`
  in the container, following the same convention as the regular track
  files referenced by `tracks[].filename`.
- `gainDb` — a gain offset in decibels applied to the backing track on
  playback, relative to its own original loudness.

### loudnessMatching

`loudnessMatching` describes the loudness-matching behavior used for a
test. The key is OPTIONAL and is omitted entirely when loudness
matching was not enabled for that test — there is no `enabled: false`
representation; the block's mere presence on the test object means it
was enabled.

```json
"loudnessMatching": {
  "mode": "integrated-lufs",
  "reference": "loudest",
  "version": 1
}
```

- `mode` — the loudness-measurement method used. Currently, the only
  defined value is `"integrated-lufs"`.
- `reference` — which track's loudness the others are matched to.
  Currently, the only defined value is `"loudest"`.
- `version` — the calculation version of `mode` at the time the test
  was exported. This is tracked independently of `formatVersion` and
  is incremented whenever that mode's calculation logic changes, so
  that a `.batest` file exported under an older calculation method
  remains distinguishable from one exported after a change, even
  though `mode`'s name stays the same. Implementations SHOULD treat
  `version` as informational metadata about how a track's
  `integratedLufs` was derived, not as something to validate or
  reject.

Implementations MUST accept files that predate this convention and may
still contain a legacy `enabled` and/or `target` field inside the
block; both MUST be ignored on import, since the block's presence
already conveys "enabled", and no known implementation ever wrote a
`target` value other than matching relative to `reference`.

### trackLengthMode

`trackLengthMode` determines how tracks with different durations are
handled during playback of a test:

- `"shortest"` — playback ends when the shortest compared track
  finishes.
- `"longest"` — playback continues until the longest compared track
  finishes, with shorter tracks going silent once they end.

This field is OPTIONAL and is only meaningful — and only ever written
— when that test's tracks actually have different durations; it is
omitted entirely when all of the test's tracks have the same length
(or there is only one track). Implementations MUST assume `"shortest"`
when the key is absent but the tracks do have different lengths.

`trackLengthMode` does not apply to a test object whose `testType` is
`rating` (`testTypeConfig.rating`): a Rating test presents one track at
a time rather than playing multiple tracks in sync, so this field is
never written for that test type, regardless of whether the tracks
have different durations. Implementations MUST ignore this key if
present on a Rating test object (e.g. from a file exported by an older
implementation).

### Track Object Schema

Each entry in a test object's `tracks[]` follows this schema,
independent of test type, so the same schema is used for A/B, A/B/X,
and multitrack Ranking tests alike.

```json
{
  "trackId": 0,
  "filename": "track1.flac",
  "originalFilename": "Vocal.wav",
  "manufacturer": "neumann",
  "model": "u87",
  "label": null,
  "recording": {
    "signalChain": [
      { "type": "microphone", "manufacturer": "Neumann", "model": "U 87 Ai" }
    ]
  },
  "originalFormat": "wav",
  "storedFormat": "flac",
  "originalSampleRate": 48000,
  "originalBitDepth": 24,
  "durationSeconds": 187.4,
  "integratedLufs": -18.7
}
```

- `trackId` — the track's zero-based integer index within its parent
  test object's `tracks` array. REQUIRED. `trackId` MUST correspond to
  the track's position in that `tracks` array (the first track has
  `trackId: 0`, the second `trackId: 1`, and so on, incrementing
  sequentially), and MUST be unique within that test object's `tracks`
  array (per [Identifier Stability](#identifier-stability), scoped to
  the parent test object rather than globally).
- `filename` — the technical, sequentially generated name the file is
  actually stored under in `required/` (e.g. `"track1.flac"`).
  REQUIRED. This is authoritative for locating the track's audio file
  inside the container.
- `originalFilename` — the name of the file as originally uploaded by
  the user (e.g. `"Vocal.wav"`), kept purely for display purposes
  (e.g. showing the listener/creator their original file name after
  re-importing a test). It has no effect on playback, randomization,
  or file resolution inside the container. REQUIRED.
- `manufacturer` / `model` — OPTIONAL, omitted entirely for
  categories where they stay optional and have no value (e.g.
  Mixes/Masters, Codecs).
- `manufacturerOther` / `modelOther` — hold the free-text value when
  `"others"` was selected for manufacturer or model respectively.
  Omitted otherwise.
- `label` — an OPTIONAL, freely chosen display name for the track
  (e.g. `"Take 1 - close mic"`), independent of its automatic
  `trackId`/ordering. This field MUST always be present; it is `null`
  when not set.
- `notes` — an OPTIONAL free-text note about the track. The key is
  omitted entirely when no note was set.
- `content` / `recording` — OPTIONAL track-level delta objects; see
  [Field Placement and Overrides](#field-placement-and-overrides).
  Omitted entirely when there is nothing track-specific to add.
- `originalFormat` — the format of the file as originally uploaded.
  REQUIRED.
- `storedFormat` — the format actually present in `required/` inside
  the container. REQUIRED. See
  [Format Conversion Rule](#format-conversion-rule).
- `originalSampleRate` — the sample rate, in Hz, of the originally
  uploaded file. REQUIRED.
- `originalBitDepth` — the bit depth of the originally uploaded file.
  REQUIRED, but `null` for lossy-compressed formats (MP3, AAC, OGG
  Vorbis), which have no fixed-bit-depth PCM concept.
- `durationSeconds` — the track's duration, in seconds. REQUIRED.
- `integratedLufs` — the track's measured integrated loudness, in
  LUFS. OPTIONAL; omitted entirely when loudness matching was not run
  for this test (i.e. the test object's `loudnessMatching` block is
  absent), since no measurement is available in that case.

### Format Conversion Rule

- Only **WAV and AIFF** sources are converted to **FLAC** on export
  (lossless to lossless, with size reduction).
- All other formats (MP3, AAC/M4A, OGG Vorbis, and FLAC itself) MUST
  be kept as-is in `required/`: re-encoding an already-lossy source
  would only reduce quality further without a meaningful size benefit.
- `originalFormat` always reflects the format of the file the user
  originally uploaded. `storedFormat` reflects the format actually
  present in `required/` inside the container. For lossy sources,
  `originalFormat` and `storedFormat` are identical.

### testTypeConfig

Test-type-specific data lives in a namespaced object, keyed by test
type, so the fields shared across the format (`tracks`,
`loudnessMatching`, `playback`, `randomization`, `assets`,
`resources`) remain identical across all test types, and new test
types can be added without breaking older parsers — unknown
`testTypeConfig` keys MUST be ignored by implementations that do not
support them. The object's single key MUST match the test object's own
`testType` field (see [Test Object Schema](#test-object-schema)).

This specification defines the following test types: A/B, A/B/X,
A/B/X→A/B, Ranking, and Rating.

**A/B**:

```json
"testType": "ab",
"testTypeConfig": {
  "ab": {
    "testRounds": 2
  }
}
```

**A/B/X**:

```json
"testType": "abx",
"testTypeConfig": {
  "abx": {
    "testRounds": 8
  }
}
```

**A/B/X→A/B**:

This combined procedure runs two phases back to back, each with its
own independent round count, so it uses two fields instead of a single
`testRounds`:

```json
"testType": "abx-then-ab",
"testTypeConfig": {
  "abx-then-ab": {
    "abxRounds": 8,
    "abRounds": 2
  }
}
```

**Ranking**:

```json
"testType": "ranking",
"testTypeConfig": {
  "ranking": {
    "testRounds": 3
  }
}
```

For `ab`, `abx`, and `ranking`, `testRounds` is the number of rounds
the listener completes. For `abx-then-ab`, `abxRounds` and `abRounds`
are the round counts for the A/B/X phase and the following A/B phase,
respectively. All fields shown above are REQUIRED for their respective
test type.

**Rating**:

```json
"testType": "rating",
"testTypeConfig": {
  "rating": {
    "ratingMode": "identified",
    "ratingCategories": [
      {
        "id": 0,
        "label": "Brightness",
        "scaleType": "numeric",
        "scaleMin": 1,
        "scaleMax": 5,
        "scaleMinLabel": "Dull",
        "scaleMaxLabel": "Bright"
      },
      {
        "id": 1,
        "label": "Low-End",
        "scaleType": "bipolar",
        "scaleRadius": 5,
        "centerLabel": "Just right",
        "negativeLabel": "Too little",
        "positiveLabel": "Too much"
      }
    ]
  }
}
```

- `ratingMode` determines how tracks are presented during the test.
  REQUIRED.
  - `"identified"` shows tracks in their original order under their
    real names (`label`, or `originalFilename` if no label was set).
  - `"blind"` shows tracks in a randomized order under anonymized
    session labels (`A`, `B`, ...), hiding which track is which.
- `ratingCategories` defines the user-defined evaluation categories.
  REQUIRED. Each track in the test object's `tracks[]` is rated once
  per category during the test; the categories themselves are
  test-wide (not per-track), so every track is judged against the same
  set of categories.
  - `id` — a sequential integer starting at `0`, matching the
    category's position among `ratingCategories`. Used to reference
    the category internally (e.g. in result data). This `id` is
    unrelated to `trackId`/`testId`; it is scoped to `ratingCategories`.
  - `label` — the user-facing name shown during the test.
  - `scaleType` — `"numeric"` or `"bipolar"`. Determines which of the
    two scale shapes below applies, and which additional fields on the
    category object are relevant.

  **`scaleType: "numeric"`** — a continuous scale between two bounds:

  - `scaleMin` / `scaleMax` — the numeric bounds of the scale.
    REQUIRED.
  - `scaleMinLabel` / `scaleMaxLabel` — OPTIONAL text labels shown at
    the ends of the rating scale (e.g. "Dull" / "Bright"). This field
    MUST always be present on a `numeric` category; it is `null` when
    not set, in which case implementations fall back to showing only
    the numeric scale.

  **`scaleType: "bipolar"`** — a scale centered on a neutral midpoint,
  for judging whether an attribute is under-, correctly, or
  over-represented relative to a target (e.g. "Low-End: too little ↔
  just right ↔ too much"), rather than rating it on an open, unipolar
  range:

  - `scaleRadius` — how far the scale extends in each direction from
    the center. The resulting scale runs from `-scaleRadius` to
    `+scaleRadius`, with `0` as the fixed, always-present midpoint
    (e.g. `scaleRadius: 5` produces a scale from -5 to +5, 11
    positions total including 0). REQUIRED.
  - `centerLabel` — the label shown at the midpoint (`0`), e.g.
    `"Just right"`. REQUIRED.
  - `negativeLabel` — the label shown at the negative end
    (`-scaleRadius`), e.g. `"Too little"`. REQUIRED.
  - `positiveLabel` — the label shown at the positive end
    (`+scaleRadius`), e.g. `"Too much"`. REQUIRED.
  - `scaleMin`, `scaleMax`, `scaleMinLabel`, and `scaleMaxLabel` are
    not used for this scale type and MUST be omitted.
  - A response to a `bipolar` category is recorded as a single signed
    integer in the range `[-scaleRadius, +scaleRadius]` (e.g. `-3` =
    leaning toward `negativeLabel`, `0` = `centerLabel`, `+2` =
    leaning toward `positiveLabel`).

> Additional test types, such as MUSHRA, are under consideration for a
> future version of this specification but are not yet defined. See
> the project [README](README.md#roadmap) for status.

## Assets

Assets are optional local files that help explain or document the
test set but never change the listening test itself.

Supported asset types: Images (JPG, PNG, WebP), Videos (MP4
recommended), PDF documents.

`testSet.json` references assets as follows, at both the root level
and, optionally, at the level of an individual test object (see
[Field Placement and Overrides](#field-placement-and-overrides)):

```json
"assets": {
  "cover": "assets/cover.jpg",
  "items": [
    { "type": "image", "file": "assets/setup.jpg", "label": "Microphone setup" },
    { "type": "video", "file": "assets/intro.mp4", "label": "Introduction" },
    { "type": "pdf", "file": "assets/measurements.pdf", "label": "Measurement report" }
  ]
}
```

- `cover` — a single OPTIONAL path, used as the test set's preview
  image. Omitted when no cover image is set.
- `items` — an open list of additional assets. Omitted when empty (an
  empty `items: []` is not written).

The `assets` key itself is OPTIONAL and is omitted entirely from
wherever it would otherwise appear in `testSet.json` when neither
`cover` nor `items` has actual content. When `assets` is present, it
MUST contain only the sub-field(s) that actually have content —
`cover` alone, `items` alone, or both — never an empty placeholder for
the field that has no content. For example, a test set with only a
cover image but no additional assets exports as:

```json
"assets": {
  "cover": "assets/cover.jpg"
}
```

Missing assets MUST NOT prevent the test from running.

## Resources

`resources` is an OPTIONAL array of external references that are
**not** embedded in the container (e.g. YouTube videos, manufacturer
pages, whitepapers, AES papers, Git repositories). The key is omitted
entirely when empty.

```json
"resources": [
  { "type": "link", "title": "Manufacturer page", "url": "https://..." },
  { "type": "paper", "title": "AES Convention Paper 12345", "url": "https://..." }
]
```

## Test Results

Results are never stored inside `.batest`. A test's results are linked
to the test set by its `testSetId` (see
[Identifier Stability](#identifier-stability)) in whatever backend or
storage system an implementation uses; this specification does not
define a results format.

## Data Integrity and Security

### File Integrity

File integrity is covered by the ZIP container's own built-in
per-entry CRC32 checksum, which detects corruption during storage or
transfer. Implementations SHOULD rely on this mechanism, together with
standard ZIP validation, when reading a `.batest` file.

### Zip-Slip Protection

When extracting a `.batest` container, implementers **MUST** validate
every entry path before writing it to disk. Entries containing `../`,
absolute paths, or that otherwise resolve outside the intended
extraction directory MUST be rejected. This is a standard ZIP
extraction vulnerability known as "Zip Slip", and applies regardless of
file extension.

### Canonical Serialization for Hashing

Implementers who need to compute a hash over the content of
`testSet.json` or `manifest.json` — for example, to detect whether the
payload has changed, for caching, or for synchronization between
systems — SHOULD compute that hash over the **canonical serialization**
of the JSON content, as defined by [RFC 8785, the JSON Canonicalization
Scheme (JCS)](https://www.rfc-editor.org/rfc/rfc8785), rather than over
the raw file bytes.

Different JSON serializers can produce different key ordering,
whitespace, or number formatting for otherwise identical data. Hashing
raw bytes would therefore cause semantically identical content to
produce different hashes depending on which tool or library wrote the
file. Canonicalizing first removes this variation. JCS implementations
already exist for most common languages, including JavaScript/
TypeScript, Python, and Rust, so implementers do not need to write
their own canonicalizer.

This recommendation applies to any tooling built around the format
(validators, sync tools, caches, etc.); it is not a requirement for
the container format itself, and does not affect how `testSet.json` or
`manifest.json` are written to disk inside a `.batest` file.

## Schema Hygiene Convention

Optional fields in `manifest.json` and `testSet.json` follow a
consistent convention: **when an optional field has no meaningful
value, its key is omitted from the JSON entirely — it is never written
as `null`, an empty object `{}`, or an empty array `[]`.** What counts
as "empty" depends on the field's type: for object-typed fields it
means an empty object or no meaningful sub-fields set; for array-typed
fields it means an empty array; for string-typed fields it means no
value present; for number-typed fields it means no measurement or
value available.

Implementations MUST accept both the omitted-key form and an explicit
`null` (or empty `{}`/`[]`) for these fields, since files exported by
older implementations may still contain the legacy representation.

This convention applies to (among others): `description`, `assets`
(and its `cover` and `items` sub-fields, at whichever level `assets`
appears), `playback`, `randomization`, `resources`, `content`,
`recording` (at whichever level each appears — testSet root, test
object, or track object, per
[Field Placement and Overrides](#field-placement-and-overrides)),
`comparisonCategoryOther`, `comparisonSubcategory`,
`comparisonSubcategoryOther`, `loudnessMatching`, `trackLengthMode`,
each test object's `title` (new in v2.1), each test object's
`soundSource`, `soundSourceOther`, `soundSourceSubtype`, and
`soundSourceSubtypeOther` (new in v2.2), and, on each track object,
`manufacturer`, `model`, `manufacturerOther`, `modelOther`, `notes`,
and `integratedLufs`.

A small number of fields are exceptions and are instead always present
with an explicit `null` value when unset, rather than being omitted:
`creatorUuid` (testSet.json root), `backingTrack` (on each test
object), `label` (on each track object), and
`scaleMinLabel`/`scaleMaxLabel` (on `numeric` rating categories).
`originalBitDepth` (on each track object) is likewise always present,
but is `null` for a different reason: not because the value is unset,
but because the concept of bit depth does not apply to lossy-compressed
formats.

## Identifier Stability

`testSet.json`'s top-level `testSetId` field is the test set's unique,
permanent identifier (a UUIDv7). This identifier **MUST** be treated as
stable and immutable: once assigned to a test set, it MUST NOT change,
even across subsequent edits to any other field in `testSet.json`.
Systems that reference a test set externally (e.g. to associate
results with it, per [Test Results](#test-results)) rely on
`testSetId` remaining constant for the lifetime of the test set.

This is unrelated to the per-test `testId` and per-track `trackId`
fields described above, both of which are positional indices scoped to
their containing array (`test` and a given test object's `tracks`,
respectively) rather than standalone stable identifiers — they
renumber if their containing array is reordered, unlike `testSetId`.

## Versioning and Compatibility

- `formatVersion` (in `manifest.json`) tracks the container/schema
  version, starting at `1`.
- Adding new optional fields, new test type values, or new
  `testTypeConfig` namespaces is considered a **non-breaking** change
  and does not require a `formatVersion` bump. Implementations MUST
  ignore unknown fields and unknown `testTypeConfig` keys rather than
  fail.
- Removing or repurposing existing fields, or changing the meaning of
  an existing field, is a **breaking** change and requires
  incrementing `formatVersion`. Implementations MUST detect an
  unsupported (higher) `formatVersion` on import and reject the file
  with a clear error message, rather than attempting to interpret it
  incorrectly.

## Migration from v1

v2 (`formatVersion: 2`) is a breaking revision of v1
(`formatVersion: 1`). A v1 `test.json` file is **not** structurally
compatible with v2's `testSet.json` and requires conversion; this is a
conceptual summary of what changed, not a full migration guide:

- The container file itself is renamed: `test.json` → `testSet.json`
  (and its schema, `schema/test.schema.json` → `schema/testSet.schema.json`).
- The document's shape changed from describing a single test to
  describing a **test set**: a new required `test` array holds one or
  more test objects. A v1 `test.json` maps to a `test` array with
  exactly one object (`testId: 0`).
- `test.json`'s top-level `id` is renamed to `testSetId`.
- `tracks`, `loudnessMatching`, `trackLengthMode`, `backingTrack`, and
  `testTypeConfig` move from the document root into each test object.
- Each test object requires a new `testId` (its position in `test`)
  and a new `testType` field (previously implicit in
  `testTypeConfig`'s single key).
- Each track object's `id` is renamed to `trackId`, and its uniqueness
  is now scoped to its parent test object's `tracks` array rather than
  to the whole document (which, for a v1-derived single-test-object
  file, is equivalent).
- `content` gains a new valid level: it can now also be set on a test
  object, in addition to the document root and track object, with
  track overriding test overriding root. `recording` is no longer
  valid at the document root at all (test object and track object
  only). `assets` gains a new valid level: it can now also be set on a
  test object, in addition to the document root. See
  [Field Placement and Overrides](#field-placement-and-overrides).
- `manifest.json`'s `testType` field is removed (a test set may now
  contain test objects of different types, so a single top-level value
  no longer applies).

No automatic v1-to-v2 conversion tool is defined by this specification.

## Guiding Principle

A `.batest` file represents one or more fully reproducible listening
tests. Required assets make the test set executable. Optional assets
(including embedded videos) provide additional context. External
resources complement the package without increasing its size
unnecessarily.
