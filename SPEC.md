# Blind Audio Test File Format Specification (`.batest`)

**Version:** 1.0
**Status:** Stable — first public release

This document is the authoritative specification of the `.batest` file
format, an open, ZIP-based container format for storing reproducible
blind audio comparison tests.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119.

## Table of Contents

1. [Overview](#overview)
2. [Design Goals](#design-goals)
3. [File Extension and Container Format](#file-extension-and-container-format)
4. [Directory Structure](#directory-structure)
5. [manifest.json](#manifestjson)
6. [test.json](#testjson)
   1. [Top-Level Fields](#top-level-fields)
   2. [content and recording](#content-and-recording)
   3. [backingTrack](#backingtrack)
   4. [loudnessMatching](#loudnessmatching)
   5. [trackLengthMode](#tracklengthmode)
   6. [Track Object Schema](#track-object-schema)
   7. [Format Conversion Rule](#format-conversion-rule)
   8. [testTypeConfig](#testtypeconfig)
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
14. [Guiding Principle](#guiding-principle)

## Overview

A `.batest` file is a ZIP container that fully describes a blind audio
comparison test: the audio tracks being compared, the metadata needed
to reproduce the listening experience, and optional supporting
material (cover images, documentation, external references). A
conforming `.batest` file is self-contained and MUST be playable
offline, without any network access, by any implementation that
supports the test type it contains.

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
├── test.json
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

- `manifest.json` and `test.json` MUST be present at the root of the
  archive.
- `required/` contains every audio file that is necessary to run the
  test (tracks, and, if present, the backing track). Implementations
  MUST treat the files in this directory as required for playback.
- `assets/` contains optional supporting files (images, video, PDF
  documents) referenced from `test.json`. See [Assets](#assets).
- `resources/` is reserved for future use; external references that
  are not embedded in the container are instead listed in
  `test.json`'s `resources` array (see [Resources](#resources)).

## manifest.json

`manifest.json` is a lightweight, redundant summary of the test. It
allows a file browser or import dialog to display key information
without unzipping and parsing `test.json`.

```json
{
  "formatVersion": 1,
  "createdWith": "Blind Audio Test 1.0.0",
  "createdAt": "2026-07-10T18:30:00Z",
  "title": "Example Test",
  "comparisonCategory": "microphones",
  "testType": "abx",
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
- `title` is the test's display title.
- `comparisonCategory` MUST be the resolved, human-readable value: if
  the author selected a free-text "Others" category, the free-text
  value is substituted in directly here. This differs from
  `test.json`, where `comparisonCategory` stays the raw category value
  and the free text lives in its own `comparisonCategoryOther` field
  (see [Top-Level Fields](#top-level-fields)) — `manifest.json`'s
  `comparisonCategory` is a display-only derivative, not a copy, of
  `test.json`'s.
- `testType` identifies the test type (see
  [testTypeConfig](#testtypeconfig) for the defined values). It exists
  purely so a file browser or import dialog can show the test type
  without parsing `test.json`. This is the only place a test's type is
  stored as its own field; in `test.json` the test type is identified
  by `testTypeConfig`'s single key instead.
- `models` lists the distinct manufacturer/model combinations used
  across all tracks in the test, deduplicated. If a track used a
  free-text "Others" manufacturer/model, that free-text value is used
  here as well. `models` MUST be an empty array if the test has no
  manufacturer/model data (e.g. a Mixes/Masters or Codecs comparison).

## test.json

`test.json` is the primary, authoritative description of the test.

```json
{
  "id": "UUIDv7",
  "creatorUuid": null,
  "title": "Example Test",
  "description": "...",
  "comparisonCategory": "microphones",
  "content": {
    "type": "vocals",
    "genres": ["indie-pop"],
    "tags": ["soft", "breathy"]
  },
  "recording": {
    "environment": "studio",
    "room": {
      "type": "vocal-booth",
      "sizeSqm": 6,
      "acoustics": "dampened"
    },
    "signalChain": [
      { "type": "preamp", "manufacturer": "Millennia", "model": "HV-3C" },
      { "type": "converter", "manufacturer": "RME", "model": "ADI-2 Pro FS" }
    ],
    "tags": ["Studio XYZ", "Stereo XY", "close-miking", "professional-recording"]
  },
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
  "testTypeConfig": {}
}
```

`description` and `assets` are omitted from the example above; see
[Schema Hygiene Convention](#schema-hygiene-convention) and
[Assets](#assets) for when each is present versus omitted.

### Top-Level Fields

- `id` is the test's unique identifier (a UUIDv7). REQUIRED. See
  [Identifier Stability](#identifier-stability) for normative
  requirements on this field.
- `creatorUuid` is an OPTIONAL identifier for the person or account
  that created the test (e.g. for attribution once publishing/sharing
  is supported). This field MUST always be present in `test.json`; it
  is `null` when not set (e.g. an anonymous local export). Unlike most
  other optional fields in this format, `creatorUuid` is represented
  by an explicit `null`, not by omitting the key — see
  [Schema Hygiene Convention](#schema-hygiene-convention).
- `title` is the test's display title. REQUIRED.
- `description` is an OPTIONAL free-text description of the whole
  test. The key is omitted entirely when no description was set.
- `comparisonCategory` is the raw category value chosen when creating
  the test (e.g. `"microphones"` or `"others"`). REQUIRED.
- `comparisonCategoryOther` holds the free-text specification when
  `"others"` was selected for `comparisonCategory`. Omitted otherwise.
- `comparisonSubcategory` is only relevant for categories that define
  subcategories (currently only `"effects"`). Omitted for all other
  categories, and also omitted when the category defines subcategories
  but none was selected.
- `comparisonSubcategoryOther` holds the free-text specification when
  `"others"` was selected for `comparisonSubcategory`. Omitted
  otherwise.
- `content` — see [content and recording](#content-and-recording).
- `recording` — see [content and recording](#content-and-recording).
- `tracks` is the array of track objects being compared. REQUIRED. See
  [Track Object Schema](#track-object-schema).
- `backingTrack` — see [backingTrack](#backingtrack).
- `loudnessMatching` — see [loudnessMatching](#loudnessmatching).
- `trackLengthMode` — see [trackLengthMode](#tracklengthmode).
- `playback` and `randomization` are open objects reserved for future
  fields; this specification does not yet define any fields for them.
  Both are OPTIONAL and omitted entirely when they hold no content.
- `assets` — see [Assets](#assets).
- `resources` — see [Resources](#resources).
- `testTypeConfig` — see [testTypeConfig](#testtypeconfig). REQUIRED.

### content and recording

`content` and `recording` are both OPTIONAL and exist at both test
level and track level (see
[Track Object Schema](#track-object-schema) for the track-level
variant). They describe the audio content and recording setup for
context — e.g. "what is being listened to, and how was it captured" —
and have no effect on playback, randomization, or scoring. Each key is
omitted entirely when there is no content to add, rather than being
present with a `null` or empty value.

`content` describes what is being compared:

- `type` — a free-text or open-vocabulary label for the source
  material (e.g. `"vocals"`, `"acoustic-guitar"`, `"full-mix"`).
- `genres` — an OPTIONAL list of applicable genre tags.
- `tags` — an OPTIONAL list of free-form descriptive tags (e.g.
  `"soft"`, `"breathy"`).

`recording` describes the recording setup that is shared across all
tracks in the test (e.g., when comparing microphones, the room,
performer/instrument, and everything downstream of the microphone —
preamp, converter, etc. — is typically identical across tracks; only
the microphone itself differs, and that distinguishing detail belongs
at track level instead):

- `environment` — e.g. `"studio"`, `"live"`, `"field"`,
  `"home-studio"`.
- `room` — a free-form object describing the space (e.g. `type`,
  `sizeSqm`, `acoustics` such as `"dampened"`, `"live"`, or
  `"neutral"`, `notes`). All sub-fields are optional; the `room` object
  itself MAY be `null` or omitted entirely.
- `instrument` — a free-form object describing the instrument or voice
  being recorded. `null` or omitted when not applicable, or when not
  shared across tracks (e.g. in an instrument comparison test, the
  instrument differs per track and belongs under the track's own
  `recording.instrument` instead).
- `signalChain` — an ordered array describing the recording chain
  shared by all tracks (e.g. preamp and converter used regardless of
  which microphone is being compared). Each entry has `type` (e.g.
  `"microphone"`, `"preamp"`, `"converter"`, `"cable"`, `"other"`),
  `manufacturer`, and `model`. In a microphone comparison test, the
  shared chain typically starts at `preamp`, since the microphone
  itself differs per track and is not part of the shared
  `signalChain`.
- `tags` — a free-form list of additional descriptive tags about the
  recording (e.g. `"Studio XYZ"`, `"Stereo XY"`, `"close-miking"`).

At track level, `content` and `recording` use the exact same shape as
above, but represent only a **delta**: only fields that differ from
track to track belong there. A consuming application MUST merge
test-level and track-level values per field, with the track-level
value winning when present and the test-level value applying
otherwise. Arrays such as `signalChain` and `tags` are NOT merged
element-by-element: if a track provides its own `signalChain`, that
array replaces the test-level `signalChain` for that track. How a
replaced/combined chain is presented for display is left to the
application; the format itself only stores the two arrays separately.

A typical usage in a microphone comparison test: `content` is entirely
test-level (identical across tracks), while each track sets its own
`recording.signalChain` with a single `"microphone"` entry — the one
thing that actually differs between tracks. All other recording
details (room, preamp, converter) stay at test level, since they are
identical for every take.

### backingTrack

`backingTrack` is an OPTIONAL, single audio track that plays
continuously in the background throughout the test (e.g. a full music
production over which the individually compared elements — instrument
takes, effects, etc. — are layered). It is not part of the blind
comparison itself: it plays unchanged, in parallel, for every track
being evaluated.

This field MUST always be present in `test.json`; it is `null` when
the test does not use a backing track (see
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

`loudnessMatching` describes the loudness-matching behavior used for
the test. The key is OPTIONAL and is omitted entirely when loudness
matching was not enabled for the test — there is no `enabled: false`
representation; the block's mere presence in `test.json` means it was
enabled.

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
handled during playback:

- `"shortest"` — playback ends when the shortest compared track
  finishes.
- `"longest"` — playback continues until the longest compared track
  finishes, with shorter tracks going silent once they end.

This field is OPTIONAL and is only meaningful — and only ever written
— when the test's tracks actually have different durations; it is
omitted entirely when all tracks have the same length (or there is
only one track). Implementations MUST assume `"shortest"` when the key
is absent but the tracks do have different lengths.

`trackLengthMode` does not apply to `testTypeConfig.rating`: a Rating
test presents one track at a time rather than playing multiple tracks
in sync, so this field is never written for that test type, regardless
of whether the tracks have different durations. Implementations MUST
ignore this key if present on a Rating test (e.g. from a file exported
by an older implementation).

### Track Object Schema

Each entry in `tracks[]` follows this schema. Track identifiers are
sequential (`track1`, `track2`, ...), independent of test type, so the
same schema is used for A/B, A/B/X, and multitrack Ranking tests alike.

```json
{
  "id": "track1",
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

- `id` — the track's sequential identifier (e.g. `"track1"`). REQUIRED.
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
  `id`/ordering. This field MUST always be present; it is `null` when
  not set.
- `notes` — an OPTIONAL free-text note about the track. The key is
  omitted entirely when no note was set.
- `content` / `recording` — OPTIONAL track-level delta objects; see
  [content and recording](#content-and-recording). Omitted entirely
  when there is nothing track-specific to add.
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
  for the test (i.e. the test-level `loudnessMatching` block is
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
support them. The object's single key **is** the test type; `test.json`
has no separate top-level `testType` field (that field exists only in
`manifest.json`, see [manifest.json](#manifestjson)).

This specification defines the following test types: A/B, A/B/X,
A/B/X→A/B, Ranking, and Rating.

**A/B**:

```json
"testTypeConfig": {
  "ab": {
    "testRounds": 2
  }
}
```

**A/B/X**:

```json
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
"testTypeConfig": {
  "abx-then-ab": {
    "abxRounds": 8,
    "abRounds": 2
  }
}
```

**Ranking**:

```json
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
  REQUIRED. Each track in `tracks[]` is rated once per category during
  the test; the categories themselves are test-wide (not per-track),
  so every track is judged against the same set of categories.
  - `id` — a sequential integer starting at `0`, matching the
    category's position among `ratingCategories`. Used to reference
    the category internally (e.g. in result data).
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
test but never change the listening test itself.

Supported asset types: Images (JPG, PNG, WebP), Videos (MP4
recommended), PDF documents.

`test.json` references assets as follows:

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

- `cover` — a single OPTIONAL path, used as the test's preview image.
  Omitted when no cover image is set.
- `items` — an open list of additional assets. Omitted when empty (an
  empty `items: []` is not written).

The `assets` key itself is OPTIONAL and is omitted entirely from
`test.json` when neither `cover` nor `items` has actual content. When
`assets` is present, it MUST contain only the sub-field(s) that
actually have content — `cover` alone, `items` alone, or both — never
an empty placeholder for the field that has no content. For example, a
test with only a cover image but no additional assets exports as:

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
to the test by its `id` (see [Identifier Stability](#identifier-stability))
in whatever backend or storage system an implementation uses; this
specification does not define a results format.

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

Implementers who need to compute a hash over the content of `test.json`
or `manifest.json` — for example, to detect whether the payload has
changed, for caching, or for synchronization between systems — SHOULD
compute that hash over the **canonical serialization** of the JSON
content, as defined by [RFC 8785, the JSON Canonicalization Scheme
(JCS)](https://www.rfc-editor.org/rfc/rfc8785), rather than over the
raw file bytes.

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
the container format itself, and does not affect how `test.json` or
`manifest.json` are written to disk inside a `.batest` file.

## Schema Hygiene Convention

Optional fields in `manifest.json` and `test.json` follow a consistent
convention: **when an optional field has no meaningful value, its key
is omitted from the JSON entirely — it is never written as `null`, an
empty object `{}`, or an empty array `[]`.** What counts as "empty"
depends on the field's type: for object-typed fields it means an empty
object or no meaningful sub-fields set; for array-typed fields it
means an empty array; for string-typed fields it means no value
present; for number-typed fields it means no measurement or value
available.

Implementations MUST accept both the omitted-key form and an explicit
`null` (or empty `{}`/`[]`) for these fields, since files exported by
older implementations may still contain the legacy representation.

This convention applies to (among others): `description`, `assets`
(and its `cover` and `items` sub-fields), `playback`, `randomization`,
`resources`, `content`, `recording` (at both test level and track
level), `comparisonCategoryOther`, `comparisonSubcategory`,
`comparisonSubcategoryOther`, and, on each track object, `manufacturer`,
`model`, `manufacturerOther`, `modelOther`, `notes`, and
`integratedLufs`.

A small number of fields are exceptions and are instead always present
with an explicit `null` value when unset, rather than being omitted:
`creatorUuid`, `backingTrack` (both at top level), `label` (on each
track object), and `scaleMinLabel`/`scaleMaxLabel` (on `numeric`
rating categories). `originalBitDepth` (on each track object) is
likewise always present, but is `null` for a different reason: not
because the value is unset, but because the concept of bit depth does
not apply to lossy-compressed formats.

## Identifier Stability

`test.json`'s top-level `id` field is the test's unique, permanent
identifier (a UUIDv7). This identifier **MUST** be treated as stable
and immutable: once assigned to a test, it MUST NOT change, even
across subsequent edits to any other field in `test.json`. Systems
that reference a test externally (e.g. to associate results with it,
per [Test Results](#test-results)) rely on `id` remaining constant for
the lifetime of the test.

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

## Guiding Principle

A `.batest` file represents a fully reproducible listening test.
Required assets make the test executable. Optional assets (including
embedded videos) provide additional context. External resources
complement the package without increasing its size unnecessarily.
