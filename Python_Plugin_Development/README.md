# Python plugin development

Create an editable `MicrostructureChecks` Python plugin starter with two intended responsibilities: checking feature tables and checking phase metadata. Both generated filter classes initially contain a generic float32-array/progress example; neither named check is implemented.

| Pipeline | Task | Output |
|---|---|---|
| [(01) Start a Microstructure Check Plugin](%2801%29%20Start%20a%20Microstructure%20Check%20Plugin.md) | Generate one plugin with `CheckFeatureTable` and `CheckPhaseMetadata`. | Four Python source files, two documentation stubs and three package/environment files. |

No input dataset or preceding pipeline is needed. Use a DREAM3D-NX build containing the SimplnxCore **Create Python Plugin and/or Filters** filter. Native source generation does not require importing simplnx Python bindings. The companion links the pinned official documentation and explains the generated code and the contracts to implement next.

## Run once in a fresh working folder

Choose a writable working folder outside this repository. The pipeline writes to `Data/Output/Python_Plugin_Development/MicrostructureChecks/` relative to it. Preflight reports planned paths and overwrite warnings; execution creates files. New-plugin execution can overwrite edited files and generates new plugin and filter UUIDs.

The following PowerShell commands assume `nxrunner` is available on PATH. Change the pipeline path to your checkout and run them from the chosen working folder. The destination must not exist, even if it is empty.

```powershell
$pipelinePath = 'C:/path/to/NXExamplePipelines/Python_Plugin_Development/(01) Start a Microstructure Check Plugin.d3dpipeline'
$pluginDestination = Join-Path (Get-Location) 'Data/Output/Python_Plugin_Development/MicrostructureChecks'
if (Test-Path -LiteralPath $pluginDestination) {
    throw 'Choose a fresh output folder. Generation can overwrite source files and replace UUIDs.'
}
nxrunner --preflight $pipelinePath
if ($LASTEXITCODE -ne 0) { throw 'Preflight failed.' }
if (Test-Path -LiteralPath $pluginDestination) {
    throw 'The plugin destination must remain absent before execution.'
}
nxrunner --execute $pipelinePath
if ($LASTEXITCODE -ne 0) { throw 'Generation failed; inspect the output before retrying.' }
```

In the GUI, open the pipeline, set an absolute output directory if needed, confirm the appended `MicrostructureChecks` destination is absent, then preflight and execute. Move the generated project into a separate development checkout before adapting it. Do not attach this source-generation step to a material-analysis pipeline that will be rerun.

The [companion guide](%2801%29%20Start%20a%20Microstructure%20Check%20Plugin.md) lists all nine output files and explains the generic behavior, intended validation contracts, UUID stability and matching Python environment requirement. Generated source, plugin loading, package installation and scientific correctness are separate stages. No generated `.py` files or datasets are committed here.

At the pinned source version, new-plugin mode ignores the human-name parameter ([simplnx issue #1822](https://github.com/BlueQuartzSoftware/simplnx/issues/1822)); this example uses matching plugin and human names.

Verified on 2026-10-15 against simplnx `fe6c4a15`: Windows Release preflight and execution passed (2/2 CTest checks), with no files created by preflight and exactly nine files created by execution. Independent checks parsed and syntax-compiled all four Python modules without importing simplnx, and checked the output structure, registration, metadata and UUIDs. Plugin loading, generated-filter execution, package installation and GUI use remain unverified.

The `.d3dpipeline` is the executable authority; its same-stem JSON-compatible YAML records exact parameters. `PythonGeneration.yaml` describes an optional, unverified temporary Python translation of the pipeline. The plugin source files produced by the native generator are outputs, not that pipeline translation.
