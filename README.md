# StateMeter · Excel → JSON Converter

A browser-based tool that merges every **Annotation for …** tab in a StateMeter
annotation workbook (`.xlsx`), validates each clip and keyframe, de-duplicates
repeated keyframes, and exports the StateMeter clip JSON schema.

Everything runs **client-side** — the workbook is parsed in the browser and the
annotation data is never uploaded anywhere.

## Use it

Open `index.html` in any modern browser (or visit the deployed URL):

1. Drop the StateMeter `.xlsx` onto the page (or click to choose a file).
2. Review the validation report and the per-clip keyframe timeline.
3. Toggle **Strict** / **Lenient**:
   - **Strict** (default): excludes keyframes/clips with missing required data and
     lists exactly what to fix.
   - **Lenient**: writes everything; missing values become `null`.
4. Click **Download JSON**.

The file is fully self-contained — the Excel-parsing library is inlined, so it
works offline with no external dependencies.

## What it validates and cleans

- Merges all `Annotation for <name>` tabs (handles a stray leading space in a tab name).
- Normalizes `Form` and `Visibility` (e.g. `partially occluded` → `partially_occluded`).
- Falls back to the `S` column when `S_Low` / `S_High` are blank.
- **De-duplicates** keyframes that match on frame_idx + form + s_low + s_high + visibility.
- Flags: missing fields, placeholder clips (all keyframes the same frame),
  reversed anchors (`anchor_min > anchor_max`), and keyframes outside the clip's frame range.

## Deploy to Vercel (via GitHub)

1. Push this folder to a GitHub repository.
2. In Vercel: **Add New → Project → import the repo → Deploy.**
   No build step or framework preset is needed — it's a static site, so leave the
   build settings empty / "Other".
3. Every push to the repo redeploys automatically.

To keep the page private (internal use only), enable
**Settings → Deployment Protection** in the Vercel project.

## Output schema

```json
[
  {
    "clip_id": "clip_010",
    "ego4d_version": "V2",
    "video_uid": "…",
    "annotator_id": "Annotator 1",
    "category": "Door",
    "frame_range": [4800, 5460],
    "anchor_min_frame": 4806,
    "anchor_max_frame": 5025,
    "keyframes": [
      {"frame_idx": 4806, "form": "point", "s_low": 0.0, "s_high": 0.0, "visibility": "visible"}
    ]
  }
]
```

## Command-line alternative

`xlsx_to_json.py` (in the parent folder) performs the identical conversion from a
terminal: `python xlsx_to_json.py input.xlsx output.json` (requires
`pip install pandas openpyxl`). Use it for batch or automated conversion.
