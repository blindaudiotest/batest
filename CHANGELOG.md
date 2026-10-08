# Changelog

All notable changes to the `.batest` file format specification are
documented in this file.

This changelog begins at **v1.0**, the initial stable public release
of the specification. The format is versioned independently of any
implementation, following the rules in
[SPEC.md's Versioning and Compatibility section](SPEC.md#versioning-and-compatibility):
breaking changes to existing fields or structures increment the major
version; additive, backward-compatible changes can go into a minor
version.

## [2.9] - 2026-10-08

### Added

- Added an OPTIONAL `originalTrack` field to each test object
  (`testSet.json`, `test[]`), a sibling of `backingTrack`. It carries
  the unprocessed source signal of the test, so that listeners can
  switch between the original and the processed (compared) track, or
  blend between them. It is intended for comparisons of signal
  processing, but the format does not tie it to a category. The key is
  omitted entirely when unused (never `null`).
  - Audio fields, with the same meaning as on a track object:
    `filename` (in `required/`), `originalFilename`, `originalFormat`,
    `storedFormat`, `originalSampleRate`, `originalBitDepth` (`null`
    for lossy formats), `durationSeconds`, and OPTIONAL
    `integratedLufs`. The Format Conversion Rule applies. There is no
    `trackId`, `manufacturer`, `model`, `itemIndex`, `label`, or
    recording metadata. `manifest.json` is unchanged.
  - `mode` (REQUIRED): `"switch"` (toggle; playback starts in the
    processed state) or `"blend"` (continuous crossfade).
  - `defaultBlendPercent` (integer, `0`–`100`): the initial share of
    the processed signal. REQUIRED for `"blend"`, forbidden for
    `"switch"` (enforced by the JSON Schema). Its meaning is defined by
    a normative equal-power curve: with `p = defaultBlendPercent / 100`,
    `gOriginal = cos(p·π/2)` and `gProcessed = sin(p·π/2)`.
  - Valid for all test types, including `"rating"`, and combinable
    with `swappedSetup`, `trackOption`, and `backingTrack`. One
    original applies to all compared tracks of the test. Several tests
    MAY reference the same original file.
  - The original is independent of the compared tracks: its
    `durationSeconds` does not count for `trackLengthMode` or for
    deciding whether the compared tracks differ in length. It SHOULD
    play position-synchronous with the compared tracks, padded with
    silence if shorter and cut off if longer.
- Added the value `"original"` to `loudnessMatching.reference`. When a
  test object has both `originalTrack` and `loudnessMatching`,
  `reference` MUST be `"original"`; `"original"` without
  `originalTrack` is invalid. Both rules are enforced by the JSON
  Schema. `loudnessMatching.version` is unchanged.
- New prose rule: implementations SHOULD treat an unknown
  `loudnessMatching.reference` value like `"loudest"` and warn.
- Added the example `examples/original-track/` (`manifest.json` and
  `testSet.json` only, no audio): one A/B test with `mode: "blend"`
  and loudness matching to the original, and one Rating test with
  `mode: "switch"` sharing the same original file.
- This is a non-breaking, additive change. `formatVersion` remains `2`,
  and files written by any v2.0–v2.8 implementation remain fully valid
  under v2.9. A file that uses `originalTrack` does not validate
  against the v2.8 schema, whose test object does not allow additional
  properties.

## [2.8] - 2026-10-05

### Added

- Added `"wide-cardioid"` as a new value of the track-level
  `polarPattern` field (`testSet.json`, `test[].tracks[]` and
  `swappedSetup.tracks[]`), in both SPEC.md and
  `schema/testSet.schema.json`. The name follows the existing
  convention for these values: lowercase, with a hyphen between the
  parts of a multi-part term (as in `"figure-8"`).
- This is a non-breaking, additive change. `formatVersion` remains `2`,
  no existing value was removed or changed in meaning, and files
  written by any v2.0–v2.7 implementation remain fully valid under
  v2.8. A file that uses `"wide-cardioid"` does not validate against
  the v2.7 schema, whose `polarPattern` enum is closed.

## [2.7] - 2026-10-03

### Added

- Added four OPTIONAL, track-only fields to the track object
  (`testSet.json`, `test[].tracks[]` and `swappedSetup.tracks[]`).
  They are deliberately not part of `recording` and have no
  override/merge rule and no test-level default:
  - `itemIndex` (integer, `>= 0`): a test-set-wide structural grouping
    key for the item under comparison. Tracks with the same
    `itemIndex` refer to the same item, which makes several tracks of
    the same item within one test representable. Values are assigned
    densely from `0`. Either all tracks of a test set carry it or none
    does. Tracks sharing a value MUST carry identical
    `manufacturer`/`model`/`manufacturerOther`/`modelOther`.
  - `distanceMm` (integer, `>= 1`): recording distance in millimetres.
  - `highPass`: a high-pass filter applied in the recording chain of
    the track, with three states. Key omitted (or legacy `null`) means
    not specified. `{ "enabled": false }` means explicitly off.
    `{ "enabled": true, "frequencyHz": <integer >= 1> }` means on.
  - `polarPattern`: one of `"cardioid"`, `"omni"`, `"figure-8"`,
    `"supercardioid"`, `"hypercardioid"`.
- Added an OPTIONAL `trackOption` field to each test object (`"distance"`
  | `"highPass"` | `"polarPattern"`), naming which of these track-level
  categories varies between the test's tracks. The values live only on
  the tracks; there is no separate value list. When `trackOption` is
  set, every track in `tracks[]` and `swappedSetup.tracks[]` MUST carry
  the mapped field (enforced by the JSON Schema). Tracks sharing an
  `itemIndex` within a test MUST have distinct values. Each
  `swappedSetup.tracks[i]` MUST match `tracks[i]` in `itemIndex` and in
  the option value. No rectangular-grid requirement applies. These
  rules are prose-only. Pairing and playback semantics for tests with
  variants are implementation-defined.
- Added the first example, `examples/track-variants/` (`manifest.json`
  and `testSet.json` only, no audio): KM 184 and CC 8, each at 80 mm
  and 160 mm.
- The CI workflow now also validates unpacked example directories
  (`examples/*/manifest.json`, `examples/*/testSet.json`).
- This is a non-breaking, additive change. `formatVersion` remains `2`,
  and files written by any v2.0–v2.6 implementation remain fully valid
  under v2.7.

## [2.6] - 2026-09-04

### Changed

- Clarified that a track object's `manufacturer` / `model` fields
  (`testSet.json`, `test[].tracks[]`) MUST hold the stable,
  human-readable name of the item under comparison (e.g. `"Neumann"` /
  `"U87"`) for a known, non-free-text selection — never a database ID,
  internal slug, or other instance-dependent identifier tied to the
  application/database instance that produced the file. Added an
  explicit example for the free-text "Others" case, where
  `manufacturer`/`model` instead carry the raw sentinel value
  `"others"` and the actual free-text name lives in
  `manufacturerOther`/`modelOther`, mirroring the existing
  `comparisonCategory`/`comparisonCategoryOther` pattern.
- Added `minLength: 1` to `manufacturer` and `model` in
  `schema/testSet.schema.json`'s track object definition, matching the
  clarified prose (an empty string is never a valid resolved name).
- This is a non-breaking clarification: `formatVersion` remains `2`,
  no field was added, removed, or changed in shape, and files written
  by any v2.0–v2.5 implementation remain fully valid under v2.6. This
  specification does not define a migration or backfill step for
  `.batest` files that predate this clarification and may contain a
  raw identifier in these fields instead of a human-readable name;
  consuming applications handle any resulting display fallback on
  their own.

## [2.5] - 2026-09-03

### Added

- Added an OPTIONAL `swappedSetup` field to each test object in
  `testSet.json`'s `test` array. `swappedSetup` describes a second,
  position-swapped physical setup for that test's A/B comparison (e.g.
  swapped microphone-to-preamp assignment), used to reduce the
  influence of a position-dependent bias unrelated to the items
  actually being compared. It is only valid when that test object's
  `testType` is `"ab"` or `"abx-then-ab"`; on an `"abx-then-ab"` test
  object it applies only to the A/B phase, never the preceding A/B/X
  identification phase, which continues to use only the regular
  `tracks[]` array unchanged. `swappedSetup.tracks[]` follows the same
  Track Object Schema as the sibling `tracks[]` array and is coupled to
  it 1:1 by array position (not by a separate ID reference), mirroring
  the format's existing positional `trackId`/`testId` convention. This
  1:1 coupling, and the accompanying rule that matched
  `manufacturer`/`model` pairs be preserved between a track and its
  swapped counterpart, are documented as normative SPEC.md prose only
  and are deliberately not mechanically enforced by the JSON Schema —
  consistent with how the existing positional `trackId` rule is
  handled. The key is omitted entirely when no swapped setup was
  recorded, per the existing Schema Hygiene Convention.
- This is a non-breaking, additive change: `formatVersion` remains `2`,
  and files written by a v2.0, v2.1, v2.2, v2.3, or v2.4 implementation
  (which omit this field) remain fully valid under v2.5 and are
  correctly interpreted as having no swapped setup recorded.

## [2.4] - 2026-09-02

### Added

- Added an OPTIONAL `requiresSuccessfulAbx` field to
  `testTypeConfig.abx-then-ab` (alongside `abxRounds`/`abRounds`).
  When `true`, an A/B/X→A/B test procedure's A/B phase only runs after
  the listener completed the preceding A/B/X phase successfully; when
  `false` (or omitted), the A/B phase always runs, matching the only
  behavior available before this field existed. The key is omitted
  entirely when `false`, per the existing Schema Hygiene Convention.
- This is a non-breaking, additive change: `formatVersion` remains
  `2`, and files written by a v2.0, v2.1, v2.2, or v2.3 implementation
  (which omit this field) remain fully valid under v2.4 and are
  correctly interpreted as "always run".

## [2.3] - 2026-09-02

### Added

- Added an OPTIONAL `poolAbxAcrossTests` field to `testSet.json`'s
  root level. When `true`, all A/B/X test procedures within the test
  set (every `abx` test object, and the A/B/X phase of every
  `abx-then-ab` test object) are evaluated together as a single pooled
  result rather than each being evaluated separately. This is the
  first plain boolean field in the specification; it follows the
  existing Schema Hygiene Convention by omitting the key entirely
  when `false` (the default, and the only behavior definable before
  this field existed) rather than writing the default out explicitly.
- This is a non-breaking, additive change: `formatVersion` remains
  `2`, and files written by a v2.0, v2.1, or v2.2 implementation
  (which omit this field) remain fully valid under v2.3 and are
  correctly interpreted as using separate, per-test ABX evaluation.

## [2.2] - 2026-08-26

### Added

- Added four OPTIONAL fields to each test object in `testSet.json`'s
  `test` array: `soundSource`, `soundSourceOther`, `soundSourceSubtype`,
  and `soundSourceSubtypeOther`. These identify the kind of source
  material that individual test's tracks are drawn from (e.g.
  `"vocals"`, `"acoustic-guitar"`), following the same
  raw-value-plus-free-text-fallback pattern already used by
  `comparisonCategory`/`comparisonCategoryOther` at the `testSet.json`
  root, but scoped per test object rather than per test set, since
  different tests within the same test set can compare different kinds
  of source material. Note that the free-text sentinel for these fields
  is exactly `"other"` (singular), unlike `comparisonCategory`'s
  `"others"` — an intentional but easy-to-miss difference. All four
  keys are omitted entirely when not set, per the existing Schema
  Hygiene Convention. `manifest.json` is unaffected: since
  `soundSource` can vary per test within the same test set (like
  `testType`), it is not surfaced as a single top-level summary field
  there.
- This is a non-breaking, additive change: `formatVersion` remains `2`,
  and files written by a v2.0 or v2.1 implementation (which omit these
  fields) remain fully valid under v2.2. None of the new fields were
  added to the test object's `required` list, specifically to preserve
  this backward compatibility.

## [2.1] - 2026-08-21

### Added

- Added an OPTIONAL `title` field to each test object in
  `testSet.json`'s `test` array, letting an individual test within a
  test set carry its own display title, independent of the test set's
  root-level `title`. The key is omitted entirely when not set, per
  the existing Schema Hygiene Convention. This is a non-breaking,
  additive change: `formatVersion` remains `2`, and files written by a
  v2.0 implementation (which omit this field) remain fully valid under
  v2.1.

## [2.0] - 2026-08-01

### Breaking

- Renamed `test.json` to `testSet.json` (and its schema,
  `schema/test.schema.json` to `schema/testSet.schema.json`).
- Renamed `testSet.json`'s top-level `id` field to `testSetId`.
- Renamed each track object's `id` field to `trackId`, now scoped to
  its parent test object's `tracks` array instead of the whole
  document.
- Restructured `testSet.json` around a new required `test` array
  (minimum 1 item): each entry is a test object with a required
  `testId` (zero-based integer, matching its position in `test`) and a
  required `testType`.
- Moved `tracks`, `loudnessMatching`, `trackLengthMode`,
  `backingTrack`, and `testTypeConfig` out of `testSet.json`'s root and
  into each test object.
- `content` is now also optionally available at the test-object level,
  in addition to the `testSet.json` root and track level; `recording`
  is now scoped to the test-object and track level only (no longer
  valid at the `testSet.json` root); `assets` is now also optionally
  available at the test-object level, in addition to the
  `testSet.json` root. Where a field is present at multiple levels for
  the same track, the more specific level overrides the less specific
  one, per field (see SPEC.md's "Field Placement and Overrides").
- Removed `testType` from `manifest.json` (a test set may now contain
  test objects of different types, so a single top-level value no
  longer applies).
- `formatVersion` incremented to `2`.

## [1.0] - 2026-07-16

### Added

- Initial public release of the `.batest` specification.
- ZIP-based container structure: `manifest.json`, `test.json`,
  `required/`, `assets/`, `resources/`.
- `manifest.json` schema: lightweight test summary.
- `test.json` schema: shared fields (`content`, `recording`,
  `backingTrack`, `loudnessMatching`, `trackLengthMode`, `assets`,
  `resources`) and the per-track object schema.
- Five defined test types under `testTypeConfig`: A/B, A/B/X,
  A/B/X→A/B, Ranking, and Rating (with `numeric` and `bipolar` scale
  types).
- FLAC conversion rule for WAV/AIFF sources; other formats preserved
  as-is.
- Schema hygiene convention: optional empty fields are omitted rather
  than written as `null`.
- Identifier stability requirement for `test.json`'s `id` field.
- Zip-Slip extraction security guidance for implementers.
- Canonical serialization (RFC 8785 / JCS) recommendation for hashing
  `test.json` or `manifest.json` content.
- JSON Schemas (Draft 2020-12) for `manifest.json` and `test.json`.
