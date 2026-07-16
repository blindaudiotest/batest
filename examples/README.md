# Examples

This directory is a placeholder. No example `.batest` files are
included yet.

Example files will be added here to demonstrate the format in
practice — one or more `.batest` archives covering the different test
types defined in [SPEC.md](../SPEC.md) (A/B, A/B/X, A/B/X→A/B,
Ranking, and Rating).

Once example files are added, the CI workflow
([`.github/workflows/validate.yml`](../.github/workflows/validate.yml))
will automatically extract and validate each one's `manifest.json` and
`test.json` against the schemas in [`schema/`](../schema/).

Example files in this directory are licensed under the MIT License
(see [`LICENSE-CODE`](../LICENSE-CODE)).
