# Versioned References between WoT Documents

> **Status:** Working draft for discussion. This document is a proposal, not a W3C specification.
>
> `tm:minVersion`, `td:minVersion`, and the processing rules below are provisional. Normative terms state the proposed requirements; they do not imply Working Group adoption.

## Abstract

A versioned reference to a Thing Model (TM) or Thing Description (TD) identifies its target and can state the minimum revision required. This document defines `tm:minVersion` for TM targets and `td:minVersion` for TD-to-TD composition targets. A TD uses SemVer `version.instance` as its revision identity; its `version.model` remains an implicit floor only for a `rel: "type"` TM link. A TD floor selects within the same major version; without a TD floor, selection can cross majors. The consumer or resolver retains exact resolution pins outside the authored reference.

## 1. Scope and conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, NOT RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14), comprising RFC 2119 and RFC 8174, when, and only when, they appear in all capitals.

The proposed requirements apply to authors of versioned TM and TD references, publishers that assign document versions, and version-aware resolvers. Ordinary processors that do not implement this proposal are outside its conformance scope. A processor claiming version-resolution support MUST either satisfy every applicable floor or report an error; it MUST NOT report unconstrained retrieval as successful version resolution.

The proposal covers `tm:extends`, `tm:submodel`, ordinary `tm:ref`, and TD links whose `rel` is `type`, as well as TD-to-TD `td:extends`, `td:subthing`, and ordinary `td:ref`. It does not define floors for other TD links, registry storage, resource discovery, or a new TD/TM media type. Version selection chooses the target document; existing composition, inheritance, JSON Pointer, and target-kind rules continue to govern its use.

This proposal leaves the TD 1.1 `Link` model and link serialization unchanged ([Sections 5.3.4.1](https://www.w3.org/TR/wot-thing-description11/#link) and [6.3.8](https://www.w3.org/TR/wot-thing-description11/#link-serialization-json)). The existing `VersionInfo` model and `version` serialization also remain unchanged ([Sections 5.3.1.6](https://www.w3.org/TR/wot-thing-description11/#versioninfo) and [6.3.3](https://www.w3.org/TR/wot-thing-description11/#version-serialization-json)). TM versioning, extension and import, and composition remain defined by [Sections 9.3.1](https://www.w3.org/TR/wot-thing-description11/#thing-model-versioning), [9.3.2](https://www.w3.org/TR/wot-thing-description11/#thing-model-extension-import), and [9.3.3](https://www.w3.org/TR/wot-thing-description11/#thing-model-composition). The separate [TD Composition and Reuse proposal](../td-composition/spec.md) defines the `td:extends`, `td:subthing`, and `td:ref` operators; this proposal defines version selection for their targets.

## 2. Terminology

**Target identifier:** The stable `id` of the referenced TM or TD. A TM identifier names a compatibility line; a TD identifier names a concrete Thing and remains stable across its instance revisions.

**Candidate:** An available version of the target document, identified by its authored `version.model` for a TM or `version.instance` for a TD, and its original document bytes.

**Floor:** The minimum target version required by a reference. Comparison uses SemVer precedence, not lexical order or publication time. A TD floor also limits selection to the floor major.

**Resolution record:** Output owned by the consumer or resolver that identifies the target, selected version, content digest, and resolver origin. Retaining the record outside the WoT document allows the consumer to pin a subsequent resolution to the same version and bytes.

A version-aware resolver MAY use a registry, a local model catalog, or another source of candidates. How it discovers candidates or maps a retrieval URI to a target identifier is outside this proposal. It MUST establish the target identity and MUST NOT combine candidates with different `id` values.

## 3. Vocabulary and JSON-LD context

The proposed term `tm:minVersion` uses the Thing Model namespace `https://www.w3.org/2019/wot/tm#`; `td:minVersion` uses the Thing Description namespace `https://www.w3.org/2019/wot/td#`. Both are string-valued members, not IRI-valued references, and both use the same syntax: a valid [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html) version without build metadata. For example, `1.2.0` and `1.3.0-rc.1` are valid; `1.2`, `v1.2.0`, `^1.2.0`, `>=1.2.0`, and `1.2.0+build.7` are invalid. SemVer's numeric and prerelease identifier rules, including the restrictions on leading zeroes, apply.

| Location | Cardinality | Meaning |
| --- | --- | --- |
| TM link with `rel: "tm:extends"` | 0..1 | Floor for the parent TM |
| TM link with `rel: "tm:submodel"` | 0..1 | Floor for the child TM |
| Object containing ordinary `tm:ref` | 0..1 | Floor for the TM containing the referenced definition |
| TD link with `rel: "type"` | 0..1 | Explicit TM floor; required when several type links occur and the TD has `version.model` |
| TD link with `rel: "td:extends"` or `rel: "td:subthing"` | 0..1 | Floor for the target TD |
| Object containing ordinary `td:ref` | 0..1 | Floor for the target TD containing the referenced definition |
| Projection source-manifest entry with a TM target | 0..1 | Floor for the source TM |
| Projection source-manifest entry with a TD target | 0..1 | Floor for the source TD |

`tm:minVersion` MUST occur on a TM link object, as a sibling of `tm:ref` in the same object, or on a Projection Plans source-manifest entry whose expected media type identifies a TM. `td:minVersion` MUST occur on a `td:extends` or `td:subthing` link object, as a sibling of `td:ref` in the same object, or on a source-manifest entry whose expected media type identifies a TD. The value of either `tm:ref` or `td:ref` is a URI-reference string, not a wrapper object. A floor controls target selection and MUST NOT be copied into an imported definition. A floor term used with the other target kind, or outside these locations, MUST fail as `invalid-floor`.

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
      "td": {
        "@id": "https://www.w3.org/2019/wot/td#",
        "@prefix": true
      },
      "tm:minVersion": {
        "@id": "https://www.w3.org/2019/wot/tm#minVersion",
        "@type": "http://www.w3.org/2001/XMLSchema#string"
      },
      "td:minVersion": {
        "@id": "https://www.w3.org/2019/wot/td#minVersion",
        "@type": "http://www.w3.org/2001/XMLSchema#string"
      }
    }
  ]
}
```

The context declares the term's mapping; it does not standardize the term or implement resolution. Adoption requires changes to the vocabulary, contexts, validation schemas, and processor behavior. Compact link relation spellings such as `tm:extends` follow the TM link-relation rules. A JSON-LD prefix declaration alone does not define how to expand arbitrary `rel` strings. `type`, `successor-version`, and `latest-version` are link relation names, not proposed TM terms.

## 4. Model identity and compatibility

A TM author MUST use a new `id` for a breaking change, including a change that makes an earlier version's valid use incompatible. All versions published under one TM `id` MUST be backward compatible with earlier versions under that identifier. The major compatibility line belongs in the identifier; an identifier such as `urn:example:pump:v1` distinguishes it from `urn:example:pump:v2`. This proposal does not prescribe an identifier syntax or require a literal `v1` suffix.

An authored SemVer major-version change MUST use a new `id`. Minor and patch changes under one `id` MUST preserve backward compatibility. A resolver MUST NOT search a successor identifier to satisfy a floor on the original identifier, even if that successor has a numerically higher version.

For a TD, `id` identifies a concrete Thing and does not carry a compatibility major. A breaking revision therefore retains the TD `id` and advances the major component of `version.instance`. Thing Description Resources use SemVer versioning with one version root. When `version.instance` is SemVer without build metadata, a registry MUST assign that value as `versionid` when omitted and MUST reject a supplied mismatch as `version-mismatch`. Build metadata MUST fail as `invalid-version`. If `version.instance` is absent or is not SemVer, the registry MUST assign the lowest free `0.0.0-N`; a supplied `versionid` MUST match that assignment or fail as `version-mismatch`. `versionorigin` is server-set to `authored` for a SemVer `version.instance` and `generated` for an assigned id. The registry MUST NOT rewrite `version.instance`. Generated identifiers MUST NOT be used as floors. The TD's `version.model` remains the implicit floor only for its `rel: "type"` TM link; it does not provide an implicit floor for TD-to-TD references.

Candidates used for TM version resolution MUST have an authored `version.model` satisfying the syntax in Section 3. A TM without such a version cannot establish that it satisfies a floor and MUST NOT be assigned an inferred version for comparison. Registry-generated version labels are not authored model versions and MUST NOT be used as floors. These restrictions concern version resolution, not whether the document can be processed as an ordinary unversioned TM.

A TM target is compared by its authored `version.model`; a TD target is compared by its authored `version.instance`. A TD's `version.model` supplies a floor only for its `rel: "type"` TM link. The referencing document's version and a registry's internal revision label do not supply the target version. A version number is an author's compatibility claim, not evidence of compatibility or document authenticity. Authors SHOULD validate compatibility before publication.

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

### 5.4 TD-to-TD floors

A TD author MAY include `td:minVersion` on a `td:extends` or `td:subthing` link, or beside an ordinary `td:ref`. The floor applies to the target TD as a whole. It MUST use the syntax in Section 3 and MUST NOT use the generated `0.0.0-N` form. A `td:minVersion` on a TM reference or a `tm:minVersion` on a TD-to-TD reference MUST fail as `invalid-floor`.

A TD floor is a caret floor. For a floor `X.Y.Z`, the selected version MUST be greater than or equal to `X.Y.Z` and lower than `(X+1).0.0`; the major component is incremented for all values of X, including `0`. The cap excludes prereleases whose major is `X+1`. For example, `0.4.2` admits versions from `0.4.2` up to but not including `1.0.0`. A TD reference without a floor has no implicit lower or upper bound and MAY resolve across major versions. Neither the referencing TD's `version.instance` nor its `version.model` supplies a floor for a TD target. Prerelease eligibility follows Section 6.1.

Assuming the context from Section 3, the following abbreviated JSON excerpts use `...` for unrelated members and are not standalone TD documents. A TD with one type link and `version.model: "2.3.0"` can also state a lower explicit floor; the effective floor remains `2.3.0`:

```json
{
  "version": { "model": "2.3.0" },
  "links": [
    {
      "rel": "type",
      "href": "https://example.net/models/pump.tm.jsonld",
      "tm:minVersion": "2.1.0"
    }
  ],
  ...
}
```

A TD composition link can constrain its target to the `1.x` major line:

```json
{
  "links": [
    {
      "rel": "td:extends",
      "href": "https://example.net/things/pump-base",
      "td:minVersion": "1.4.2"
    }
  ],
  ...
}
```

## 6. Resolution semantics

### 6.1 Selection

For each reference requiring version resolution, a resolver MUST:

1. Validate the reference and determine the target identifier and effective floor according to Sections 3–5.
2. Obtain candidates for that identifier and validate their document kind and `id`. Compare a TM candidate by its authored `version.model`, or a TD candidate by its authored `version.instance`. Invalid candidate representations MUST NOT be treated as eligible versions.
3. Exclude prerelease candidates unless the effective floor is a prerelease or the consumer explicitly opts in to prerelease selection. A prerelease floor enables prerelease candidates for that reference; it is not an exact prerelease pin.
4. Exclude candidates below the floor, if a floor is present, using SemVer 2.0.0 precedence. For a TD target with a floor, also exclude candidates whose major is at least `floor.major + 1`, including prereleases of the next major. A TD target without a floor has no major-version cap.
5. Select the highest eligible version by SemVer precedence. Publication order, retrieval order, and registry default pointers MUST NOT replace version precedence.
6. Complete ordinary reference processing against the selected document, including its fragment and dependencies, or report an error.

A release has higher precedence than prereleases of the same major, minor, and patch values. Numeric prerelease identifiers are compared numerically under SemVer; for example, `1.3.0-rc.10` has higher precedence than `1.3.0-rc.2`. A prerelease such as `1.2.0-rc.1` is below a release floor of `1.2.0`, even when the consumer opts in.

If no eligible candidate satisfies the floor and, for a TD target, its major-version cap, resolution MUST fail. A resolver MUST NOT downgrade below the floor, cross the cap, discard an invalid floor, substitute a different identifier, or report a partial composition as successful. If candidates with different bytes claim the same identifier and version, the resolver MUST report an inconsistent publication rather than choose one.

An explicit resolution pin, such as a version identifier or a digest from a prior resolution record, lets the caller repeat a resolution against the selected candidate. The pin selects that candidate before the unpinned floor, prerelease, and highest-version rules apply. The resolver MUST verify the target identity, candidate version syntax, and exact bytes when a digest is supplied. A missing pinned candidate or digest mismatch MUST fail without an unpinned fallback. The authored reference and its floor do not change; the floor governs unpinned resolution.

### 6.2 Resolution records and reproducibility

A successful resolution SHOULD return a record containing the target identifier, selected target version (`version.model` for a TM or `version.instance` for a TD), a cryptographic digest of the selected document's exact bytes, and the resolver origin. A consumer requiring reproducibility MUST retain both the version and digest. The digest algorithm and digest MUST be identified; SHA-256 is RECOMMENDED. The resolver MUST NOT canonicalize or transform the bytes before calculating an exact-bytes digest.

A consumer MAY supply that record on a subsequent resolution. The resolver MUST retrieve the pinned version and verify its target identity, target-kind-specific version syntax, and exact-byte digest. A missing pinned version or mismatching digest MUST fail; it MUST NOT trigger an unpinned fallback. The record repeats the caller's selection without rerunning floor or prerelease candidate selection. To select the highest eligible version, the consumer performs a new unpinned resolution and retains a new record.

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
| TD target: `1.4.0`, `1.7.0`, `2.0.0` | `1.4.0` | No | `1.7.0` |
| TD target: `1.4.0`, `2.0.0` | `1.4.0` | No | `1.4.0` |
| TD target: `1.2.0`, `2.0.0` | — | No | `2.0.0` |
| TD target: `1.2.0-rc.1`, `1.3.0-rc.1`, `2.0.0-rc.1` | `1.2.0-rc.1` | No | `1.3.0-rc.1` |

A TD with one type link and `version.model: "1.2.0"` has the same result as an explicit floor of `1.2.0`. A TD with two type links, `version.model`, and a link without an explicit floor fails attribution before candidate selection. Without `version.model`, the same TD resolves each link without a floor.

## 7. Version-navigation links

A TD or TM MAY include links with `rel: "successor-version"` or `rel: "latest-version"` using the registered relations defined in [RFC 5829](https://www.rfc-editor.org/rfc/rfc5829.html). TD 1.1 also supports `predecessor-version`. A consumer can follow these relations to navigate a version history; they do not constrain versions or determine candidate selection.

A deprecated TM's successor link MAY identify its replacement under a new major-version identifier. A latest-version link MAY locate the latest version in the corresponding version history. Neither link establishes that its target is compatible, has the original identifier, or satisfies a floor. A resolver MUST NOT use navigation links to redirect an existing reference across compatibility lines or to override the selection rules in Section 6.

Deprecation does not itself make compatible versions ineligible. The [deprecated v1 example](./examples/tm-deprecated-v1.jsonld) states deprecation in its description and links to v2; this proposal introduces no new deprecation member or automatic migration rule.

## 8. Relationship to other proposals

### 8.1 TD Composition and Reuse

[TD Composition and Reuse](../td-composition/spec.md) Sections 5–7 define `td:extends`, `td:subthing`, and `td:ref` and their composition behavior. This proposal defines target-version selection for those operators in Sections 5.4 and 6.1. They use `td:minVersion`, not `tm:minVersion`; a floored target is bounded to its major line, while an unfloored reference can cross majors. The target's `version.instance` identifies its revision, and the referencing TD's version members do not supply an implicit floor. Version selection does not change the operator's composition semantics.

Ordinary TM composition uses `tm:extends`, `tm:submodel`, and `tm:ref` with version selection performed before the existing operator's processing. Version selection does not change inheritance, child-model semantics, overrides, context preservation, source ownership, security, or cycle handling. A version floor cannot make an otherwise invalid composition valid. When a composed TD includes type links to TMs, the type-link rules apply to those model references independently of TD-to-TD composition.

### 8.2 WoT Projection Plans

[WoT Projection Plans](../wot-projections/spec.md) defines `wotproj:sourceDigest` on a source-manifest entry as an exact-byte SHA-256 pin. The projection processor MUST verify that digest against the source bytes as retrieved, before transforming the source or resolving a nested projection plan. A mismatching or unverified digest MUST fail projection resolution; a higher model version or a satisfied floor cannot substitute for the pinned bytes.

A projection source-manifest entry MAY carry `tm:minVersion` for a TM source or `td:minVersion` for a TD source, as specified in the [WoT Projection Plans proposal](../wot-projections/spec.md#4-source-manifest). The entry's expected media type determines its target kind. The floor uses the same target-kind rules as an ordinary reference: a TM floor has no upper bound, and a TD floor is capped at the next major. Prerelease eligibility follows Section 6.1. Without a floor, a TD source can resolve across major versions. The projection envelope's `version.model` and `version.instance` do not supply an implicit source floor.

These terms apply to manifest-source resolution, not to selector annotations. Projection selectors that use `tm:ref` in a TM envelope retain their projection-specific meaning; they do not acquire ordinary `tm:ref` floor semantics merely by sharing that member name. The projection proposal's closed selector-annotation set does not admit either floor term. Authors MUST NOT use a selector-local floor to select a different version of a manifest source.

An ordinary `tm:ref` that is not a projection selector can carry `tm:minVersion` as specified in Section 5.1. Ordinary TM references inside a retrieved source likewise retain their floors during dependency resolution. A source's TD type-link floors are evaluated against that source's own root metadata, not against the projection envelope's `version.model`.

A source digest pins the representation retrieved from that source. If the source is an unresolved nested plan, the digest pins that plan, not its resolved result or the bytes of transitive sources or dependencies. The consumer MUST separately retain dependency resolution records or use the projection proposal's source pins at the relevant dependency boundaries when reproducibility is required.

If a resolver is given both a projection source floor and `wotproj:sourceDigest` for the same source, both MUST be satisfied. It MUST verify the exact pinned representation and check its authored version against the floor: `version.model` for a TM or `version.instance` for a TD, with the TD caret cap and prerelease rules. A pinned version below the floor or outside the TD major range fails; different bytes fail the digest pin. No fallback to another source version is permitted.

## 9. Registry implementations (informative)

The draft xRegistry WoT Registry and OPC UA WoT Connectivity specifications specify version resolution and how a registry exposes candidates, authored versions, generated revision labels, and consumer resolution records. For TDs, `version.instance` supplies an authored SemVer `versionid`; generated ids use `0.0.0-N`, and a TD floor limits selection to its major line. This proposal does not define registry storage or information-model mappings. A version-aware resolver uses the target TD's `id` and `version.instance`, or the target TM's `id` and `version.model`, for comparison.

## 10. Security considerations

A floor is not an authenticity or authorization guarantee. Resolvers SHOULD authenticate candidate publishers or apply an appropriate trust policy and SHOULD use authenticated transport. A digest detects byte changes relative to a retained trusted pin; an untrusted digest does not establish provenance.

Resolvers SHOULD enforce safe-fetching policies covering URI schemes, destinations, redirects, response size, recursion depth, retrieval time, and dependency cycles. They MUST NOT copy retrieval credentials into imported interaction security or weaken the source's security during composition. Resolver origin is provenance metadata, not authority to change a document's identity or compatibility contract.

## 11. Examples and adoption work

The [example guide](./examples/README.md) lists illustrative JSON-LD documents for TM floors, TD-to-TD composition floors, TD instance versions, and model navigation. Example target IRIs identify externally supplied documents; the examples are not a candidate repository or an executable resolution service.

Integrating these requirements into the specification requires adding both floor terms to their vocabularies and supported JSON-LD contexts, updating link and reference validation schemas, documenting processor capabilities, and testing TM and TD version selection across independent implementations. This proposal does not modify the base TD/TM context or schemas in this repository. Open issues are how to signal support when processors can ignore extension terms, how to discover candidates safely, and what evidence is needed to verify backward compatibility claims.
