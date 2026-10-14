> **Future expansion:** If the full text of [Rowenhorst, Callahan, and Ånes (2024)](https://doi.org/10.1107/S1600576723010221) becomes available, revisit this example against the paper and expand it where supported. The present workflow uses the authors' public Ti64 notebook only to support raw-pattern import, selection, and inspection; it does not reproduce the paper's method or results.

# PyEBSDIndex Ti64 raw-pattern example

This example imports a small measured Ti-6Al-4V EBSD acquisition crop, selects two neighboring scan points, and exports their unchanged detector patterns with source identities. It retains all 1,024 input patterns in a DREAM3D checkpoint.

| Pipeline | Task | Main outputs |
|---|---|---|
| [(01) Ti64 Raw Pattern Inspection](%2801%29%20Ti64%20Raw%20Pattern%20Inspection.md) | Follow scan cells (15,15) and (16,15) back to their raw patterns. | DREAM3D checkpoint, 26,912-byte binary stream, and two-row identity manifest. |

## Input setup and execution

Use a DREAM3D-NX build with the OrientationAnalysis and SimplnxCore filters used by the pipeline, including **Read EBSD Pattern File**. The native pipeline does not require installing Python or PyEBSDIndex.

The input is a local derivative of the [PyEBSDIndex Demo dataset](https://doi.org/10.5281/zenodo.8400425), supplied by David Rowenhorst and Patrick Callahan with contributor Håkon Ånes under CC BY 4.0. Its public `PyEBSDIndexDemo.zip` archive contains `PyEBSDIndexDemo/Ti64 Demo/Ti64_930C4hr_Scan4_cr.up1` and the authors' `Ti64_index.ipynb`. Downloading the whole archive requires approximately 20.4 GB; the original UP1 member is approximately 3.36 GB.

The required `Ti64_930C4hr_Scan4_ROI32.up1` is **not a public release asset or an installed application input**. Prepare it from the original member using the recipe below. It retains source scan columns 0–31 and rows 0–31, each complete 116×116 `uint8` pattern, and the 0.25 micrometer scan steps. Only the UP1 header's scan dimensions change. [DataSources.json](DataSources.json) records the exact transformation, byte counts, checksums, and verification limits; the example's original intake did not verify the complete archive MD5 or original member CRC.

<details>
<summary>Prepare the crop from the original UP1 file</summary>

Save this standard-library Python code as `prepare_ti64_crop.py` outside the repository. Pass the extracted original UP1 path and the desired output path. The destination must not exist. The checksum verifies the selected input bytes; it does not verify the unselected source data.

```python
"""Keep complete detector patterns from source scan rows/columns 0 through 31."""
from pathlib import Path
import hashlib
import struct
import sys

source, destination = map(Path, sys.argv[1:3])
with source.open('rb') as stream:
    header = bytearray(stream.read(42))
    if len(header) != 42 or struct.unpack_from('<4I', header) != (3, 116, 116, 42):
        raise ValueError('Expected the original version 3, 116x116 UP1 file.')
    if struct.unpack_from('<II', header, 17) != (500, 500):
        raise ValueError('Expected the original 500-column, 500-row acquisition.')
    if header[16] != 0 or header[25] != 0 or struct.unpack_from('<dd', header, 26) != (0.25, 0.25):
        raise ValueError('Unexpected scan flags or spacing.')
    pixels = 116 * 116
    rows = []
    for row in range(32):
        stream.seek(42 + row * 500 * pixels)
        block = stream.read(32 * pixels)
        if len(block) != 32 * pixels:
            raise ValueError('Source is truncated within the selected region.')
        rows.append(block)
struct.pack_into('<II', header, 17, 32, 32)
result = header + b''.join(rows)
expected = '6d30640bb9d9a6b15949d61a0d518d4b288af9ba9fac15a64e0b61849334b7cbe8fefd60d03c5c50527624945402570922e0ba1a1aa8d8cdf48aa48e1eb521df'
if hashlib.sha512(result).hexdigest() != expected:
    raise ValueError('Selected bytes do not match the documented raw-pattern input.')
destination.parent.mkdir(parents=True, exist_ok=True)
with destination.open('xb') as output:
    output.write(result)
print(f'Created {destination}: {len(result)} bytes; SHA512 matches.')
```

For example, from your writable working folder:

```text
python prepare_ti64_crop.py "C:/data/PyEBSDIndexDemo/Ti64 Demo/Ti64_930C4hr_Scan4_cr.up1" "Data/PyEBSDIndex_Ti64/Ti64_930C4hr_Scan4_ROI32.up1"
```

This preparation step needs Python 3. The resulting file is 13,778,986 bytes; running the native pipeline does not require Python.

</details>

Place the prepared file at `Data/PyEBSDIndex_Ti64/Ti64_930C4hr_Scan4_ROI32.up1` beneath a writable working folder. Open `(01) Ti64 Raw Pattern Inspection.d3dpipeline`, confirm its paths, preflight, then execute. Outputs go to `Data/Output/PyEBSDIndex_Ti64/Inspection/`. If the GUI uses a different working folder, set absolute input and output paths. No preceding pipeline is required, and no input dataset is stored in this repository.

The [companion guide](%2801%29%20Ti64%20Raw%20Pattern%20Inspection.md) explains scan versus detector dimensions, the relative coordinate convention, expected outputs, and adaptation limits. The `.d3dpipeline` is the executable authority; the same-stem YAML records its arguments. This example performs no NLPAR processing, pattern-center refinement, indexing, orientation analysis, or phase discrimination.
