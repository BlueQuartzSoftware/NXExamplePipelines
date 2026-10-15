# NOMAD silver nanoparticle coordinate interchange

Import the complete, published **Ag1415ico** configuration from Tarrat and Loffreda's [silver nanoparticle study](https://doi.org/10.1002/smll.75018), inspect its atoms in DREAM3D-NX, and export a LAMMPS coordinate/type table. The 1,415 Ag positions are **simulation-derived DFT-relaxed coordinates**, not measured atom positions. The example performs no relaxation, molecular dynamics, energy calculation, or morphology ranking.

## At a glance

| Item | Contract |
| --- | --- |
| Pipeline | [(01) Silver Nanoparticle Coordinate Interchange](%2801%29%20Silver%20Nanoparticle%20Coordinate%20Interchange.d3dpipeline), 10 native SimplnxCore steps |
| Guide | [Parameter rationale and interpretation](%2801%29%20Silver%20Nanoparticle%20Coordinate%20Interchange.md) |
| Input | One original `Ag1415ico/CONTCAR`, locally named `Ag1415ico.CONTCAR` |
| Source | 1,415 Ag atoms; Mackay icosahedral model; fractional coordinates in a 50 × 50 × 50 Å cell |
| Outputs | `Ag1415ico.lammps` and `Ag1415ico_Interchange.dream3d` |
| Reproduction status | **Illustrative** coordinate interchange using original paper-linked data |

## Source data and license

The paper's Data Availability Statement links [NOMAD dataset 10.17172/NOMAD/2026.03.23-1](https://doi.org/10.17172/NOMAD/2026.03.23-1). Use its [Ag1415ico entry](https://nomad-lab.eu/prod/v1/gui/entry/id/nlCDj9UZ1jDDb1L_X35QnJNJ-yRd), member `SILVER_NANOPARTICLES_DATABASE/ICOSAHEDRA/MACKAY/Ag1415ico/CONTCAR`.

[Download the original CONTCAR](https://nomad-lab.eu/prod/v1/api/v1/uploads/zwh0pQTnT3ek9K4HVAED3Q/raw/SILVER_NANOPARTICLES_DATABASE/ICOSAHEDRA/MACKAY/Ag1415ico/CONTCAR) and save it without editing or reformatting at `Data/NOMAD_Silver_Nanoparticle/Ag1415ico.CONTCAR`, relative to a writable working folder. Only the local filename changes. The input is not included in this repository; data packaging is separate.

Expected size: **182,857 bytes**. SHA-256:

```text
3b43097bdb568ba31f4d8de1a98bd40f45512c80f3f9897d79826cd9924dd9fd
```

The selected entry declares [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Preserve attribution and indicate changes when redistributing derived outputs:

> Source: David Loffreda (2026), NOMAD dataset: VASP_CLEAN_SILVER_NANOPARTICLES_DATABASE, DOI 10.17172/NOMAD/2026.03.23-1, entry nlCDj9UZ1jDDb1L_X35QnJNJ-yRd, Ag1415ico/CONTCAR. Licensed CC BY 4.0. This example converts fractional positions to Cartesian Å, makes a Float32 geometry copy, assigns atom type 1 to Ag, and exports coordinates at six decimal places.

Related publication: Nathalie Tarrat and David Loffreda (2026), *The Fierce Morphological Battle of Silver Nanoparticles: How Irregularities, Vacancies and Defects Tip the Crossing Sizes*, **Small** 22(56), e75018, [DOI 10.1002/smll.75018](https://doi.org/10.1002/smll.75018). The paper has separate CC BY-NC-ND terms; no article image is copied here.

## Run from a fresh working folder

These PowerShell commands create a new run folder, download and verify the one original file, and stop if the folder already exists. Run them from a location where you may create `NOMADSilverRun`.

```powershell
$runDir = Join-Path (Get-Location) 'NOMADSilverRun'
if (Test-Path -LiteralPath $runDir) { throw 'Choose a new run folder to avoid stale outputs.' }
New-Item -ItemType Directory -Path (Join-Path $runDir 'Data/NOMAD_Silver_Nanoparticle') | Out-Null
$inputFile = Join-Path $runDir 'Data/NOMAD_Silver_Nanoparticle/Ag1415ico.CONTCAR'
$sourceUrl = 'https://nomad-lab.eu/prod/v1/api/v1/uploads/zwh0pQTnT3ek9K4HVAED3Q/raw/SILVER_NANOPARTICLES_DATABASE/ICOSAHEDRA/MACKAY/Ag1415ico/CONTCAR'
Invoke-WebRequest -Uri $sourceUrl -OutFile $inputFile
$expectedHash = '3b43097bdb568ba31f4d8de1a98bd40f45512c80f3f9897d79826cd9924dd9fd'
if ((Get-Item -LiteralPath $inputFile).Length -ne 182857 -or
    (Get-FileHash -LiteralPath $inputFile -Algorithm SHA256).Hash.ToLowerInvariant() -ne $expectedHash) {
    throw 'The input does not match the selected original CONTCAR. Do not run this schema against it.'
}
Set-Location -LiteralPath $runDir
```

Set `$nxrunner` to the matching application's `nxrunner.exe` and `$pipelineFile` to this bundle's `.d3dpipeline`, using absolute paths. Then run:

```powershell
& $nxrunner --preflight $pipelineFile
if ($LASTEXITCODE -ne 0) { throw 'Pipeline preflight failed.' }
& $nxrunner --execute $pipelineFile
if ($LASTEXITCODE -ne 0) { throw 'Pipeline execution failed.' }
```

Outputs are written under `Data/Output/NOMAD_Silver_Nanoparticle/` in the working folder. In the GUI, set the reader and both writer file parameters to absolute paths if the application's working folder is uncertain. GUI execution is a separate verification step.

## Read the export in LAMMPS

From the same working folder, these are the read-only LAMMPS input commands:

```text
units metal
atom_style atomic
boundary s s s
read_data Data/Output/NOMAD_Silver_Nanoparticle/Ag1415ico.lammps
```

`units metal` interprets lengths in Å. The `s s s` boundaries preserve the finite cloud on import while allowing the reader's box to adjust. The exporter places atoms on coordinate-extrema box faces: fixed `f f f` boundaries can reject upper-face atoms, and default periodic boundaries can wrap them. Use the stated read configuration when checking unchanged coordinates. This is not a simulation restart: the file supplies no masses, force field, velocities, constraints, or DFT settings.

## Before adapting

- Keep the checksum gate. The reader uses an explicit six-column whitespace schema, not a general CONTCAR parser: coordinate rows are lines 10–1424, exactly 1,415 tuples. Its `Consecutive Delimiters` argument is **false** in this version so repeated spaces do not create empty columns.
- `FractionalXYZ * 50` is specific to the verified scale factor 1 and diagonal 50 Å cell. Another cell, coordinate mode, species list, or atom count needs an independently checked import and coordinate transform.
- Type **1** means Ag by this source contract. `SourceRow1Based` is the original atom-record number, not a bond, grain, or physical phase ID.
- The writer uses Float32 vertices, six-decimal text, zero image flags, and atom-coordinate extrema as box bounds. It does not preserve the original 50 Å vacuum cell. Keep the Float64 coordinates and the original input for precision-sensitive work.
- The current writer has a documented crash for fewer than 1,000 atoms ([simplnx issue 1818](https://github.com/BlueQuartzSoftware/simplnx/issues/1818)). Do not crop, duplicate, or pad atoms to work around it. This selected source contains 1,415 original atoms.

## Verification scope

Native Windows Release preflight and execution passed against simplnx `fe6c4a15` on 2026-10-15. All 41 independent output checks passed, and a read-only LAMMPS 22 Jul 2025 Update 4 import preserved all 1,415 exported atom records using the configuration above. The maximum coordinate difference between original Cartesian positions and exported text was approximately 2.3283 × 10⁻⁶ Å. The [guide](%2801%29%20Silver%20Nanoparticle%20Coordinate%20Interchange.md#outputs-and-interpretation) and YAML sidecar record the precision checks and output hashes. OOC storage, GUI, installation/bookmarks, generated Python, and physical simulation remain unverified. [PythonGeneration.yaml](PythonGeneration.yaml) records how to generate temporary Python without keeping a handwritten pipeline copy.
