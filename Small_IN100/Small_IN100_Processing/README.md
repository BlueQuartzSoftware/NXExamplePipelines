# Small IN100 processing

This folder contains **01 Archive** for single-scan inspection and serial-section archiving, and the new **02 Section Alignment and Replay** for estimating and replaying translations on a small prepared crop. The older processing examples 02-09 remain in simplnx and are not included in this collection. New 02 is a different pipeline from the legacy **02 Full Reconstruction**.

| Guide | Input and result |
|---|---|
| [(01) Small IN100 Archive](%2801%29%20Small%20IN100%20Archive.md) | All 117 ANG sections; saves a one-scan inspection and the established H5EBSD archive. |
| [(02) Small IN100 Section Alignment and Replay](%2802%29%20Small%20IN100%20Section%20Alignment%20and%20Replay.md) | Feature Preparation checkpoint; saves an untouched crop, MI estimate, two shift replays and a shift CSV. |

## Run the archive and inspection

Use a writable working folder whose `Data/Small_IN100/` contains **all 117 files**, `Slice_1.ang` through `Slice_117.ang`. These are the normal SmallIN100 example data. If needed, obtain the public [Small_IN100.tar.gz archive](https://www.dream3d.io/Data_Archive/Small_IN100.tar.gz) and extract its `Small_IN100/` folder under `Data/`.

```powershell
Set-Location 'C:\path\to\writable\working-folder'
New-Item -ItemType Directory -Force -Path 'Data\Output\Small_IN100_Processing\Inspection', 'Data\Output\Reconstruction' | Out-Null
& 'C:\path\to\nxrunner.exe' --preflight 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(01) Small IN100 Archive.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Archive preflight failed.' }
& 'C:\path\to\nxrunner.exe' --execute 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(01) Small IN100 Archive.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Archive execution failed.' }
```

For the GUI, open pipeline 01 with **File > Open**. If relative paths do not resolve, set the file input/output parameters to absolute paths. The pipeline saves a one-scan inspection under `Data/Output/Small_IN100_Processing/Inspection/` and the existing archive under `Data/Output/Reconstruction/Small_IN100.h5ebsd`. Read the [archive companion](%2801%29%20Small%20IN100%20Archive.md) before interpreting the frame corrections, quality mask or 2D placeholder Z spacing. After the archive is complete, continue with [Feature Preparation](../Small_IN100_Examples/%2801%29%20Small%20IN100%20Feature%20Preparation.md).

## Run section alignment and replay

Run **Archive → [Feature Preparation](../Small_IN100_Examples/%2801%29%20Small%20IN100%20Feature%20Preparation.md) → Section Alignment and Replay**. Feature Preparation is in the neighboring `Small_IN100_Examples` folder. It already aligns, repairs, segments and cleans the measured stack; the new example compares an alternative pattern estimate on eight consecutive sections from that processed result. It does not recover known acquisition translations.

After Archive succeeds, continue in the same writable working folder:

```powershell
New-Item -ItemType Directory -Force -Path 'Data\Output\Small_IN100_Examples\Preparation', 'Data\Output\Small_IN100_Processing\Alignment' | Out-Null
& 'C:\path\to\nxrunner.exe' --preflight 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Examples\(01) Small IN100 Feature Preparation.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Feature Preparation preflight failed.' }
& 'C:\path\to\nxrunner.exe' --execute 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Examples\(01) Small IN100 Feature Preparation.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Feature Preparation execution failed.' }
& 'C:\path\to\nxrunner.exe' --preflight 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(02) Small IN100 Section Alignment and Replay.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Section Alignment preflight failed.' }
& 'C:\path\to\nxrunner.exe' --execute 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(02) Small IN100 Section Alignment and Replay.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Section Alignment execution failed.' }
```

If the preparation checkpoint already exists, run only the last two commands and their error checks. Its required path is `Data/Output/Small_IN100_Examples/Preparation/SmallIN100_Features.dream3d`. The 17-step alignment pipeline writes `SmallIN100_SectionAlignment.dream3d` and `SmallIN100_Shifts.csv` under `Data/Output/Small_IN100_Processing/Alignment/`. The companion explains the fixed highest-Z anchor, reverse-Z row order, shift sign and invalid padding. No extra archive download is required.

## Source and scope

The archive is declared by OrientationAnalysis CMake with SHA-512:

```text
79e9f6948d4e8e06187e11216a67596fa786ffd2700e51f594ad014090383eb8bcc003e14de2e88082aa9ae512cc4fc9cee22c80066fc54f38c3ebc75267eb5b
```

It contains 117 measured ANG scans, 189 by 201 pixels per section, with 0.25 micrometer XY step and established 0.25 micrometer section pitch. The [official tutorial](https://dream3d.bluequartz.net/Help/2_Tutorials/EBSDReconstruction/) attributes the data to M. Uchic and colleagues at AFRL. The [Groeber and Jackson paper](https://doi.org/10.1186/2193-9772-3-5) provides scientific context; the inspection is illustrative and claims no numerical paper reproduction. Raw-source identity with the publisher's processed supplement and separate archive redistribution terms have not been established. No raw or processed volume files are included here; the preview is derived from native output.

`FilterCoverage.json` indexes all 25 steps across the two companion-backed pipelines. `PythonGeneration.yaml` covers both and records the external Feature Preparation dependency; the original shared specification ID is retained for pipeline 01 compatibility. The `.d3dpipeline` is authoritative. Generated Python, GUI and out-of-core execution need separate validation.

## Alignment validation record

On 2026-10-15, the final 17-step pipeline passed **2/2 native Release CTest checks** (preflight and execution) with the matching SimplnxCore and OrientationAnalysis runtime built from simplnx `fe6c4a15e5a5bffef2523f52cec01be33b1e62a9`. Independent output verification passed **76 checks**. The 1980 × 720 PNG was generated from the native saved IPF arrays and shifts, then visually inspected; gray padding is a display overlay, not saved RGB data.

The new example passed the canonical bundle artifact validator through a private nested-layout adapter and 114 focused authoring checks covering exact YAML parameters, wrapper versions, all suite coverage entries, local links, preview hash and UTF-8 text. This collection retains its existing flat layout.

Verification compared all five retained arrays in the MI result and both replays with an independent bounded integer-transfer calculation from `DataContainer_Before`. It also checked the highest-Z anchor, all eight shift rows, cumulative sums, zero-filled borders, CSV contents, array shapes/types, geometry metadata and valid quaternion norms. All results matched exactly: 128,399 of 131,072 source cells remain, with 2,673 newly invalid border cells. This establishes computational agreement for the selected processed input, not an independent oracle for the MI optimum or physical acquisition-alignment accuracy. Reapplying alignment to this prepared cohort can overcorrect real changes in grain morphology.

| Checked artifact | SHA-256 |
|---|---|
| Feature Preparation input checkpoint | `a3da8f8ccb9f2df3334ecfbaa659070f6f81a37cc027cf3259070f3fa7ecd8be` |
| Native alignment output | `12ec852b5d3597d9f736fe9e76cd97b808d3f7afb8a4dd4e2048493e312be707` |
| Shift CSV | `aca756af7a5d053c6f69053e44d099d29f0c7f05b4f4ac40e7448880faa31e9f` |
| `SmallIN100SectionAlignment.png` | `128a8e2ce914879111654c6fba104ecd15c333b8f1fb83f64d0b9d49caece8e0` |

Implementation constraints are explicit: the chosen crop has all masks true and all phases one, so it does not test the current MI seed-only masking behavior on mixed validity; both replay arrays must have exactly eight tuples even though the source's cumulative-mode preflight does not make the same tuple-count check as relative mode. Explicit geometry creation preserves Micrometer units. The general ASCII writer retains tuple zero; the feature CSV writer's row-zero convention is unsuitable for this shift table.

The pipeline reads the full preparation checkpoint before cropping. Its retained five-array ROI uses 3.5 MiB and the four ROI copies use 14 MiB of cell storage, excluding that initial read and other runtime allocations. This is not an OOC or peak-memory claim. Python generation/execution, OOC storage, GUI execution, install layout and bookmarks remain unverified.
