# Versioned References between WoT Documents

> [!NOTE]
> Proposal Status: In Progress

A Thing Model identifier alone does not tell a resolver which compatible version supplies the required definitions. The companion [specification draft](./spec.md) proposes version floors using `tm:minVersion` and a TD's `version.model`. The vocabulary and processing rules are provisional, not a W3C Recommendation.

## Summary

A reference identifies a Thing Model (TM) by its stable `id` and can state a minimum model version. A version-aware resolver selects the highest eligible version at or above that floor. Versions under one identifier are backward compatible; a breaking change requires a new TM identifier. For reproducible resolution, the consumer retains the selected version and content digest outside the document.

## Motivation

A TM reference alone does not tell a resolver which compatible revision supplies the required definitions. A TD's `version.model` records its model revision, but does not by itself identify the `type` link to which that revision applies. The resolver needs that association to avoid resolving a TD that requires model `1.2.0` against `1.1.0`.

## Goals

- Define optional `tm:minVersion` on `tm:extends` and `tm:submodel` links, ordinary `tm:ref` objects, and TD `type` links.
- Use the TD's `version.model` as an implicit floor when exactly one `type` link identifies its model.
- Require explicit per-link floors when several `type` links identify models and the TD has `version.model`; without that member, per-link floors are optional.
- Select the highest eligible SemVer version and fail if the floor cannot be satisfied.
- Keep authored version requirements portable and resolution pins under consumer control.
- Define optional version-navigation links without using them for resolution.

## Non-goals

- Defining version ranges, document-embedded resolver pins, or registry-specific identifiers.
- Versioning TD instance revisions with SemVer or extending TD-to-TD composition with model-version constraints.
- Defining a discovery protocol, registry storage model, compatibility checker, or automatic migration across breaking changes.
- Changing how projections select definitions, preserve source ownership, or verify digests.

## Design

`tm:minVersion` sets a minimum for the target TM's `version.model`. Its value is a SemVer 2.0.0 string without build metadata, not an exact-version pin or a version range. The referencing document's version and a registry's revision label do not determine the target version.

For a TD with exactly one `type` link, the effective floor is the greater of the link's explicit `tm:minVersion` and the TD's implicit `version.model` by SemVer precedence, or the only floor present. If a TD has several type links and `version.model`, each link requires `tm:minVersion`; its effective floor is the greater of those two values. A missing per-link floor produces `ambiguous-floor`. If the TD has no `version.model`, each type link uses its explicit floor when present. A link without one has no floor and does not cause an error.

Prereleases are ineligible unless the floor is a prerelease or the consumer opts in. If no eligible version satisfies the floor, resolution fails rather than selecting a lower version. The consumer can retain the selected identifier, version, content digest, and resolver origin outside the WoT document.

## Examples

The [example guide](./examples/README.md) covers model references and version navigation in JSON-LD:

- [TM extension with a floor](./examples/tm-extends.jsonld).
- [TM submodel with a floor](./examples/tm-submodel.jsonld).
- [TM definition reference with a floor](./examples/tm-ref.jsonld).
- [TD with one implicit type floor](./examples/td-single-type.jsonld).
- [TD with two explicit type floors](./examples/td-multiple-types.jsonld).
- [Deprecated major-version model with navigation links](./examples/tm-deprecated-v1.jsonld).

## Relationship to other proposals

[TD Composition and Reuse](../td-composition/spec.md) defines TD-to-TD operators. The version floors in this proposal apply to TM targets, not to TD instance revisions.

[WoT Projection Plans](../wot-projections/spec.md) defines `wotproj:sourceDigest`, which pins the exact bytes of a source. That authored pin serves a different purpose from a version floor or a consumer-owned resolution record. [Section 8 of the draft](./spec.md#8-relationship-to-other-proposals) specifies how these constraints interact, including when a source contains a nested plan or dependencies.

Registries such as the xRegistry WoT Registry and OPC UA WoT Connectivity can implement these resolution rules. Neither registry is required to implement this proposal.

## Next steps

- Review floor attribution, compatibility obligations, and resolver failure behavior with TD/TM implementers.
- Integrate the proposed term into vocabulary, JSON-LD contexts, validation schemas, and conformance tests before normative adoption.
- Exercise shared resolution cases across independent resolvers, including prereleases, missing floors, and digest failures.
