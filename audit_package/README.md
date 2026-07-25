# Pseudo-label audit package (UniVL-ImgGen)

Addresses reviewer **3FD4 W3** / **MR5**: human validation of pseudo-label
(mask, label) correctness.

- **400 tasks** = 100 per category x 4
  (mask, add, replace, extract).
- Within each category, unique phrases are ranked by frequency and split
  head (top 20%) / mid / tail (bottom 20%),
  sampling 40/30/30.
- At most **3 samples per unique phrase**, so frequent phrases
  (e.g. "person") cannot dominate the audit.
- Task order is shuffled; `manifest.csv` retains full provenance.
- Deterministic for `--seed 0`.

## Import into Label Studio

1. Create a project, paste `labeling_interface.xml` into
   *Settings -> Labeling Interface -> Code*.
2. Serve the images. For local serving:

   ```bash
   export LOCAL_FILES_SERVING_ENABLED=true
   export LOCAL_FILES_DOCUMENT_ROOT=<parent dir of audit_package>
   label-studio start
   ```

   Image URLs are written as `/data/local-files/?d=audit_package/images/...`.
   Re-run with `--url-prefix`/`--url-root` (or S3/GCS URLs) to change them.
3. Import `tasks.jsonl` (*Import* accepts JSONL directly).

## Fields

| field | meaning |
|---|---|
| `source` | original image, before masking / text rendering |
| `condition` | composite the model consumes: mask blackened + label rendered inside |
| `label` | ground-truth phrase rendered into the mask |
| `category` | one of mask, add, replace, extract |

Images are downscaled to a 1024px long edge at JPEG q90
for annotation speed; `manifest.csv` records original paths and resolutions.

## Analysis targets

Per-category correctness by majority vote, Cohen's kappa for Q1/Q2, and
Krippendorff's alpha for Q3 (see `rebuttal_notes.md` MR5).
