# Versioned Reference Examples

These JSON-LD documents illustrate version floors on TM references, TD type links, and TD-to-TD composition references, plus SemVer TD instance identity. They use the TD 1.1 context and explicitly declare the provisional `tm:minVersion` or `td:minVersion` string term in its owning namespace. Every file is parseable JSON, but resolving the references requires the vocabulary and processing changes described in the [draft](../spec.md).

| File | Contract |
| --- | --- |
| [tm-extends.jsonld](./tm-extends.jsonld) | Extend pump model v1 at `1.2.0` or a higher eligible compatible version. |
| [tm-submodel.jsonld](./tm-submodel.jsonld) | Use temperature model v2 as a submodel at `2.1.0` or higher. |
| [tm-ref.jsonld](./tm-ref.jsonld) | Select a version of the definitions TM at or above `1.1.0`, then resolve `/properties/status`. |
| [td-single-type.jsonld](./td-single-type.jsonld) | Use `version.model` as the implicit floor `1.2.0` for the single type link. |
| [td-multiple-types.jsonld](./td-multiple-types.jsonld) | Resolve each type link using the greater of its explicit floor and the root model version. |
| [tm-deprecated-v1.jsonld](./tm-deprecated-v1.jsonld) | Mark pump v1 as deprecated and link to v2 without redirecting existing references. |
| [td-extends.jsonld](./td-extends.jsonld) | Extend a target TD at floor `1.4.0`; selection stays below `2.0.0`. |
| [td-subthing.jsonld](./td-subthing.jsonld) | Select a child TD at floor `0.8.0`; selection stays below `1.0.0`. |
| [td-ref.jsonld](./td-ref.jsonld) | Select a TD definition source using sibling `td:minVersion`. |
| [td-instance-v1.jsonld](./td-instance-v1.jsonld) and [td-instance-v2.jsonld](./td-instance-v2.jsonld) | Publish two SemVer instance revisions under one concrete Thing `id`. |

Model identifiers and reference targets are illustrative HTTPS IRIs. The target models and their candidate versions are external to this folder, so parsing the JSON does not establish that the references can be resolved. The TD examples use illustrative endpoint forms and `nosec` security; these are not production security recommendations. The TM deprecation example states deprecation in descriptive metadata and does not introduce a vocabulary term.

To check JSON syntax from the repository root:

```sh
python -c "import json; from pathlib import Path; files = sorted(Path('proposals/td-versioned-references/examples').glob('*.jsonld')); [json.loads(p.read_text(encoding='utf-8')) for p in files]; print(f'{len(files)} JSON-LD examples parsed')"
```

The [multiple-type TD without a model version](./td-multiple-types-no-model-version.jsonld) has an explicit floor on one link and no floor on the other. Each link resolves independently. With no root model version, the absent per-link floor does not cause an error.

TD-to-TD floors use `td:minVersion` on `td:extends` and `td:subthing` links, and beside `td:ref`. A floor caps selection at the next major; omitting it permits selection across majors. The TD instance examples use the same `id` for `version.instance` values `1.4.0` and `2.0.0`, because the identifier names the concrete Thing rather than a compatibility major.
