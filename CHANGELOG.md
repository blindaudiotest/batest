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
