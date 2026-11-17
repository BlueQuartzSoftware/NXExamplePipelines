# Gille OnScale Export

- Executable pipeline: `(02) Gille OnScale Export.d3dpipeline`
- Source Material: [Gille et al. (2024), *3D strain heterogeneity and fracture studied by X-ray tomography and crystal plasticity in an aluminium alloy*](https://doi.org/10.1016/j.ijplas.2024.104146) (source data and scientific context; this is an additional format-conversion illustration).
- Input Data: `Data/Output/Gille_Aluminum_LabDCT_Examples/GilleAluminum.dream3d`, created by [LabDCT Preparation](%2801%29%20Gille%20Aluminum%20LabDCT%20Preparation.md) from the [public dataset](https://doi.org/10.18126/yc1m-nk41); keep datasets and generated results outside this repository.

> **Before adapting:** Do not treat this example as a validated acceptance procedure. Adapt and validate it for the scanner, material, resolution, and quality requirement. Establish the length units, grain-ID meaning and coordinate frame. The table has no unit field, its material names are indexed by grain ID, and this small crop contains incomplete grains. An exported table is only part of a simulation setup.

## At a glance

| Item | What to expect |
| --- | --- |
| Question | How can I export a measured grain-label volume while keeping its scale and grain identity explicit? |
| Prerequisite | Run stage 01 first; `GilleAluminum.dream3d` must already exist for preflight. See [setup and run](README.md). |
| Region | Inclusive source indices **[43,18,113] through [66,41,136]**, in X,Y,Z order |
| Grid | **24 × 24 × 24**, **13,824 cells**, **4 µm** spacing; a **96 µm** cube |
| Population | **37 observed grain IDs**, ranging from **623 to 935 with gaps** |
| Results | `GilleROI24.flxtbl` and `GilleROI24_Export.dream3d` under `Data/Output/Gille_Aluminum_LabDCT_Examples/OnScale/Export/` |
| Next step | [Import and Uniform Grid](%2803%29%20Gille%20OnScale%20Import%20and%20Uniform%20Grid.md), after this export finishes |

## Purpose and real-world setting

The Gille study connects measured aluminium microstructure with deformation and fracture. Its released AA6016 T4 LabDCT crop is also useful for learning how to move categorical grain maps between grid representations. This example adds an OnScale/PZFLEX table export to that preparation workflow. The paper does not supply this pipeline, and this example does not reproduce its Neper meshing or crystal-plasticity finite-element calculations.

The source preparation has dimensions 110 × 60 × 250 and 0.004 mm spacing. A small cubic region makes the exchange easy to inspect and keeps all three export axes equal. The cube is an illustrative subset, with additional crop boundaries; no representative-volume claim follows from its size or grain count.

## Data flow and choices

| Stage | Motivation, alternatives and pitfalls |
| --- | --- |
| Read the preparation checkpoint | Reuse the established source labels and orientation preparation. Reimporting the raw reconstruction would duplicate stage 01 and obscure the dependency. |
| Crop to `MeasuredROI_mm` | Keep the original grain IDs, all cropped cell arrays and the measured placement in millimetres. Inclusive bounds give 24 cells on each axis. Renumbering would make comparison with the original grain table harder. |
| Create `OnScaleROI_m` | Use dimensions [24,24,24], spacing **[0.000004,0.000004,0.000004] m** and origin **[0,0,0]**. This explicitly converts scale and establishes a local ROI frame. It does not rotate the data. |
| Copy `GrainId` | Copy labels unchanged to the metre geometry. Grain IDs are categories, so neither interpolation nor averaging is appropriate. The table exports only this label field and grid coordinates. |
| Write OnScale Table File | Equal dimensions satisfy the writer's X ≥ Y ≥ Z rule without an implicit axis reorder. The writer uses the original `MaterialName` array and supplies fallback names for higher IDs; their meaning is explained below. |
| Remove the temporary full input and inherited grain statistics; save | Crop Image Geometry also copies the full-source `Grain Data` table. Remove it because those sizes and boundary flags describe the predecessor volume, not this ROI. Retain both small geometries, cropped cell arrays and phase metadata. Recompute feature statistics if ROI-specific values are needed; the on-disk preparation retains the original measurements. |

The table writer writes coordinate numbers in the selected geometry's units. The reader used in stage 03 always assigns metres and does not infer a conversion. Exporting the original 0.004 mm spacing unchanged would therefore produce a recovered volume 1,000 times too large in each direction. Merely changing the unit label would cause the same problem.

The local frame is deliberate: an exported position `p_m` maps back to the measured frame as **`p_mm = ROI_origin_mm + 1000 × p_m`**. The retained `MeasuredROI_mm` geometry supplies that origin. No orientation rotation accompanies this translation and scale conversion.

## Outputs and interpretation

| Check | Expected result and meaning |
| --- | --- |
| Native geometries | `MeasuredROI_mm` and `OnScaleROI_m` each contain 13,824 `GrainId` values, equal cell for cell. The former keeps 0.004 mm spacing and measured placement; the latter uses local metre coordinates. |
| Measured crop origin | Approximately **[0.36752726, -0.27009904, 4.1392568] mm**, obtained by adding the crop's lower indices times spacing to the source origin. |
| Coordinate sections | `xcrd`, `ycrd` and `zcrd` each contain **25 bounds**, from **0 to 0.000096 m**, with approximately 0.000004 m increments. Bounds describe cell faces, not centres. |
| `divisions`; `matr` | **24 24 24**; **13,824** original labels in X-fastest order. Compare every label with the native ROI, not just the minimum and maximum. |
| `name 935` | There are slots for IDs **1 through 935**, including unused IDs. This is **not 935 observed grains or physical phases**. |

The source `MaterialName` array is `[InvalidPhase, Aluminum]`. The writer skips row 0, so ID 1 receives `Aluminum`; ID 1 is absent from this ROI. Higher IDs receive `Phase_<id>`, including all 37 active grains. For example, grain 623 is exported as `Phase_623`. These strings are format labels, while the source physical phase remains aluminium. They contain no constitutive properties or grain orientations.

## Adapting the example

For another crop, change the inclusive bounds and recompute the export dimensions, physical spacing and expected label population. Keep the mapping to the measured frame. If unequal dimensions require the writer to reorder axes, inspect the resulting coordinates and cell correspondence before comparing them with the source.

Do not interpret the table's material-name count as a grain count. If a downstream solver needs a distinct orientation or material definition for each grain, establish that mapping explicitly from the retained source data. Loading, boundary conditions, constitutive models and actual OnScale execution remain outside this example. No Avizo import or export is validated here.

Preserve the [CC BY 4.0 attribution](SourceDataLicense.md) and identify the new crop, unit conversion and local translation when sharing derived results. Run stage 03 only after this stage has created the table; its reader needs the file during preflight.
