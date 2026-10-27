# Versioned Reference Examples

These JSON-LD documents illustrate SemVer floors and incremental compatibility requirements on existing WoT references. The SemVer examples explicitly declare provisional `tm:minVersion` or `td:minVersion` terms. The incremental examples use the informative `uav` deployment mapping, not proposed core WoT term names. All use the TD 1.1 context with local extension mappings. Parsing their JSON does not implement the [draft's resolution contract](../spec.md).

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
| [td-incremental-extends.jsonld](./td-incremental-extends.jsonld) | Extend a TD in explicit line `control-a` at integer revision `42` or higher in one selected source sequence. |
| [tm-incremental-ref.jsonld](./tm-incremental-ref.jsonld) | Select the newest definitions revision in line `status-a`, without a numeric floor; the containing model's free-form label and root line do not constrain the target. |

Model identifiers and reference targets are illustrative HTTPS and URN IRIs. The target models and their candidate versions are external to this folder, so parsing the JSON does not establish that the references can be resolved. The TD examples use illustrative endpoint forms and `nosec` security; these are not production security recommendations. The TM deprecation example states deprecation in descriptive metadata and does not introduce a vocabulary term.

To check JSON syntax from the repository root:

```sh
python -c "import json; from pathlib import Path; files = sorted(Path('proposals/td-versioned-references/examples').glob('*.jsonld')); [json.loads(p.read_text(encoding='utf-8')) for p in files]; print(f'{len(files)} JSON-LD examples parsed')"
```

The [multiple-type TD without a model version](./td-multiple-types-no-model-version.jsonld) has an explicit floor on one link and no floor on the other. Each link resolves independently. With no root model version, the absent per-link floor does not cause an error.

SemVer TD-to-TD floors use `td:minVersion` on `td:extends` and `td:subthing` links, and beside `td:ref`. A floor caps selection at the next major; omitting it permits selection across majors. The TD instance examples use the same `id` for `version.instance` values `1.4.0` and `2.0.0`, because the identifier names the concrete Thing rather than a compatibility major.

The incremental examples require a provider-declared incremental candidate set and trusted revision/line metadata. Neither a version string nor these context declarations select that profile. Given target history 42/control-a, 43/control-b and 44/control-a with no sticky default, the TD reference selects 44/control-a. A sticky 43/control-b fails the line requirement instead of falling back. The TM example's root `monitor-a` line declares its own revision's compatibility; its dependency independently names `status-a`.

These are authored documents, not registry uploads, complete publication snapshots or executable resolver fixtures. Snapshot-bound composition uses the exact occurrence binding and then validates the same requirements; it does not rerun live highest-revision selection. Section 9 of the draft defines publication closure without introducing another JSON manifest format here.
