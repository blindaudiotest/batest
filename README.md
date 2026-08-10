# batest

`.batest` is an open, ZIP-based file format for storing reproducible
blind audio comparison test data: the audio tracks being compared,
the metadata needed to reproduce the listening experience, and
optional supporting material. It was created for the
[Blind Audio Test](https://blindaudiotest.com) project, so that
listening tests can be packaged, shared, and re-run independently of
any single application.

This repository contains the format's specification, JSON Schemas,
and supporting material for third-party developers who want to read
or write `.batest` files.

## Specification

The authoritative specification lives in [SPEC.md](SPEC.md). If you
are implementing a `.batest` reader or writer, start there.

Machine-readable JSON Schemas (Draft 2020-12) for the format's two
JSON documents are in [`schema/`](schema/):

- [`schema/manifest.schema.json`](schema/manifest.schema.json)
- [`schema/testSet.schema.json`](schema/testSet.schema.json)

## Status

The specification is versioned independently of any implementation.
The current version is **v2**, a breaking revision of v1. **v1**
remains available for anyone still implementing against it, via this
repository's `v1.0` git tag/release. See [CHANGELOG.md](CHANGELOG.md)
for release history.

## Roadmap

- **MUSHRA** support is planned and conceptual only — it is **not**
  yet part of this specification. Field names and structure for a
  future MUSHRA test type are not finalized and are subject to change
  once the test type is actually implemented and specified.
- A user-defined **Rating** test type (with `numeric` and `bipolar`
  scale types) is already part of v1 — see [SPEC.md](SPEC.md#testtypeconfig).

## License

This repository is dual-licensed:

- The specification and other prose documentation (`SPEC.md`,
  `README.md`, and related Markdown files) are licensed under
  [CC-BY-4.0](LICENSE).
- The JSON Schemas and example files (`schema/`, `examples/`) are
  licensed under the [MIT License](LICENSE-CODE).

## Contributing

Contributions and proposals are welcome. See
[CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes.
