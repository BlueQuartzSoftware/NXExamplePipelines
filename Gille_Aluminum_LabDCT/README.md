# Gille aluminium LabDCT examples

Keep input datasets and generated results outside this repository, under the documented `Data/...` paths in a writable working folder. The pipeline files can remain here.

These examples use the same public measured AA6016 T4 grain-map crop. Preparation preserves its orientation conventions and grain identities; the two additional OnScale stages demonstrate file and grid conversion. They do not reproduce the associated paper's deformation study.

| Run order | Purpose and dependency |
| --- | --- |
| [01 LabDCT Preparation](%2801%29%20Gille%20Aluminum%20LabDCT%20Preparation.md) | Read the original HDF5, prepare orientations and observed grain statistics, and write native, FFT and SPPARKS outputs. |
| [02 OnScale Export](%2802%29%20Gille%20OnScale%20Export.md) | Read the stage-01 checkpoint, preserve a measured 24³ ROI and export its unchanged grain labels in local metre coordinates. |
| [03 OnScale Import and Uniform Grid](%2803%29%20Gille%20OnScale%20Import%20and%20Uniform%20Grid.md) | Read the stage-02 table into a Rectilinear Grid, recover an equivalent uniform Image Geometry, and restore its known single aluminum phase. |

## Setup and run

1. Choose a writable working folder. Download the original [GrainMapper3D HDF5](https://data.materialsdatafacility.org/mdf_open/0989ee07-e015-494d-8805-8e2703e76763/1.0/sample_S_undeformed_6_grain_center_slice.h5) to `Data/Gille_Aluminum_LabDCT_Examples/sample_S_undeformed_6_grain_center_slice.h5` beneath it. The source [MDF record](https://doi.org/10.18126/yc1m-nk41) documents the dataset and CC BY 4.0 terms.
2. Check **567,693,056 bytes** and SHA256 `8802bbe5413f766330ac394c150a8d75818dd458d9e47123b05b7c8dc53cb522`, for example with `Get-FileHash -Algorithm SHA256`. A partial download is not usable. The original file includes a large absorption-CT volume; this example reads only LabDCT.
3. Create `Data/Output/Gille_Aluminum_LabDCT_Examples/`. With SimplnxCore and OrientationAnalysis loaded, run from the working folder:

```text
nxrunner --preflight "<pipeline-folder>/(01) Gille Aluminum LabDCT Preparation.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(01) Gille Aluminum LabDCT Preparation.d3dpipeline"
nxrunner --preflight "<pipeline-folder>/(02) Gille OnScale Export.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(02) Gille OnScale Export.d3dpipeline"
nxrunner --preflight "<pipeline-folder>/(03) Gille OnScale Import and Uniform Grid.d3dpipeline"
nxrunner --execute "<pipeline-folder>/(03) Gille OnScale Import and Uniform Grid.d3dpipeline"
```

Execute each predecessor before preflighting its successor. In particular, stage 03 must read the exported table during preflight; this dependency is why export and import are separate pipelines. All stages use SimplnxCore; stages 01 and 03 also need OrientationAnalysis.

For GUI use, set absolute file paths if relative `Data/...` paths do not resolve. Stage 01 writes `GilleAluminum.dream3d`, `GilleAluminum_Grains.csv`, `GilleAluminum_FFT.txt` and `GilleAluminum_SPPARKS.txt` under `Data/Output/Gille_Aluminum_LabDCT_Examples/`. Stage 02 writes `OnScale/Export/GilleROI24.flxtbl` and `GilleROI24_Export.dream3d` in that export directory. Stage 03 writes `OnScale/Import/GilleROI24_RoundTrip.dream3d`.

Read the companions before interpreting orientation colours, crop-boundary sizes, coordinate units or material-name indexing. The OnScale cube contains 37 observed grain IDs despite its 935 name slots; its names do not describe 935 physical phases. Text exports require additional solver setup, and this collection does not establish OnScale or Avizo runtime compatibility.

The `.d3dpipeline` is executable authority. YAML records constraints and reference checks; `PythonGeneration.yaml` describes temporary generation. GUI, OOC and generated Python remain unverified. The original data are not included here or installed by this collection. Preserve the [source attribution and CC BY 4.0 terms](SourceDataLicense.md) with derived results.
