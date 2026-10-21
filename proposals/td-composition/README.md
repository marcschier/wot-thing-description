# TD Composition and Reuse

> [!NOTE]
> Proposal Status: In Progress

The companion [specification draft](./spec.md) explores TD-to-TD use of the existing
`tm:extends`, `tm:submodel`, and `tm:ref` mechanisms.

## Summary

The Thing Description specification defines extension, composition, and definition reuse
using Thing Models as their sources. This proposal explores making those same mechanisms
available between Thing Descriptions: a TD may extend another TD, compose a TD as a
submodel, or reuse a definition from another TD.

The proposal is deliberately limited to TD-to-TD relationships. Existing TM-to-TM behavior
remains unchanged, and cross-kind relationships (a TD using a TM or a TM using a TD) are
out of scope. The intent is to reuse the existing vocabulary and concepts, not to create a
new general-purpose composition language.

## Motivation

Thing Descriptions often represent deployed Things with concrete forms, security metadata,
identifiers, and other instance-level details. Reuse between TDs could reduce duplication
when a Thing is assembled from separately described parts, when a TD specializes a reusable
TD, or when multiple descriptions share one affordance definition.

Today, `tm:extends`, `tm:submodel`, and `tm:ref` are specified in the Thing Model sections
and their targets are Thing Models. This leaves authors without a standard way to express
the corresponding TD-to-TD relationships. Requiring authors to recast an already concrete
TD as a TM can lose or obscure its instance-level meaning.

## Goals

- Define TD-to-TD targets for `tm:extends`, `tm:submodel`, and `tm:ref`.
- Preserve the existing TM-to-TM semantics and conformance requirements.
- Keep the source TDs unchanged and make resolution and errors predictable.
- Define how TD identity, affordance naming, forms, security, context, and base URIs behave
  when definitions are reused or inherited.
- Keep the mechanism generic to WoT and independent of protocol-specific bindings.

## Non-goals

- Allowing TD-to-TM or TM-to-TD composition.
- Replacing Thing Models as templates or changing the existing TM-to-TM behavior.
- Defining a new document format or projection mechanism.
- Defining runtime discovery, synchronization, or propagation when a source TD changes.
- Defining a protocol-specific way to bind composed descriptions.

## Proposed scope

The proposal keeps the current distinction between the three operators:

- **`tm:extends`**: inherit a whole TD as the basis of another TD, subject to rules for
  identifying the resulting Thing and resolving conflicting definitions.
- **`tm:submodel`**: state that a TD is composed of one or more child TDs, with an optional
  `instanceName` to distinguish children. The draft examines whether this is only a
  navigable relationship or also affects the parent TD's affordance maps.
- **`tm:ref`**: reuse a definition or sub-definition from a TD, including the dependencies
  needed to interpret it.

Each operator continues to use its existing relation or term. A TD-to-TD source is identified
as a TD, for example with `application/td+json`; it is not treated as a Thing Model. Each
mechanism retains its own meaning rather than becoming interchangeable syntax.

## Design questions

TDs are concrete descriptions rather than templates, so extending them raises issues that
do not arise in the same way for TMs:

- Which TD's `id`, title, version, base, and root metadata describe the resulting TD?
- When two TDs define the same affordance name, should the result reject the conflict, let
  the extending TD override it under compatibility constraints, or offer another rule?
- How are source forms and relative references preserved when definitions move into another
  TD, and can a parent `base` change their meaning?
- How are effective security requirements and definitions kept source-owned?
- Does `tm:submodel` only link a parent and child Thing, or should it also contribute
  affordances to the parent?
- Which dependencies are copied for `tm:ref`, and how are contexts and opaque extensions
  preserved?
- Must ordinary TD consumers resolve these relationships before use, or can some
  relationships remain descriptive links?

These questions are captured in the working draft; decisions should be made with TD
implementers and existing TM processing experience.

## Relationship to other proposals

This proposal is distinct from [WoT Projection Plans](../wot-projections/README.md).
Projection Plans select pieces across sources and combine them into a task-specific view.
This proposal instead examines extensions to the existing TD/TM inheritance, submodel, and
definition-reference operators. Neither proposal depends on protocol-specific binding
semantics.

## Next steps

- Review the proposed TD-to-TD scope with TD and TM implementers.
- Decide whether each relationship is resolved into a TD or remains an ordinary link.
- Specify conflict, identity, URI/base, dependency, and security behavior.
- Add interoperable examples and processor tests before considering normative integration.
