# Versioned Reference Examples

These JSON-LD documents illustrate version floors on TM references and TD type links, and navigation between model versions. They use the TD 1.1 context and explicitly declare the provisional `tm:minVersion` string term in the TM namespace. Every file is parseable JSON, but resolving the references requires the vocabulary and processing changes described in the [draft](../spec.md).

| File | Contract |
| --- | --- |
| [tm-extends.jsonld](./tm-extends.jsonld) | Extend pump model v1 at `1.2.0` or a higher eligible compatible version. |
| [tm-submodel.jsonld](./tm-submodel.jsonld) | Use temperature model v2 as a submodel at `2.1.0` or higher. |
| [tm-ref.jsonld](./tm-ref.jsonld) | Select a version of the definitions TM at or above `1.1.0`, then resolve `/properties/status`. |
| [td-single-type.jsonld](./td-single-type.jsonld) | Use `version.model` as the implicit floor `1.2.0` for the single type link. |
| [td-multiple-types.jsonld](./td-multiple-types.jsonld) | Resolve each type link using the greater of its explicit floor and the root model version. |
| [tm-deprecated-v1.jsonld](./tm-deprecated-v1.jsonld) | Mark pump v1 as deprecated and link to v2 without redirecting existing references. |

Model identifiers and reference targets are illustrative HTTPS IRIs. The target models and their candidate versions are external to this folder, so parsing the JSON does not establish that the references can be resolved. The TD examples use illustrative endpoint forms and `nosec` security; these are not production security recommendations. The TM deprecation example states deprecation in descriptive metadata and does not introduce a vocabulary term.

To check JSON syntax from the repository root:

```sh
python -c "import json; from pathlib import Path; files = sorted(Path('proposals/td-versioned-references/examples').glob('*.jsonld')); [json.loads(p.read_text(encoding='utf-8')) for p in files]; print(f'{len(files)} JSON-LD examples parsed')"
```

The [multiple-type TD without a model version](./td-multiple-types-no-model-version.jsonld) has an explicit floor on one link and no floor on the other. Each link resolves independently. With no root model version, the absent per-link floor does not cause an error.
