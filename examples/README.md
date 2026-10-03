# Examples

No complete example `.batest` archives are included yet.

Unpacked examples (JSON documents only, without audio files):

- [`track-variants/`](track-variants/): a test set comparing two
  microphones (KM 184, CC 8), each recorded at 80 mm and 160 mm. It
  uses `itemIndex`, `trackOption: "distance"`, `distanceMm`,
  `highPass`, and `polarPattern` (new in v2.7). Its audio files are
  not included, so it cannot be played.

Example files will be added here to demonstrate the format in
practice — one or more `.batest` archives covering the different test
types defined in [SPEC.md](../SPEC.md) (A/B, A/B/X, A/B/X→A/B,
Ranking, and Rating).

Once example files are added, the CI workflow
([`.github/workflows/validate.yml`](../.github/workflows/validate.yml))
will automatically extract and validate each one's `manifest.json` and
`testSet.json` against the schemas in [`schema/`](../schema/).

Example files in this directory are licensed under the MIT License
(see [`LICENSE-CODE`](../LICENSE-CODE)).
