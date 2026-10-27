# Versioned References between WoT Documents

> [!NOTE]
> Proposal Status: In Progress

A WoT document identifier alone does not tell a resolver which revision supplies a referenced definition or whether a captured dependency is compatible. The companion [specification draft](./spec.md) separates SemVer selection, incremental compatibility lines, exact replay and publication-set acceptance. The vocabulary and processing rules are proposals, not a W3C Recommendation.

## Summary

A reference identifies a TM or TD by its stable `id`. The provider declares whether its revisions use SemVer or an incremental sequence; a resolver never guesses from document content. SemVer retains uncapped TM floors, next-major-bounded TD floors and TD type-link implicit model floors. Incremental references select an explicit compatibility line and can require a minimum integer revision. Without a sticky default, live resolution selects the highest qualifying revision. A sticky choice that violates the requirement fails rather than being bypassed.

A snapshot fixes a complete publication set and exact dependency bindings. A channel selects a snapshot, not a compatibility class. Snapshot-bound composition checks each bound target against the authored requirement; a raw consumer pin instead retrieves an exact revision without rerunning live selection. These are different operations.

## Motivation

SemVer fits authored compatibility versions but does not make an unversioned document or a free-form external version label into an incremental revision. A provider can publish unchanged bytes under an integer revision and retain an authorized compatibility-line declaration alongside them. The integer establishes order within one source sequence; the line supplies the compatibility claim.

Publishing several related documents introduces a separate problem: exact entry bytes do not fix their transitive dependencies. A publication needs bindings for each recognized external reference occurrence, together with original bases, contexts and trusted metadata. Merely pinning a parent or naming a channel does not establish that closure.

## Goals

- Define optional `tm:minVersion` on TM references and TD `rel:type` links, and `td:minVersion` on TD-to-TD composition references.
- Preserve implicit and explicit SemVer type-link floors, applying them only to the SemVer-target subset of mixed-profile links.
- Define incremental line selection and optional numeric floors without inventing core WoT term names.
- Honour sticky defaults during live selection and fail unsatisfied requirements without fallback.
- Keep authored version requirements portable and resolution pins under consumer control.
- Define WoT closure and compatibility acceptance independently of the mechanism publishing snapshots and channels.
- Define optional version-navigation links without using them for resolution.

## Non-goals

- Defining version ranges, document-embedded resolver pins, or registry-specific identifiers.
- Defining floors for other TD links or changing TD-to-TD composition semantics.
- Defining a discovery protocol, registry allocation/storage model, publication API or manifest format, compatibility checker, or automatic migration across breaking changes.
- Changing how projections select definitions, preserve source ownership, or verify digests.

## Design

### SemVer references

`tm:minVersion` sets a minimum for the target TM's `version.model`. `td:minVersion` sets a minimum for the target TD's `version.instance` and caps selection below the next major, including when that major is zero. Both values use SemVer 2.0.0 without build metadata. An unfloored TD reference can select across majors. A referencing TD's versions do not determine the target TD version.

For a TD with exactly one SemVer `type` target, the effective floor is the greater of the link's explicit floor and the TD's implicit `version.model`, or the only floor present. With several SemVer targets and `version.model`, each SemVer link requires an explicit floor; a missing one produces `ambiguous-floor`. Without `version.model`, those per-link floors are optional. Incremental type targets never inherit a model floor.

For non-sticky selection, prereleases are ineligible unless the effective floor is a prerelease or the consumer opts in. A sticky default bypasses prerelease filtering but must satisfy the floor and TD cap. Generated `0.0.0-N` bookkeeping identifiers are not incremental revisions and cannot be floors.

### Incremental references

An incremental reference explicitly selects an absolute compatibility-line URI and can add a canonical positive decimal floor. With history 42/A, 43/B and 44/A, a line-A reference selects 44 unless a sticky default intervenes; a sticky 43/B makes it fail. A floor without a line fails. Independent providers' revision numbers are not interchangeable.

The [deployment mapping](./spec.md#31-incremental-requirements-and-deployment-mapping) illustrates xRegistry's proposed `uav:minVersion` and `uav:compatibilityLine` terms in the existing OPC UA WoT Binding namespace. It changes no composition operator. Core WoT names remain open rather than treating that external vocabulary as adopted W3C language.

### Publication and replay

The [publication contract](./spec.md#9-publication-and-registry-mappings) requires complete reference-occurrence bindings and compatibility checks over one selected snapshot. Missing or incompatible content fails without live resolution or an older-head search. Offline replay also needs trusted metadata and context material, not only document bytes.

A consumer retains exact revision, digest, origin and applicable profile/line expectations outside the document. Raw exact replay bypasses live candidate selection but verifies those recorded expectations. It does not imply compatibility-checked composition or pin the transitive closure.

## Examples

The [example guide](./examples/README.md) covers model references and version navigation in JSON-LD:

- [TM extension with a floor](./examples/tm-extends.jsonld).
- [TM submodel with a floor](./examples/tm-submodel.jsonld).
- [TM definition reference with a floor](./examples/tm-ref.jsonld).
- [TD with one implicit type floor](./examples/td-single-type.jsonld).
- [TD with two explicit type floors](./examples/td-multiple-types.jsonld).
- [Deprecated major-version model with navigation links](./examples/tm-deprecated-v1.jsonld).
- [TD composition floors and TD instance versions](./examples/td-extends.jsonld), [td-subthing.jsonld](./examples/td-subthing.jsonld), [td-ref.jsonld](./examples/td-ref.jsonld), [td-instance-v1.jsonld](./examples/td-instance-v1.jsonld), and [td-instance-v2.jsonld](./examples/td-instance-v2.jsonld).
- [Incremental TD composition](./examples/td-incremental-extends.jsonld) and [incremental TM definition reference](./examples/tm-incremental-ref.jsonld), using the informative deployment mapping.

## Relationship to other proposals

[TD Composition and Reuse](../td-composition/spec.md) defines TD-to-TD operators. This proposal defines how their target TD version is selected; selection does not change composition semantics.

[WoT Projection Plans](../wot-projections/spec.md) defines `wotproj:sourceDigest`, which pins the exact bytes of a source. That authored pin serves a different purpose from a version floor or a consumer-owned resolution record. [Section 8 of the draft](./spec.md#8-relationship-to-other-proposals) specifies how these constraints interact, including when a source contains a nested plan or dependencies.

The [source mapping](./spec.md#94-xregistry-mapping-informative) records the reviewed head of [xregistry/spec#571](https://github.com/xregistry/spec/pull/571), distinguishes its SemVer specification from the proposed incremental and closure designs, and links the generic publication and federation contracts. xRegistry-specific epochs, XIDs, manifest formats and revision allocation remain outside WoT language requirements.

## Next steps

- Review floor attribution, compatibility obligations, and resolver failure behavior with TD/TM implementers.
- Settle the core incremental vocabulary and integrate the proposed terms into contexts, validation schemas, and conformance tests before normative adoption.
- Exercise shared resolution cases across independent resolvers, including prereleases, missing floors, and digest failures.
