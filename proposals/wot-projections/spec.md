# WoT Projection Plans

> **Status:** Working draft for discussion. This document is a proposal, not a W3C
> specification.
>
> The vocabulary namespace and exact term names below are provisional. Normative terms
> (MUST, MUST NOT, SHOULD, and MAY) are used to make the proposed processing contract
> concrete; they do not imply Working Group adoption.

## Abstract

This document proposes a way to describe a projection plan using an ordinary WoT Thing
Description (TD) or Thing Model (TM) envelope. The plan is marked as unresolved, identifies
source TDs or TMs, and selects source affordances using references in the envelope's
affordance maps. A projection-aware processor resolves the plan into an ordinary TD or TM
by combining imported affordances with complete local definitions and permitted annotations.

The proposal does not define a separate document format or media type. It does not define
protocol bindings or materialization into a platform-specific information model.

## 1. Introduction

TDs describe Things and TMs describe reusable models. Applications that need a focused view
across multiple documents otherwise maintain a separate TD or TM whose definitions may drift
from its sources.

A projection plan is a TD or TM envelope marked to indicate that it is not yet an ordinary,
usable TD or TM. It describes selections by reference, optionally contributes annotations or
local affordance definitions, and can be resolved by a projection-aware processor. Resolution
produces a complete TD or TM that can be processed by existing WoT implementations.

A projection plan is distinct from TM inheritance and reuse mechanisms such as `tm:extends`,
`tm:submodel`, and ordinary `tm:ref`. It selects definitions across source documents to
create a task-specific resolved envelope; it does not change the meaning of those existing
mechanisms.

### 1.1 Conformance

This proposal describes requirements for projection-plan authors, ordinary TD/TM admission
processors, and projection-aware processors. An ordinary TD/TM processor MUST reject an
unresolved projection marker unless it invokes projection-aware processing.

A projection-aware processor MUST either return a fully resolved and validated TD or TM or
report a resolution error. It MUST NOT expose an unresolved plan as an ordinary TD or TM,
and MUST NOT return a success-shaped partial result.

### 1.2 Provisional vocabulary

The examples use `wotproj:` for a proposed WoT projection vocabulary. Its namespace IRI,
term names, and validation format have not been assigned. Example IRIs use
`https://example.org/` and are placeholders, not stable identifiers.

## 2. Terminology

**Projection plan:** A TD or TM envelope whose root type includes the projection marker.

**Source:** A TD, TM, or projection plan listed in a projection plan's source manifest.

**Selection:** An affordance reference or bulk rule that identifies affordances to include in
the resolved view.

**Imported affordance:** A selected source affordance included in the result.

**Local affordance:** A complete affordance definition authored directly in the projection
plan rather than selected from a source.

**Resolved view:** The ordinary TD or TM produced by resolving a projection plan.

**Routing:** The choice of whether an imported affordance's forms address its source endpoint
or the projection endpoint.

## 3. Projection envelope

A projection plan MUST use an ordinary TD or TM envelope and MUST include
`wotproj:Projection` in the root `@type`. It MUST also include exactly one ordinary root kind:
`Thing` or `tm:ThingModel`. That ordinary root kind determines the result kind. A separate
projection-kind member MUST NOT be used.

The envelope MUST include:

- an absolute `id`;
- an absolute `wotproj:scenario` IRI identifying the purpose of the view;
- a non-empty ordered `wotproj:projects` source manifest; and
- at least one imported selection or local affordance.

The plan uses the ordinary media type for its envelope kind: `application/td+json` for a TD
or `application/tm+json` for a TM. There is no projection-specific media type or stored
document kind. The projection marker distinguishes an unresolved plan from an ordinary TD
or TM after retrieval.

The example vocabulary and syntax below are provisional:

```json
{
  "@context": [
    "https://www.w3.org/2022/wot/td/v1.1",
    {
      "tm": {
        "@id": "https://www.w3.org/2019/wot/tm#",
        "@prefix": true
      },
      "wotproj": "https://example.org/wot/projection#"
    }
  ],
  "@type": ["Thing", "wotproj:Projection"],
  "id": "urn:example:building:maintenance-view",
  "title": "Pump condition view",
  "wotproj:scenario": "https://example.org/scenarios/predictive-maintenance",
  "wotproj:projects": [
    {
      "wotproj:sourceName": "pump",
      "href": "./pump.td.json",
      "type": "application/td+json",
      "wotproj:routing": "source",
      "wotproj:select": [
        { "wotproj:affordanceKind": "property" }
      ]
    }
  ],
  "properties": {
    "pumpSpeed": {
      "tm:ref": "./pump.td.json#/properties/pumpSpeed",
      "title": "Pump speed for condition monitoring",
      "wotproj:semanticId": "https://example.org/ontology/ConditionSignal"
    },
    "viewStatus": {
      "type": "string",
      "readOnly": true,
      "forms": [
        {
          "href": "/status",
          "op": "readproperty",
          "contentType": "application/json"
        }
      ]
    }
  }
}
```

`pumpSpeed` is an imported selection because its `tm:ref` identifies a declared source and
points to an affordance in the matching map. `viewStatus` is a local definition because it
does not use a projection-selection reference; it must satisfy ordinary TD requirements.

An unresolved plan is not yet an executable TD/TM. Projection-aware admission MUST validate
the envelope and projection grammar. Local affordances MUST satisfy the ordinary definition
requirements for the envelope kind. Imported selectors are validated as references and
annotations and are completed during resolution. The processor removes the projection marker,
source manifest, and unresolved selectors, retains the plan's identifier and scenario, and
validates the resulting ordinary TD or TM before returning it.

## 4. Source manifest

`wotproj:projects` is a non-empty ordered array. Every source entry MUST contain:

- `wotproj:sourceName`: a non-empty alias unique in the manifest;
- `href`: a URI reference to the source document; and
- `type`: the expected TD/TM media type, such as `application/td+json` or
  `application/tm+json`.

The order of entries is significant and MUST be preserved by a processor. The source `href`
is resolved against the plan's effective base URI. A processor MUST verify the retrieved
document's actual kind and MUST NOT infer its kind only from a URI suffix.

A source `href` MUST NOT contain a fragment, and no two manifest entries may resolve to the
same absolute URI. To identify a source in a projection-selection `tm:ref`, a processor
resolves the reference against the plan's effective base URI, separates its fragment, and
matches the remaining absolute URI to exactly one resolved source `href`. The fragment is a
URI-encoded RFC 6901 JSON Pointer. This matching rule makes the source manifest authoritative
and prevents an undeclared document from being used as a projection source.

A source entry MAY also contain:

- `wotproj:routing`: `source` or `projection`; `source` is the default;
- `wotproj:sourceDigest`: an optional digest pin in the form
  `sha-256:<64 lowercase hexadecimal digits>`;
- `wotproj:namePrefix`: an optional prefix applied to affordance names selected in bulk;
- `wotproj:selectAll`: a boolean indicating selection of every affordance in the source; and
- `wotproj:select`: an array of bounded selection predicates.

A source MAY itself be a projection plan. It is resolved before its affordances are selected.
The graph of projection plans MUST be acyclic.

When `wotproj:sourceDigest` is present, the processor MUST compute SHA-256 over the exact
source bytes as retrieved and compare it to the declared value. The hexadecimal digest MUST
be lowercase. The processor MUST NOT canonicalize or otherwise transform the bytes before
hashing. A mismatch MUST fail resolution and identify the source. If the source is provided
only as an already-parsed object and its original bytes are unavailable, the processor MUST
report the pin as unverified and MUST NOT treat it as satisfied.

## 5. Selection

An affordance-map member in `properties`, `actions`, or `events` is a projection selection
when its `tm:ref` fragment is a JSON Pointer to a member of a source affordance map. The
target map MUST match the plan member's category, and the target affordance MUST exist.
The source MUST be declared in `wotproj:projects` and MUST match exactly one source as
specified in Section 4; an undeclared source, wrong-category target, or missing target is
an error. The member key in the plan is the selected affordance's name in the resolved view
and need not match the source name.

A `tm:ref` whose fragment does not point into a source affordance map is not a projection
selector. If it is otherwise valid, it retains its ordinary TD/TM reference semantics. A
member without a projection-selection `tm:ref` is a local affordance definition.

### 5.1 Explicit selection and annotations

An explicit selection names one affordance. Its member key names the affordance in the
resolved view; its `tm:ref` identifies the source definition. It MAY add only the following
annotation members:

- `title` and `titles`;
- `description` and `descriptions`;
- additional `@type` values;
- `wotproj:semanticId` and `wotproj:metadata`; and
- `forms` and `security` when the source is routed to the projection endpoint, as described
  in Section 8.

Annotations MUST NOT restate or change a source affordance's data schema. Additional
`@type` values MUST NOT remove source types or change the affordance's ordinary WoT kind.
Annotation IRIs are interpreted in the plan's effective context. Source-owned definition
members retain their source's context.

The selector's `tm:ref` is consumed during resolution. Any `tm:ref` or other defined
reference already present in the source affordance is retained and relocated according to
Section 7.

### 5.2 Whole-source selection

`wotproj:selectAll: true` selects every affordance in every category from that source.
Selecting a source with no affordances in a category contributes none from that category.
A source MAY provide `wotproj:namePrefix` to distinguish its bulk-selected names. The
resolved name is the prefix followed by the source name with its first Unicode code point
upper-cased. For example, source name `serialNumber` with prefix `device` becomes
`deviceSerialNumber`.

### 5.3 Predicate selection

`wotproj:select` is an array of filter objects. A filter object MAY contain only:

- `wotproj:affordanceKind`: `property`, `action`, or `event`;
- `wotproj:semanticId`: an IRI to match against the affordance's semantic identifier; and
- `@type`: one type IRI or an array of type IRIs.

The constraints in one filter object are conjunctive. Multiple filter objects are
disjunctive. When `@type` is an array, an affordance matches only if it has every listed
type. A predicate selects all matching affordances in the source. Predicates MUST NOT
contain other members, executable expressions, wildcard syntax, or implementation-defined
operators.

Predicate matching MUST compare semantic identifiers and types by expanded JSON-LD meaning,
not by compact spelling.

### 5.4 Local affordances

A member of an affordance map without a projection-selection `tm:ref` is a local affordance.
It is a complete definition validated under ordinary TD/TM rules, not an annotation. Its
schema, forms, security, context, and extension terms are owned by the plan.

If a local affordance and one or more imported selections have the same name in the same
category, the local definition replaces the imported selection in the resolved view.
Processors SHOULD report imported selections displaced by local definitions. Local
precedence is independent of manifest order.

## 6. Deterministic conflict resolution

The source manifest order is the first selection-order key. Within a source, explicit
selections precede bulk selections (whole-source and predicate selections). Within each
group, categories are ordered `properties`, `actions`, `events`. Within a category,
selections are ordered by Unicode code point using the name they will have in the resolved
view; ties are ordered by the source affordance name.

The first imported selection of a given view name and category is retained. Later imported
selections of that same name and category are dropped; processors SHOULD report the
displacement. After imported selections are resolved, local affordances with colliding names
replace the imported selection. A local definition never replaces another local definition:
duplicate local names are invalid under ordinary TD/TM rules.

These ordering rules MUST NOT depend on JSON object member order, retrieval completion order,
or processor scheduling. A conforming processor MUST produce equivalent affordance
membership for equivalent plans and source representations.

## 7. Resolution and dependency closure

Resolution produces an ordinary TD or TM whose kind is determined solely by the projection
envelope's ordinary root kind. A source may be a TD, TM, or another projection plan; each
copied affordance and dependency MUST be representable in the resolved envelope, or
resolution fails during final validation.

Resolution MUST:

1. Validate the marked envelope and projection grammar.
2. Resolve and validate every source and verify any declared source digest.
3. Recursively resolve source projection plans while detecting cycles.
4. Identify explicit, bulk, and predicate selections.
5. Apply deterministic imported-selection precedence and local-definition precedence.
6. Copy each selected definition with permitted annotations.
7. Close known local dependencies of selected definitions.
8. Relocate defined references, preserve contexts and security, and record provenance.
9. Remove projection-only members and the projection marker.
10. Validate the completed ordinary TD or TM.

Resolution is atomic. Failure at any step MUST produce an error and no partial resolved
document.

### 7.1 Dependencies and references

A processor MUST carry the known document-local dependencies required to interpret selected
affordances. These include referenced reusable schemas or data type definitions, required
URI-variable declarations, security definitions, and other TD/TM-defined dependencies.
Carried support definitions MUST NOT replace selected or owner definitions. A dependency
reference MUST target the definition copied for its source, never a same-named definition
owned by another source.

The processor MUST distinguish definitions by expanded identity and context-resolved facts,
not by lexical prefixes alone. It MUST preserve literal values and opaque metadata; dependency
discovery and reference relocation MUST NOT search or rewrite values such as `const`,
`default`, `enum`, or `examples`, or opaque extension payloads.

The exact closure of WoT-defined dependencies and handling of extension-defined references
requires further specification. An implementation MUST NOT rewrite unknown extension values
as though they were URI references.

### 7.2 Provenance

Every imported affordance in the resolved view MUST retain provenance identifying its source
document and original affordance definition pointer. The proposed member
`wotproj:resolvedFrom` carries a URI reference to that original definition. A relative
provenance reference MUST be resolved against the original source document URI before the
definition is moved.

An affordance already carrying provenance retains that original provenance through
subsequent projection resolution. Local affordances MUST NOT receive synthetic provenance.

### 7.3 URI references and forms

References whose meaning depends on a containing document's location MUST be resolved
against their original owner and location before the referenced content is moved. References
to definitions copied into the resolved document MUST target the mapped output location.
Only defined reference-bearing terms and terms explicitly defined as IRI-valued may be
relocated.

With `source` routing, each carried form's relative `href` MUST be resolved against the
source's effective `base`, or the source document URI if no `base` is declared. The resolved
view MUST carry an absolute `href` for that form, and MUST NOT declare a root `base` that
changes the carried form's meaning.

With `projection` routing, the selected affordance's forms are served by the projection
endpoint. An explicitly selected affordance MUST provide forms; for bulk and predicate
selections, the projection host MUST supply forms while resolving the view. Such forms are
resolved against the plan's effective base.

### 7.4 Context preservation

The resolved root retains the plan's effective JSON-LD context. Each imported subtree
retains the effective ordered, local, and term-scoped context of its original owner. Source
data and security definitions retain source contexts. Plan-owned forms and their dependencies
retain the plan's context.

Compact prefix spelling alone does not establish namespace identity. Selection predicates
and reference comparisons MUST use expanded semantic identities. A processor MAY serialize
scoped context differences or equivalent deterministic namespace-qualified names, provided
that it preserves opaque lexical values and the original context interpretation.

## 8. Routing and security

`wotproj:routing` is declared per source and has the value `source` or `projection`. The
default is `source`.

Under `source` routing, an imported affordance uses the source's endpoint and effective
security. A source-routed selection MUST NOT carry its own `forms` or `security` annotation.
If it does, the plan is invalid; a processor MUST report the error rather than ignore those
members. Complete local affordances are not subject to this restriction.

Under `projection` routing, the projection host serves the imported affordance. An explicit
selection MUST provide its forms and security. For bulk or predicate selection, the host
supplies forms and security during resolution. Projection-owned forms and their references
use the plan's context and base.

A source form MUST NOT inherit credentials or security defaults from the plan. The processor
MUST preserve the form's effective security according to the applicable TD/TM rules,
including any inherited source-level security. It MUST NOT invent or weaken a security
policy; a missing, cyclic, or contradictory security dependency MUST fail resolution.

Source-owned security definitions are qualified in the result as
`q:s:<encoded-source-name>:<encoded-scheme-name>`. Plan-owned security definitions use
`q:p:<encoded-scheme-name>`. Encoding operates on exact UTF-8 bytes, leaves only RFC 3986
unreserved characters unchanged, and percent-encodes all other bytes with uppercase
hexadecimal digits without Unicode normalization. All references, including nested security
combinations and issued-token references defined by WoT vocabularies, MUST be rewritten
consistently. Identical repeated references reuse one definition; contradictory definitions
MUST fail resolution.

Retrieval authorization for the plan and sources is distinct from the security needed to
interact with affordances. Retrieval credentials MUST NOT be copied into the resolved
document.

## 9. Source changes and security considerations

A projection may change its resolved result when a source changes. A source digest can pin a
selection to exact source bytes, as described in Section 4. This proposal does not define
notifications or continuous synchronization.

Projection-aware processors may retrieve multiple documents using author-supplied URIs.
Implementations SHOULD apply safe-fetching policies for allowed network destinations,
redirects, response size, retrieval time, and recursive resolution. The detailed policy and
limits require review with WoT processing guidance.

Resolved views may reveal a combined set of affordances from multiple documents. Authors
and processors should consider whether this combined view discloses information to consumers
who would not otherwise receive the individual sources.

## 10. Example

The example in Section 3 selects properties of a pump by predicate and adds a title and
semantic identifier to the explicitly selected `pumpSpeed` property. Its `viewStatus`
property is local to the projection plan. Resolution imports selected source definitions,
applies the permitted annotations, preserves each source's forms and security under
source routing, retains provenance for imported properties, and combines them with the
complete local property definition.

The result is an ordinary TD, because the plan's root type includes `Thing`. It has no
projection marker, source manifest, or unresolved selectors. The source documents are
unchanged.

## 11. Open issues

1. Assign the projection vocabulary namespace and finalize its term names and JSON-LD context.
2. Define validation for marked TD/TM envelopes and selectors that are not yet complete
   affordance definitions.
3. Complete the list of known document-local dependencies and extension reference rules.
4. Define how a projection host supplies forms and security for projection-routed bulk and
   predicate selections, and review whether both routing modes preserve TD/TM security
   semantics.
5. Define safe-fetching requirements and limits for recursive plan resolution.
6. Determine what implementation and interoperability evidence is needed before adoption.
