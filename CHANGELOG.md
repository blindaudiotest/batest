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
