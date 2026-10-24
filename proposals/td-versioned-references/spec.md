# Versioned References between WoT Documents

> **Status:** Working draft for discussion. This document is a proposal, not a W3C specification.
>
> `tm:minVersion` and the processing rules below are provisional. Normative terms state the proposed requirements; they do not imply Working Group adoption.

## Abstract

A reference to a Thing Model (TM) needs to identify the model and can state the minimum compatible version that supplies the required definitions. This document defines an optional `tm:minVersion` member as a floor on the target's `version.model`. A Thing Description (TD) with exactly one `type` link uses its own `version.model` as an implicit floor for that link. The resolver selects the highest eligible version under the target's identifier or reports failure; a breaking change requires a new identifier. The consumer or resolver retains exact resolution pins outside the authored versioned reference.

## 1. Scope and conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, NOT RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14), comprising RFC 2119 and RFC 8174, when, and only when, they appear in all capitals.

The proposed requirements apply to authors of versioned TM references and to version-aware resolvers. Ordinary processors that do not implement this proposal are outside its conformance scope. A processor claiming version-resolution support MUST either satisfy every applicable floor or report an error; it MUST NOT report unconstrained retrieval as successful version resolution.

The proposal covers `tm:extends`, `tm:submodel`, ordinary `tm:ref`, and TD links whose `rel` is `type`. It does not define TD-to-TD version compatibility, registry storage, resource discovery, or a new TD/TM media type. Existing inheritance, composition, JSON Pointer, and target-kind requirements apply after target-version selection.

## 2. Terminology

**Target identifier:** The stable `id` of the referenced TM. Versions under this identifier form a compatibility line; the identifier does not select one immutable representation.

**Candidate:** An available version of that TM, identified by its authored `version.model` and original document bytes.

**Floor:** The minimum target-model version required by a reference. Comparison uses SemVer precedence, not lexical order or publication time.

**Resolution record:** Output owned by the consumer or resolver that identifies the target, selected version, content digest, and resolver origin. Retaining the record outside the WoT document allows the consumer to pin a subsequent resolution to the same version and bytes.

A version-aware resolver MAY use a registry, a local model catalog, or another source of candidates. How it discovers candidates or maps a retrieval URI to a target identifier is outside this proposal. It MUST establish the target identity and MUST NOT combine candidates with different `id` values.

## 3. Vocabulary and JSON-LD context

The proposed term `tm:minVersion` uses the Thing Model namespace `https://www.w3.org/2019/wot/tm#`. It is a string-valued member, not an IRI-valued reference. Its value MUST be a valid [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html) version without build metadata. For example, `1.2.0` and `1.3.0-rc.1` are valid; `1.2`, `v1.2.0`, `^1.2.0`, `>=1.2.0`, and `1.2.0+build.7` are invalid. SemVer's numeric and prerelease identifier rules, including the restrictions on leading zeroes, apply.

| Location | Cardinality | Meaning |
| --- | --- | --- |
| TM link with `rel: "tm:extends"` | 0..1 | Floor for the parent TM |
| TM link with `rel: "tm:submodel"` | 0..1 | Floor for the child TM |
| Object containing ordinary `tm:ref` | 0..1 | Floor for the TM containing the referenced definition |
| TD link with `rel: "type"` | 0..1 | Explicit floor for that model link; required when several type links occur and the TD has `version.model` |

`tm:minVersion` MUST occur on the link object, or as a sibling of the `tm:ref` member in the same object. The value of `tm:ref` is a URI-reference string, not a wrapper object. The floor controls reference resolution; it MUST NOT be copied into the imported definition as an affordance annotation. This proposal does not assign meaning to `tm:minVersion` in other locations or on other link relations.

The examples extend the TD 1.1 context with an explicit prefix and string mapping:

```json
{
  "@context": [
    "https://www.w3.org/2022/wot/td/v1.1",
    {
      "tm": {
        "@id": "https://www.w3.org/2019/wot/tm#",
        "@prefix": true
      },
      "tm:minVersion": {
        "@id": "https://www.w3.org/2019/wot/tm#minVersion",
        "@type": "http://www.w3.org/2001/XMLSchema#string"
      }
    }
  ]
}
```

The context declares the term's mapping; it does not standardize the term or implement resolution. Adoption requires changes to the vocabulary, contexts, validation schemas, and processor behavior. Compact link relation spellings such as `tm:extends` follow the TM link-relation rules. A JSON-LD prefix declaration alone does not define how to expand arbitrary `rel` strings. `type`, `successor-version`, and `latest-version` are link relation names, not proposed TM terms.

## 4. Model identity and compatibility

A TM author MUST use a new `id` for a breaking change, including a change that makes an earlier version's valid use incompatible. All versions published under one `id` MUST be backward compatible with earlier versions under that identifier. The major compatibility line belongs in the identifier; an identifier such as `urn:example:pump:v1` distinguishes it from `urn:example:pump:v2`. This proposal does not prescribe an identifier syntax or require a literal `v1` suffix.

An authored SemVer major-version change MUST use a new `id`. Minor and patch changes under one `id` MUST preserve backward compatibility. A resolver MUST NOT search a successor identifier to satisfy a floor on the original identifier, even if that successor has a numerically higher version.

Candidates used for the version-resolution algorithm MUST have an authored `version.model` satisfying the syntax in Section 3. A TM without such a version cannot establish that it satisfies a floor and MUST NOT be assigned an inferred version for comparison. Registry-generated version labels are not authored model versions and MUST NOT be used as floors. These restrictions concern version resolution, not whether the document can be processed as an ordinary unversioned TM.

The resolver uses the target's `version.model` for comparison. Its own internal revision label, the referencing TM's version, and a TD's `version.instance` do not supply the target model version. A version number is an author's compatibility claim, not evidence of compatibility or document authenticity. Authors SHOULD validate compatibility before publication.

## 5. Determining the floor

### 5.1 Explicit TM-reference floors

A TM author MAY include `tm:minVersion` on a `tm:extends` or `tm:submodel` link, or beside an ordinary `tm:ref`. The floor applies to the target TM as a whole. For an external `tm:ref`, the resolver MUST select the document version before evaluating the fragment's JSON Pointer. A selected version lacking the required definition MUST fail reference resolution; the processor MUST NOT retry an older version to find the definition.

For a document-local `tm:ref`, the target is the containing TM already being processed. If `tm:minVersion` is present, the processor MUST check that TM's `version.model` against it and fail if the version is missing, invalid, or below the floor. It MUST NOT substitute another version of the containing document to satisfy a local reference.

An omitted floor imposes no minimum version. When version resolution is requested, the eligibility and highest-version rules in Section 6 still apply. Omission does not imply `0.0.0`, an exact pin, or permission to cross identifiers.

### 5.2 Exactly one TD type link

When a TD contains exactly one link with `rel: "type"` and has `version.model`, that value MUST be treated as the floor for the link. The value MUST satisfy Section 3's version syntax. It states the model version needed by the TD, not an exact revision to retrieve.

The link MAY also carry `tm:minVersion`. If both an explicit and an implicit floor apply, both MUST be satisfied: the effective floor is the greater value by SemVer precedence. An explicit lower value MUST NOT weaken the TD's implicit floor. Prerelease eligibility is determined from that effective floor.

If `version.model` is absent, the link's explicit floor applies when present. The model link MUST target a TM for this algorithm; a non-TM target causes an error rather than an unconstrained model match.

### 5.3 Several TD type links

When a TD contains several links with `rel: "type"` and has `version.model`, each link MUST carry its own `tm:minVersion`. If any link lacks that member, unpinned resolution MUST fail as `ambiguous-floor`, including when resolving a different, fully specified link. When every link has an explicit floor, the effective floor for each link MUST be the greater of its own `tm:minVersion` and the TD's `version.model` by SemVer precedence. The resolver MUST validate both values and determine prerelease eligibility from the effective floor. It MUST NOT use `version.model` in place of a missing per-link floor.

A TD without `version.model` has no implicit floor. Each type link MUST use its own `tm:minVersion` if present, and otherwise has no floor. A missing floor in this case MUST NOT produce `ambiguous-floor`; TDs with several unversioned type links can be resolved without adding floors. Each link is resolved independently against its own target TM.

If a TD has no `type` links, the resolver has no link to which it can attribute an implicit floor. This proposal does not infer a target from `version.model` alone or change ordinary TD validation for that case.

## 6. Resolution semantics

### 6.1 Selection

For each reference requiring version resolution, a resolver MUST:

1. Validate the reference and determine the target identifier and effective floor according to Sections 3–5.
2. Obtain candidates for that identifier and validate their document kind, `id`, and authored `version.model`. Invalid candidate representations MUST NOT be treated as eligible model versions.
3. Exclude prerelease candidates unless the effective floor is a prerelease or the consumer explicitly opts in to prerelease selection. A prerelease floor enables prerelease candidates for that reference; it is not an exact prerelease pin.
4. Exclude candidates below the floor, if a floor is present, using SemVer 2.0.0 precedence.
5. Select the highest eligible version by SemVer precedence. Publication order, retrieval order, and registry default pointers MUST NOT replace version precedence.
6. Complete ordinary reference processing against the selected document, including its fragment and dependencies, or report an error.

A release has higher precedence than prereleases of the same major, minor, and patch values. Numeric prerelease identifiers are compared numerically under SemVer; for example, `1.3.0-rc.10` has higher precedence than `1.3.0-rc.2`. A prerelease such as `1.2.0-rc.1` is below a release floor of `1.2.0`, even when the consumer opts in.

If no eligible candidate satisfies the floor, resolution MUST fail. A resolver MUST NOT downgrade below the floor, discard an invalid floor, substitute a different model identifier, or report a partial composition as successful. If candidates with different bytes claim the same identifier and version, the resolver MUST report an inconsistent publication rather than choose one.

An explicit resolution pin, such as a version identifier or a digest from a prior resolution record, lets the caller repeat a resolution against the selected candidate. The pin selects that candidate before the unpinned floor, prerelease, and highest-version rules apply. The resolver MUST verify the target identity, candidate version syntax, and exact bytes when a digest is supplied. A missing pinned candidate or digest mismatch MUST fail without an unpinned fallback. The authored reference and its floor do not change; the floor governs unpinned resolution.

### 6.2 Resolution records and reproducibility

A successful resolution SHOULD return a record containing the target identifier, selected `version.model`, a cryptographic digest of the selected document's exact bytes, and the resolver origin. A consumer requiring reproducibility MUST retain both the version and digest. The digest algorithm and digest MUST be identified; SHA-256 is RECOMMENDED. The resolver MUST NOT canonicalize or transform the bytes before calculating an exact-bytes digest.

A consumer MAY supply that record on a subsequent resolution. The resolver MUST retrieve the pinned version and verify its target identity, version syntax, and exact-byte digest. A missing pinned version or mismatching digest MUST fail; it MUST NOT trigger an unpinned fallback. The record repeats the caller's selection without rerunning floor or prerelease candidate selection. To select the highest eligible version, the consumer performs a new unpinned resolution and retains a new record.

The resolver's selected version and digest MUST NOT be written into the authored WoT document as a pin for these references. A resolver MUST NOT replace the floor with the selected version or rewrite the target to a registry-specific revision identifier. Keeping the authored requirement separate from the resolution result allows the document to be used across catalogs and registries.

This separation does not prohibit the distinct, explicitly authored `wotproj:sourceDigest` mechanism in the projection proposal; its interaction is specified in Section 8.2. An exact source-bytes pin does not automatically pin the source's transitive dependencies. Reproducible dependency resolution requires retaining a resolution record for every version-resolved dependency.

### 6.3 Resolution cases

These cases assume unpinned resolution of one target identifier and no operator restriction. A dash in the floor column means the reference omits a floor.

| Available authored versions | Floor | Consumer allows prereleases | Result |
| --- | --- | --- | --- |
| `1.0.0`, `1.1.0`, `1.0.1` | `1.0.0` | No | `1.1.0`, independent of publication order |
| `1.1.0` | `1.2.0` | No | Failure: unsatisfied floor |
| `1.2.0`, `1.3.0-rc.1` | `1.2.0` | No | `1.2.0` |
| `1.2.0`, `1.3.0-rc.1` | `1.2.0` | Yes | `1.3.0-rc.1` |
| `1.3.0-rc.2`, `1.3.0-rc.10`, `1.3.0` | `1.3.0-rc.1` | No | `1.3.0` |
| `1.3.0-rc.2`, `1.3.0-rc.10` | `1.3.0-rc.1` | No | `1.3.0-rc.10` |
| `1.2.0-rc.1` | `1.2.0` | Yes | Failure: prerelease is below the floor |
| `1.3.0-rc.1` | — | No | Failure: no eligible release |
| `1.2.0`, `1.3.0` | — | No | `1.3.0` |

A TD with one type link and `version.model: "1.2.0"` has the same result as an explicit floor of `1.2.0`. A TD with two type links, `version.model`, and a link without an explicit floor fails attribution before candidate selection. Without `version.model`, the same TD resolves each link without a floor.

## 7. Version-navigation links

A TD or TM MAY include links with `rel: "successor-version"` or `rel: "latest-version"` using the registered relations defined in [RFC 5829](https://www.rfc-editor.org/rfc/rfc5829.html). TD 1.1 also supports `predecessor-version`. A consumer can follow these relations to navigate a version history; they do not constrain versions or determine candidate selection.

A deprecated TM's successor link MAY identify its replacement under a new major-version identifier. A latest-version link MAY locate the latest version in the corresponding version history. Neither link establishes that its target is compatible, has the original identifier, or satisfies a floor. A resolver MUST NOT use navigation links to redirect an existing reference across compatibility lines or to override the selection rules in Section 6.

Deprecation does not itself make compatible versions ineligible. The [deprecated v1 example](./examples/tm-deprecated-v1.jsonld) states deprecation in its description and links to v2; this proposal introduces no new deprecation member or automatic migration rule.

## 8. Relationship to other proposals

### 8.1 TD Composition and Reuse

[TD Composition and Reuse](../td-composition/spec.md) proposes `td:extends`, `td:subthing`, and `td:ref` for TD-to-TD relationships. Those targets are concrete TDs, not versioned model compatibility lines. This proposal MUST NOT be interpreted as assigning `tm:minVersion` to those operators or comparing a TD's `version.instance` as a model version.

Ordinary TM composition uses `tm:extends`, `tm:submodel`, and `tm:ref` with version selection performed before the existing operator's processing. Version selection does not change inheritance, child-model semantics, overrides, context preservation, source ownership, security, or cycle handling. A version floor cannot make an otherwise invalid composition valid. When a composed TD includes type links to TMs, the type-link rules apply to those model references independently of TD-to-TD composition.

### 8.2 WoT Projection Plans

[WoT Projection Plans](../wot-projections/spec.md) defines `wotproj:sourceDigest` on a source-manifest entry as an exact-byte SHA-256 pin. The projection processor MUST verify that digest against the source bytes as retrieved, before transforming the source or resolving a nested projection plan. A mismatching or unverified digest MUST fail projection resolution; a higher model version or a satisfied floor cannot substitute for the pinned bytes.

This proposal does not extend the projection source-manifest grammar with `tm:minVersion`. Source acquisition and source-name matching follow the projection proposal. Projection selectors that use `tm:ref` in a TM envelope retain their projection-specific meaning; they do not acquire ordinary `tm:ref` floor semantics merely by sharing that member name. The projection proposal's closed selector-annotation set does not admit `tm:minVersion`. Authors MUST NOT use a selector-local floor to select a different version of a manifest source.

An ordinary `tm:ref` that is not a projection selector can carry `tm:minVersion` as specified in Section 5.1. Ordinary TM references inside a retrieved source likewise retain their floors during dependency resolution. A source's TD type-link floors are evaluated against that source's own root metadata, not against the projection envelope's `version.model`.

A source digest pins the representation retrieved from that source. If the source is an unresolved nested plan, the digest pins that plan, not its resolved result or the bytes of transitive sources or dependencies. The consumer MUST separately retain dependency resolution records or use the projection proposal's source pins at the relevant dependency boundaries when reproducibility is required.

If a resolver is given both an external minimum-version requirement and a projection source pin for the same TM source, both MUST be satisfied. It MUST locate and verify the exact pinned representation and check that representation's authored `version.model` against the floor and prerelease policy. A pinned version below the floor fails; a newer version with different bytes fails the pin. No fallback to another source version is permitted. These constraints do not introduce a manifest member.

## 9. Registry implementations (informative)

The draft xRegistry WoT Registry and OPC UA WoT Connectivity specifications specify version resolution and how a registry exposes candidates, authored versions, and consumer resolution records. This W3C proposal does not define their storage identifiers, generated revision labels, default-version policies, or information-model mappings. A conforming version-aware resolver does not depend on either registry: it uses the document's portable model identifier and authored `version.model` for comparison.

## 10. Security considerations

A floor is not an authenticity or authorization guarantee. Resolvers SHOULD authenticate candidate publishers or apply an appropriate trust policy and SHOULD use authenticated transport. A digest detects byte changes relative to a retained trusted pin; an untrusted digest does not establish provenance.

Resolvers SHOULD enforce safe-fetching policies covering URI schemes, destinations, redirects, response size, recursion depth, retrieval time, and dependency cycles. They MUST NOT copy retrieval credentials into imported interaction security or weaken the source's security during composition. Resolver origin is provenance metadata, not authority to change a document's identity or compatibility contract.

## 11. Examples and adoption work

The [example guide](./examples/README.md) lists complete illustrative JSON-LD documents for extension, submodels, definition reuse, single- and multiple-model TDs, and major-version navigation. Example target IRIs identify externally supplied models; the examples are not a candidate repository or an executable resolution service.

Integrating these requirements into the specification requires adding the proposed term to the TM vocabulary and supported JSON-LD contexts, updating link and reference validation schemas, documenting processor capabilities, and testing version selection across independent implementations. This proposal does not modify the base TD/TM context or schemas in this repository. Open issues are how to signal support when processors can ignore extension terms, how to discover candidates safely, and what evidence is needed to verify backward compatibility claims.
