# Contributing

Thanks for your interest in the `.batest` file format.

## Maintainership

This format is currently maintained by a single maintainer, Oliver
Laschke, for the [Blind Audio Test](https://blindaudiotest.com)
project. Proposals, questions, and discussion from the community are
very welcome, but final decisions on the specification rest with the
maintainer.

## Proposing changes

1. Before submitting a pull request, please open a
   [GitHub Issue](../../issues) describing the change you'd like to
   make and why. This gives space to discuss the proposal, alternative
   approaches, and any compatibility implications before any code or
   spec text is written.
2. Once there's rough agreement on the approach, feel free to open a
   pull request that implements it.

## Breaking vs. non-breaking changes

- A **breaking change** modifies the meaning or shape of an existing
  field or structure (e.g. changing a field's type, removing a field,
  repurposing a field's meaning). Breaking changes require a new major
  version of the specification.
- A **non-breaking change** adds something new without altering
  existing behavior (e.g. a new optional field, a new `testTypeConfig`
  test type, a new enum value that implementations can safely ignore
  if unrecognized). Non-breaking changes can go into a minor version.

See [SPEC.md's Versioning and Compatibility section](SPEC.md#versioning-and-compatibility)
for the normative rules implementations must follow.

## Pull request guidelines

- Schema changes (`schema/manifest.schema.json`,
  `schema/test.schema.json`) should be accompanied by corresponding
  updates to the example files in `examples/`.
- CI must pass before a pull request can be merged — this includes
  schema validity checks and, where example files exist, validation of
  those examples against the schemas.
- Keep pull requests focused: one proposal or fix per PR makes review
  easier.
