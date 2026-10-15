# Gille aluminium LabDCT examples

Keep input datasets and generated results outside this repository, under the documented `Data/...` paths in a writable working folder. The pipeline files can remain here.

These examples use the same public measured AA6016 T4 grain-map crop. Preparation preserves its orientation conventions and grain identities; the two additional OnScale stages demonstrate file and grid conversion. They do not reproduce the associated paper's deformation study.

| Run order | Purpose and dependency |
| --- | --- |
| [01 LabDCT Preparation](%2801%29%20Gille%20Aluminum%20LabDCT%20Preparation.md) | Read the original HDF5, prepare orientations and observed grain statistics, and write native, FFT and SPPARKS outputs. |
| [02 OnScale Export](%2802%29%20Gille%20OnScale%20Export.md) | Read the stage-01 checkpoint, preserve a measured 24³ ROI and export its unchanged grain labels in local metre coordinates. |
| [03 OnScale Import and Uniform Grid](%2803%29%20Gille%20OnScale%20Import%20and%20Uniform%20Grid.md) | Read the stage-02 table into a Rectilinear Grid, recover an equivalent uniform Image Geometry, and compare inline phase metadata with a separately authored INI while retaining the known single aluminum phase. |

## Setup and run

1. Choose a writable working folder. Download the original [GrainMapper3D HDF5](https://data.materialsdatafacility.org/mdf_open/0989ee07-e015-494d-8805-8e2703e76763/1.0/sample_S_undeformed_6_grain_center_slice.h5) to `Data/Gille_Aluminum_LabDCT_Examples/sample_S_undeformed_6_grain_center_slice.h5` beneath it. The source [MDF record](https://doi.org/10.18126/yc1m-nk41) documents the dataset and CC BY 4.0 terms.
2. Check **567,693,056 bytes** and SHA256 `8802bbe5413f766330ac394c150a8d75818dd458d9e47123b05b7c8dc53cb522`, for example with `Get-FileHash -Algorithm SHA256`. A partial download is not usable. The original file includes a large absorption-CT volume; this example reads only LabDCT.
3. Create the phase metadata file below before stage-03 preflight. It is an additional local input, not another measured dataset.

### Create the phase metadata file

Stage 03 reads `Data/Gille_Aluminum_LabDCT_Examples/GilleAluminum_Phases.ini`. Its exact contents are:

```ini
[EnsembleInfo]
Number_Phases=1

[1]
CrystalStructure=Cubic_High
PhaseType=PrimaryPhase
```

Run this PowerShell recipe from the writable working folder **outside this repository**. It writes UTF-8 without a BOM, with LF endings and a final LF. An existing canonical file is accepted; any different file is preserved and reported as an error.

```powershell
$gillePhasePath = Join-Path (Get-Location).Path 'Data/Gille_Aluminum_LabDCT_Examples/GilleAluminum_Phases.ini'
$gillePhaseText = "[EnsembleInfo]`nNumber_Phases=1`n`n[1]`nCrystalStructure=Cubic_High`nPhaseType=PrimaryPhase`n"
$gillePhaseEncoding = [System.Text.UTF8Encoding]::new($false)
$gillePhaseBytes = $gillePhaseEncoding.GetBytes($gillePhaseText)
if (Test-Path -LiteralPath $gillePhasePath) {
    if (-not (Test-Path -LiteralPath $gillePhasePath -PathType Leaf)) {
        throw "Expected a file at '$gillePhasePath'. The existing path was preserved."
    }
    $gilleExistingBytes = [System.IO.File]::ReadAllBytes($gillePhasePath)
    if ([Convert]::ToBase64String($gilleExistingBytes) -cne [Convert]::ToBase64String($gillePhaseBytes)) {
        throw "Existing '$gillePhasePath' differs from this example's canonical INI and was preserved. Review your file or use a separate working folder."
    }
} else {
    New-Item -ItemType Directory -Path (Split-Path -Parent $gillePhasePath) -Force | Out-Null
    [System.IO.File]::WriteAllText($gillePhasePath, $gillePhaseText, $gillePhaseEncoding)
}
Get-Item -LiteralPath $gillePhasePath | Select-Object Length, FullName
Get-FileHash -LiteralPath $gillePhasePath -Algorithm SHA256
```

Expect **87 bytes** and SHA256 `1761da5cfdb473bc069ba11540b999553830dd5c4f0aaec34e703f62bbb2bb4d`. If using an editor, preserve the exact spelling, blank line and final newline, select UTF-8 without BOM and LF endings, and check the hash. Do not include the Markdown fences or add a phase-0 section.

`Number_Phases=1` counts the physical phase; the reader adds reserved row 0, producing two rows. It is not the total row count, the 37 observed grains or the 935 material-name slots. The source HDF5 records `PhaseInfo/Phase01/Name = Aluminum`, space group **225** and `F m -3 m`, supporting cubic-high symmetry. All **13,824** cells in this ROI have source `PhaseId=1`. `PrimaryPhase` is an explicit NX role chosen for the example, not a measured source field.

This example-authored transcription illustrates a shared laboratory metadata file supplied separately from a label-only table. The inline table retains the names; Read Ensemble Info imports only `CrystalStructures` and `PhaseTypes` into `Imported Phase Metadata`. Both tables use `PhaseId` as the row index. See [input provenance](DataSources.json).

### Run the stages

Create `Data/Output/Gille_Aluminum_LabDCT_Examples/`. With SimplnxCore and OrientationAnalysis loaded, run from the working folder:

```text
nxrunner --preflight "<pipeline-folder>/(01) Gille Aluminum LabDCT Preparation.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(01) Gille Aluminum LabDCT Preparation.d3dpipeline"
nxrunner --preflight "<pipeline-folder>/(02) Gille OnScale Export.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(02) Gille OnScale Export.d3dpipeline"
nxrunner --preflight "<pipeline-folder>/(03) Gille OnScale Import and Uniform Grid.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(03) Gille OnScale Import and Uniform Grid.d3dpipeline"
```

Execute each predecessor before preflighting its successor. Stage 03 requires **both** the exported `GilleROI24.flxtbl` and the authored `GilleAluminum_Phases.ini` before preflight: its readers open the files to determine dimensions. This table dependency is why export and import are separate pipelines. All stages use SimplnxCore; stages 01 and 03 also need OrientationAnalysis.

For GUI use, set absolute file paths if relative `Data/...` paths do not resolve. Stage 01 writes `GilleAluminum.dream3d`, `GilleAluminum_Grains.csv`, `GilleAluminum_FFT.txt` and `GilleAluminum_SPPARKS.txt` under `Data/Output/Gille_Aluminum_LabDCT_Examples/`. Stage 02 writes `OnScale/Export/GilleROI24.flxtbl` and `GilleROI24_Export.dream3d` in that export directory. Stage 03 writes `OnScale/Import/GilleROI24_RoundTrip.dream3d`.

Read the companions before interpreting orientation colours, crop-boundary sizes, coordinate units or material-name indexing. The OnScale cube contains 37 observed grain IDs despite its 935 name slots; its names do not describe 935 physical phases. Text exports require additional solver setup, and this collection does not establish OnScale or Avizo runtime compatibility.

On 2026-10-15, the six-step stage 03 passed native Windows in-memory Release preflight and execution. Its five-step baseline also passed with the same runtime; 40 independent output checks confirmed the two numeric metadata arrays and preservation of the 10 existing datasets and geometry. The existing figure remains a grain-label preview. Source and pipeline hashes are recorded in the [stage-03 YAML](%2803%29%20Gille%20OnScale%20Import%20and%20Uniform%20Grid.yaml).

The `.d3dpipeline` is executable authority. YAML records constraints and reference checks; `PythonGeneration.yaml` describes temporary generation. GUI, OOC and generated Python remain unverified. The original HDF5 and the recipe-generated INI remain manual external inputs; neither is included here or installed by this collection. Packaging is pending. Preserve the [source attribution and CC BY 4.0 terms](SourceDataLicense.md) with derived results.
