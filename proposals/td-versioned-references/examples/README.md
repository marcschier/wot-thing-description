# Versioned Reference Examples

These JSON-LD documents use the TD 1.1 context and explicitly declare the proposed `tm:minVersion` string term in the TM namespace. The term is provisional. Every file is parseable JSON; support for the proposed extension requires the vocabulary and processing changes described in the [draft](../spec.md).

| File | Contract |
| --- | --- |
| [tm-extends.jsonld](./tm-extends.jsonld) | Extend pump model v1 at `1.2.0` or a higher eligible compatible version. |
| [tm-submodel.jsonld](./tm-submodel.jsonld) | Use temperature model v2 as a submodel at `2.1.0` or higher. |
| [tm-ref.jsonld](./tm-ref.jsonld) | Select a version of the definitions TM at or above `1.1.0`, then resolve `/properties/status`. |
| [td-single-type.jsonld](./td-single-type.jsonld) | The single type link receives the implicit floor `1.2.0` from `version.model`. |
| [td-multiple-types.jsonld](./td-multiple-types.jsonld) | Each type link has its own floor; the effective floor is the greater of that floor and the root model version. |
| [tm-deprecated-v1.jsonld](./tm-deprecated-v1.jsonld) | Describe deprecation of pump v1 and provide navigation to v2 without redirecting existing references. |

Model identifiers and reference targets are illustrative HTTPS IRIs. The target models and their candidate versions are external to this folder; JSON parsing alone does not demonstrate successful reference resolution. The TD examples include illustrative endpoint forms and `nosec` security for readability, not as a production security recommendation. The TM deprecation example uses descriptive metadata rather than proposing a deprecation vocabulary term.

To check JSON syntax from the repository root:

```sh
python -c "import json; from pathlib import Path; files = sorted(Path('proposals/td-versioned-references/examples').glob('*.jsonld')); [json.loads(p.read_text(encoding='utf-8')) for p in files]; print(f'{len(files)} JSON-LD examples parsed')"
```

The [multiple-type TD without a model version](./td-multiple-types-no-model-version.jsonld) has one explicitly floored link and one unfloored link. Each resolves independently; the absent floor is not an error.
