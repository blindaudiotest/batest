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
