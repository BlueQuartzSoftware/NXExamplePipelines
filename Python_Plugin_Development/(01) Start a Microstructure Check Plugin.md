# Start a Microstructure Check Plugin

- Executable pipeline: [(01) Start a Microstructure Check Plugin.d3dpipeline](%2801%29%20Start%20a%20Microstructure%20Check%20Plugin.d3dpipeline)
- Source Material: The official [Create Python Plugin and/or Filters documentation](https://github.com/BlueQuartzSoftware/simplnx/blob/fe6c4a15e5a5bffef2523f52cec01be33b1e62a9/src/Plugins/SimplnxCore/docs/CreatePythonSkeletonFilter.md), [source-file generator](https://github.com/BlueQuartzSoftware/simplnx/blob/fe6c4a15e5a5bffef2523f52cec01be33b1e62a9/src/Plugins/SimplnxCore/src/SimplnxCore/utils/PythonPluginTemplateFile.hpp) and [Python filter template](https://github.com/BlueQuartzSoftware/simplnx/blob/fe6c4a15e5a5bffef2523f52cec01be33b1e62a9/src/Plugins/SimplnxCore/src/SimplnxCore/utils/PythonFilterTemplate.py), pinned to simplnx `fe6c4a15e5a5bffef2523f52cec01be33b1e62a9`.
- Input Data: None. This filter consumes parameters and writes source files; it does not consume DataStructure objects.

**Before adapting:** Generate into a fresh plugin directory, then move the starter to a separate development checkout. Rerunning new-plugin generation can overwrite edited files and replace plugin/filter UUIDs. The two named checks are not implemented by the generated code.

## At a glance

| Item | Meaning |
|---|---|
| Question | How can a laboratory start custom feature-table and phase-metadata checks as Python filters? |
| Input | Plugin and filter names; no dataset or preceding pipeline. |
| Result | One plugin starter containing two filter classes, two documentation stubs and package metadata. |
| Scope | Generate editable source. Loading, installation, custom validation logic and scientific acceptance are separate work. |

## Purpose and workflow

Feature tables and phase metadata often need laboratory-specific consistency checks before a downstream analysis. This example starts a `MicrostructureChecks` plugin with `CheckFeatureTable` and `CheckPhaseMetadata` classes so a developer has the native plugin structure to adapt. Their names describe intended responsibilities, not completed checks.

Keep this pipeline separate from routine material analysis. Source generation is an initial development step; putting it in an analysis pipeline would regenerate code and identities whenever that analysis is rerun. There is one generator step for both classes and no empty DREAM3D checkpoint.

## Parameters and execution

| Native parameter | Value and reason |
|---|---|
| Use Existing Plugin | Off: create one new plugin starter. |
| Name of Plugin | `MicrostructureChecks`, used for the Python package and plugin class. |
| Human Name of Plugin | `MicrostructureChecks`, matching the package name. |
| Plugin Output Directory | `Data/Output/Python_Plugin_Development` beneath the chosen writable working folder. The generator appends the plugin name. |
| Existing Plugin Location | Empty; inactive in new-plugin mode. |
| Filter Names | `CheckFeatureTable,CheckPhaseMetadata`; exact Python identifiers, with no spaces around the comma. |

Use valid Python identifiers for plugin and class names: letters, digits and underscores, no leading digit, no Python keywords and no spaces. A readable filter display name belongs in its generated `human_name()` method, not in its class name.

At the pinned source version, new-plugin mode ignores `plugin_human_name` and forwards the package name in its place ([simplnx issue #1822](https://github.com/BlueQuartzSoftware/simplnx/issues/1822)). This example therefore uses matching values. Edit the generated plugin's `description()` when adapting it; this limitation is separate from each filter's editable `human_name()`.

Use the [README execution guard](README.md#run-once-in-a-fresh-working-folder) before running. Preflight reports planned new/replaced files and an overwrite warning; it does not create the plugin. Execution writes the files. Review the preflight paths, then execute only if the destination is fresh. For GUI use, set an absolute output path if its working folder differs. Existing-plugin extension mode is an alternative supported by the filter; this example does not exercise it.

## Output files

All nine files are created beneath `Data/Output/Python_Plugin_Development/MicrostructureChecks/`:

```text
MicrostructureChecks/
  src/MicrostructureChecks/
    __init__.py
    Plugin.py
    CheckFeatureTable.py
    CheckPhaseMetadata.py
  docs/
    CheckFeatureTable.md
    CheckPhaseMetadata.md
  pyproject.toml
  environment.yml
  conda/meta.yaml
```

`Plugin.py` holds the plugin identity and filter registration list; `__init__.py` exposes the package and `get_plugin()` entry point. Each filter file holds its identity, parameters, preflight and execution methods. The Markdown files are documentation stubs. The TOML, environment and Conda files are package starters, not evidence of a successful package build or installation. Review their dependency versions, vendor/description placeholders, license and project metadata before packaging.

## What the generated filters actually do

Both classes receive the same generic template. Their parameters are `Created Array` (`output_array_path`) and `Num Tuples` (`num_tuples`, default 0). Preflight rejects zero tuples with error `-65020`; otherwise it requests a `float32` array with tuple shape `[num_tuples]` and component shape `[1]` at the selected output path. The framework applies that array-creation action before execution.

Execution obtains a NumPy view of that array, emits an informational tuple-count message, and runs a sample loop that sends `i=0` through `i=99` while checking cancellation. It does not populate the view or inspect feature tables, phase IDs, crystal structures or material names. Merely changing the class name cannot add those behaviors.

## Contracts to implement next

| Intended filter | Contract to define and implement | Cases to test |
|---|---|---|
| `CheckFeatureTable` | Select the feature Attribute Matrix and required arrays; specify accepted types/component shapes, equal tuple counts, units, feature-ID mapping and the treatment of tuple/ID zero. Inspect required numerical values for finiteness and agreed physical bounds. Report actionable errors without silently repairing the input. | Valid table; missing array; wrong type/shape; count mismatch; invalid feature IDs; NaN/Inf; explicit background and empty-input policies. |
| `CheckPhaseMetadata` | Select phase IDs and ensemble metadata; define unknown/background phase handling. Check referenced IDs against metadata bounds and required phase names/crystal-structure entries using the target application's conventions. A structurally valid phase table is not proof of correct phase identification. | Valid references; negative/out-of-range IDs where the type permits them; missing or mismatched metadata; reserved phase zero; unknown crystal structure; empty data. |

These are proposed development contracts, not behavior supplied by this pipeline. Decide the inputs, outputs and error rules before coding. Replace the sample array parameters and `preflight_impl()` with the required structural checks; implement value-dependent checks in `execute_impl()`, keeping preflight data access consistent with simplnx's execution model. Update `human_name()`, tags, documentation and parameter versioning with the final API.

Move the generated project into its own development checkout before editing. Preserve the plugin `id()` and filter `uuid()` values once adopted: saved pipelines use those identities to find their filters. Keep class/name/clone references consistent. Regenerate only into another fresh directory when comparing newer templates; merge intentional changes without replacing adopted identities.

## Validation boundary

Native generation and Python syntax/structure inspection can establish that the starter files were emitted coherently. They do not prove the filters load or execute. To develop the checks, use simplnx Python bindings and dependencies matching the target application, then test plugin discovery, registration, preflight, execution, cancellation and the contract cases above. Syntax compilation alone does not import `simplnx` or validate its API.

The generated `environment.yml` and `pyproject.toml` are suggestions to review, not a guarantee of a compatible environment. No package installation or plugin-loading procedure is performed by this example. Do not use the starter to accept scientific results until the implemented checks have independent expected outcomes and representative input tests.

The `.d3dpipeline` is authoritative; the same-stem YAML mirrors its arguments. `PythonGeneration.yaml` separately describes an optional, unverified temporary Python translation of this pipeline. That translation is distinct from the four plugin Python files emitted as output. No generated Python file is stored in this example repository.
