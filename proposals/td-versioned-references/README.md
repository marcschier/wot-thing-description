# Versioned References between WoT Documents

> [!NOTE]
> Proposal Status: In Progress

The companion [specification draft](./spec.md) proposes version floors for references to Thing Models, using `tm:minVersion` and a TD's `version.model`. The vocabulary and processing rules are provisional, not a W3C Recommendation.

## Summary

A reference identifies a Thing Model (TM) by its stable `id` and may state the minimum model version it needs. A version-aware resolver selects the highest eligible version at or above that floor. Breaking changes use a new TM identifier; versions under one identifier are backward compatible. A consumer can retain the selected version and content digest outside the document for reproducibility.

## Motivation

A TM reference identifies its target without stating which compatible revision supplies the required definitions. A TD's `version.model` records its model revision, but a resolver also needs a rule associating that revision with a particular `type` link. Without a floor, a TD requiring model `1.2.0` could resolve against `1.1.0`.

## Goals

- Define optional `tm:minVersion` on `tm:extends` and `tm:submodel` links, ordinary `tm:ref` objects, and TD `type` links.
- Use the TD's `version.model` as an implicit floor when exactly one `type` link identifies its model.
- Require explicit per-link floors when several `type` links identify models and the TD has `version.model`; without that member, per-link floors are optional.
- Select the highest eligible SemVer version and fail if the floor cannot be satisfied.
- Separate portable author requirements from consumer-owned resolution pins.
- Define optional version-navigation links without using them for resolution.

## Non-goals

- Defining version ranges, document-embedded resolver pins, or registry-specific identifiers.
- Versioning TD instance revisions with SemVer or extending TD-to-TD composition with model-version constraints.
- Defining a discovery protocol, registry storage model, compatibility checker, or automatic migration across breaking changes.
- Changing projection selection, source ownership, or digest verification.

## Design

`tm:minVersion` is a SemVer 2.0.0 string without build metadata. It constrains the target TM's `version.model`, not the referencing document's version or a registry's revision label. It is a floor, not an exact version or a range.

A TD with exactly one `type` link uses the greater of its explicit `tm:minVersion` and implicit `version.model` by SemVer precedence, or the only floor present. With several type links and `version.model`, each link requires `tm:minVersion` and uses the greater of those two values; a missing per-link floor is `ambiguous-floor`. Without `version.model`, each type link uses only its own explicit floor if present, and otherwise has no floor without error.

Prereleases are ineligible unless the floor is a prerelease or the consumer opts in. Resolution fails rather than downgrading. The selected identifier, version, content digest, and resolver origin can be retained by the consumer outside the WoT document.

## Examples

The [example guide](./examples/README.md) describes six JSON-LD documents:

- [TM extension with a floor](./examples/tm-extends.jsonld).
- [TM submodel with a floor](./examples/tm-submodel.jsonld).
- [TM definition reference with a floor](./examples/tm-ref.jsonld).
- [TD with one implicit type floor](./examples/td-single-type.jsonld).
- [TD with two explicit type floors](./examples/td-multiple-types.jsonld).
- [Deprecated major-version model with navigation links](./examples/tm-deprecated-v1.jsonld).

## Relationship to other proposals

[TD Composition and Reuse](../td-composition/spec.md) defines TD-to-TD operators. This proposal constrains TM targets and does not assign SemVer compatibility to TD instance revisions.

[WoT Projection Plans](../wot-projections/spec.md) defines an exact source-bytes pin, `wotproj:sourceDigest`. That pin is distinct from a version floor and from this proposal's consumer-owned resolution record. The interaction, including nested plans and dependencies, is specified in [Section 8 of the draft](./spec.md#8-relationship-to-other-proposals).

Registries such as the xRegistry WoT Registry and OPC UA WoT Connectivity can implement these resolution rules. Neither registry is required to implement this proposal.

## Next steps

- Review floor attribution, compatibility obligations, and resolver failure behavior with TD/TM implementers.
- Integrate the proposed term into vocabulary, JSON-LD contexts, validation schemas, and conformance tests before normative adoption.
- Exercise shared resolution cases across independent resolvers, including prereleases, missing floors, and digest failures.
