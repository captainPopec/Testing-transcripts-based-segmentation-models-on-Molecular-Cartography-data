# Transcript-Based Segmentation Benchmarking on Molecular Cartography Data

Benchmarks three cell segmentation approaches — **raw MERSCOPE**, **Proseg**, and **Segger** (with and without scRNA reference) — on Molecular Cartography spatial transcriptomics data, evaluated with [SegTraQ](https://github.com/theislab/segtraq).

---

## Workflow

Run notebooks in this order. Steps 2–4 are independent and can run in any order.

```
1a. tiff_2_parquet        .tif masks → nucleus_boundaries.parquet
1b. spottable_2_parquet   MC spot tables + .tif masks → transcripts.parquet

    (after 1a + 1b each sample folder has transcripts.parquet + nucleus_boundaries.parquet)

2.  raw_transcripts2spatial   → raw_spatialdata.zarr
3.  running_proseg            → proseg-output.zarr          (needs GPU)
4.  running_segger            → segger_spatialdata.zarr ×2  (needs GPU, run twice)

5.  segtraq_analysis          → metric CSVs + UMAPs
```

For step 4, run `running_segger` once with `single_cell_used="no"` and once with `"with"` to produce both outputs.

---

## Expected folder layout

```
INPUT_DIR/
  {sample}/
    transcripts.parquet         ← from spottable_2_parquet
    nucleus_boundaries.parquet  ← from tiff_2_parquet
    raw_spatialdata.zarr/       ← from raw_transcripts2spatial

PROSEG_OUTPUT_DIR/
  {sample}/proseg-output.zarr/

SEGGER_OUTPUT_BASE_DIR/
  no_SC_reference/{sample}/segger_spatialdata.zarr/
  with_SC_reference/{sample}/segger_spatialdata.zarr/

SEGTRAQ_OUTPUT_DIR/
  {sample}/umap_leiden.png, celltype_proportions.csv, ...
  ccs_comparison.csv, silhouette_comparison.csv, purity_comparison.csv, aris_comparison.csv
```

---

## Notebooks

| Notebook | What it does | Inputs | Outputs |
|---|---|---|---|
| `tiff_2_parquet` | TIFF mask → boundary polygons | `.tif` masks | `nucleus_boundaries.parquet` |
| `spottable_2_parquet` | MC spot table → transcripts | `.txt` spot tables, `.tif` masks, overview CSV | `transcripts.parquet` |
| `raw_transcripts2spatial` | Raw data → SpatialData | `transcripts.parquet` + `nucleus_boundaries.parquet` | `raw_spatialdata.zarr` |
| `running_proseg` | Run Proseg segmentation | `transcripts.parquet` | `proseg-output.zarr` |
| `running_segger` | Run Segger + post-process | `transcripts.parquet` + `nucleus_boundaries.parquet` | `segger_spatialdata.zarr` |
| `segtraq_analysis` | Compute SegTraQ metrics | All four `.zarr` stores + scRNA reference | Metric CSVs + plots |

---

## Notes

- **GPU required** for Proseg and Segger. The GPU index is set via `CUDA_VISIBLE_DEVICES` inside the respective notebooks (default: `"1"`).
- **Proseg zarr keys** differ from the others (`assignment`, `gene`, `cell` instead of standard names); this is handled in `segtraq_analysis`.
- **Segger run twice**: once unsupervised (`no_SC_reference`) and once scRNA-guided (`with_SC_reference`).
- All paths in the notebooks are set to `Path("")` — fill them in before running.
- Sample folder names must be consistent across all directories (loops use `os.listdir` on the root input folder).
