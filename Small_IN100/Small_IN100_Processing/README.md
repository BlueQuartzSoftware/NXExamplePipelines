# Small IN100 processing

This folder contains **01 Archive** for single-scan inspection and serial-section archiving, **02 Section Alignment and Replay** for estimating and replaying translations on a prepared crop, and the two-stage **03 Missing Sections Preparation → 04 Nearest Section Fill** comparison. The latter deliberately withholds three sections, copies their nearest retained neighbors and preserves a complete reference to show lost detail. The older processing examples 02-09 remain in simplnx; matching numbers here identify different workflows.

| Guide | Input and result |
|---|---|
| [(01) Small IN100 Archive](%2801%29%20Small%20IN100%20Archive.md) | All 117 ANG sections; saves a one-scan inspection and the established H5EBSD archive. |
| [(02) Small IN100 Section Alignment and Replay](%2802%29%20Small%20IN100%20Section%20Alignment%20and%20Replay.md) | Feature Preparation checkpoint; saves an untouched crop, MI estimate, two shift replays and a shift CSV. |
| [(03) Small IN100 Missing Sections Preparation](%2803%29%20Small%20IN100%20Missing%20Sections%20Preparation.md) | Feature Preparation checkpoint; saves the full eight-section reference, five retained sections and generated sampling decisions. |
| [(04) Small IN100 Nearest Section Fill](%2804%29%20Small%20IN100%20Nearest%20Section%20Fill.md) | Completed stage 03 outputs; fills the original eight-section grid, records each plane's source and saves a boolean feature-label mismatch comparison. |

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

## Run the controlled missing-section comparison

Run **Archive → [Feature Preparation](../Small_IN100_Examples/%2801%29%20Small%20IN100%20Feature%20Preparation.md) → Missing Sections Preparation → Nearest Section Fill**. Section Alignment and Replay is an independent branch and is not needed for this chain. The input is already aligned and cleaned; missing local sections 3, 5 and 6 are deliberately withheld for comparison, not observed acquisition gaps.

After Feature Preparation succeeds, continue in the same writable working folder:

```powershell
New-Item -ItemType Directory -Force -Path 'Data\Output\Small_IN100_Processing\ZSpacing' | Out-Null
& 'C:\path\to\nxrunner.exe' --preflight 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(03) Small IN100 Missing Sections Preparation.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Missing Sections Preparation preflight failed.' }
& 'C:\path\to\nxrunner.exe' --execute 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(03) Small IN100 Missing Sections Preparation.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Missing Sections Preparation execution failed.' }
& 'C:\path\to\nxrunner.exe' --preflight 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(04) Small IN100 Nearest Section Fill.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Nearest Section Fill preflight failed.' }
& 'C:\path\to\nxrunner.exe' --execute 'C:\path\to\NXExamplePipelines\Small_IN100\Small_IN100_Processing\(04) Small IN100 Nearest Section Fill.d3dpipeline'
if ($LASTEXITCODE -ne 0) { throw 'Nearest Section Fill execution failed.' }
```

Stage 03 must **finish execution before stage 04 preflight**. It generates `SmallIN100_MissingSections.dream3d` and `SmallIN100_ZDecisionBounds.txt` under `Data/Output/Small_IN100_Processing/ZSpacing/`. Stage 04 reads both and writes `SmallIN100_NearestSectionFill.dream3d` alongside them. The text file is a generated output, so this chain needs no new dataset or packaged input artifact.

The five-layer `MissingSections_OrdinalStack` is ordinal storage only. Its physical observation offsets are stored separately; do not interpret its packed Z extent as a calibrated specimen. The final file keeps the complete 128 × 128 × 8 reference and a fill on the same grid, with prepared-source Z mapping `[52,53,54,54,56,56,59,59]`. Display the saved `NearestSectionFill/Cell Data/FeatureIdMismatch` with a binary color preset to inspect changed labels. False means the same feature label only; it does not establish equal orientations or correct reconstruction. This comparison requires the complete withheld reference, which would be unavailable in a real acquisition gap. The temporary numerical label difference is deleted because its magnitude has no physical meaning. The guides explain the first-observation-relative midpoint decisions, the lower-section tie rule and the distinction between copied masks and missing-data provenance. Copying complete sections preserves their values but cannot recover missing grain morphology.

## Source and scope

The archive is declared by OrientationAnalysis CMake with SHA-512:

```text
79e9f6948d4e8e06187e11216a67596fa786ffd2700e51f594ad014090383eb8bcc003e14de2e88082aa9ae512cc4fc9cee22c80066fc54f38c3ebc75267eb5b
```

It contains 117 measured ANG scans, 189 by 201 pixels per section, with 0.25 micrometer XY step and established 0.25 micrometer section pitch. The [official tutorial](https://dream3d.bluequartz.net/Help/2_Tutorials/EBSDReconstruction/) attributes the data to M. Uchic and colleagues at AFRL. The [Groeber and Jackson paper](https://doi.org/10.1186/2193-9772-3-5) provides scientific context; the inspection is illustrative and claims no numerical paper reproduction. Raw-source identity with the publisher's processed supplement and separate archive redistribution terms have not been established. No raw or processed volume files are included here; the preview is derived from native output.

`FilterCoverage.json` indexes all 52 steps across the four companion-backed pipelines. `PythonGeneration.yaml` covers all four and records the external Feature Preparation dependency plus the two analysis branches; the original shared specification ID is retained for pipeline 01 compatibility. The `.d3dpipeline` is authoritative. Generated Python, GUI and out-of-core execution need separate validation.

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

## Missing-section comparison validation record

On 2026-10-15, the final pipeline versions passed **4/4 unique native Release CTest checks**: the unchanged 21-step Missing Sections Preparation passed preflight and execution from a clean output directory, and the final six-step Nearest Section Fill passed preflight and execution against those generated files. The required text file was generated by the preparation stage. The matching SimplnxCore and OrientationAnalysis runtime used simplnx `fe6c4a15e5a5bffef2523f52cec01be33b1e62a9`.

Independent output verification passed **75 checks**. Its nearest-source oracle compares actual retained observation centers with target centers, independently of the filter's decision-loop traversal. All five reference fields equal the exact prepared-source crop; every retained section is unchanged, and every filled value matches the independently selected source. The saved boolean `FeatureIdMismatch` has the expected shape and exactly equals an independent reference-versus-fill label-inequality calculation; the numerical `FeatureIdDifference` intermediate is absent. The checks also cover dimensions, origin, spacing, Micrometer units, crystal symmetry, metadata, exactly six decision-file tokens, source-Z labels, source masks, phases and unit quaternions. The output mapping is `[52,53,54,54,56,56,59,59]`, including the lower-section choice at the midpoint for local Z 3.

The 1950 × 750 `SmallIN100MissingSections.png` compares local Z 3 in the complete reference with the fill copied from local Z 2, then displays the native boolean `FeatureIdMismatch` array. It was generated from native output and visually inspected. The two new triplets passed the canonical artifact validator through a private nested-layout adapter and **153 focused authoring checks**, including exact YAML arguments, pinned source schemas, wrapper versions, all 52 suite coverage entries, links, UTF-8 text and preservation of the older 01/02 triplets.

| Local Z | Feature-label mismatches against withheld reference |
|---:|---:|
| 3 | 2,145 |
| 5 | 1,904 |
| 6 | 1,405 |
| All other sections | 0 |

The total is 5,454 cells: 4.1611% of all 131,072 cells, or 11.0962% of the 49,152 deliberately withheld cells. The reference has 231 distinct source labels and the fill has 230. Label `2055` is absent from this cropped ROI after the fill; this does not establish that a whole physical grain disappeared from the specimen. These values describe the checked preparation cohort and omission pattern. They demonstrate lost information, not accurate recovery of the missing sections.

| Checked artifact | SHA-256 |
|---|---|
| Feature Preparation input checkpoint | `a3da8f8ccb9f2df3334ecfbaa659070f6f81a37cc027cf3259070f3fa7ecd8be` |
| Missing Sections Preparation output | `d376c04550ac73dfce1ed592e517948965b8736e3cbd7e4408bb0d88c5f88a9b` |
| Nearest Section Fill output | `6ca3f244a96ca2e9b42fdc5ecdd117da50b98cafbef92dc0351be6a475608225` |
| Generated decision text | `a8eafdc87e71feb36c32b19388ab021caa5d560abdd470c4f464ecac49a513fb` |
| `SmallIN100MissingSections.png` | `8a5af84b790c5059819e2617cc31ab73de5a29336cae93a23e980bed23b9b46c` |

This is a controlled omission from processed measured data. It does not validate real irregular physical-spacing calibration or 3D recovery. The packed five-section geometry uses ordinal storage; the final grid returns to the reference cell centers. All source masks are true, so mask copying does not identify filled positions. The current filter's Micrometer default matches this input and does not prove general unit preservation. Python generation/execution, OOC storage, GUI execution, install layout and bookmarks remain unverified. Full checkpoint loading precedes cropping, so the small comparison volume is not a bounded-memory claim.
