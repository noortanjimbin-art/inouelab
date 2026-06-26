# StateMeter · Excel → JSON Converter

A browser-based tool that merges every **Annotation for …** tab in a StateMeter
annotation workbook (`.xlsx`), reads the **Cliplist** as the master roster (so un-annotated clips still appear), validates each clip against the client output schema (v1.5), and exports **one JSON file per clip**.

Everything runs **client-side** — the workbook is parsed in the browser and the
annotation data is never uploaded anywhere. The file is self-contained (the
Excel and ZIP libraries are inlined), so it also works offline.

## Use it

1. Drop the StateMeter `.xlsx` onto the page (or click to choose a file).
2. The summary ribbon shows tabs merged, clip count, complete vs incomplete, and
   total keyframes.
3. Browse the clip list. Each clip shows its keyframe count and a
   **complete / N issues** badge. Click **details** to see the keyframe timeline
   and the full issue list for that clip.
4. Select clips to export:
   - Complete clips are pre-selected on load.
   - Use **All / Complete / Incomplete** filters and the search box.
   - **Select complete** picks only the clean clips; **Select all shown** /
     **Clear** adjust the rest. Incomplete clips can still be selected manually.
5. **Download selected (.zip)** — one `clip_id.json` per clip inside the zip
   (a single selected clip downloads as a plain `.json`).

## Output schema (per clip, client spec v1.5)

```json
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
    {"frame_idx": 4806, "form": "point", "s_low": 0.0, "s_high": 0.0,
     "visibility": "visible", "bbox": {"x": 224, "y": 106, "w": 1216, "h": 948}}
  ]
}
```

## What it does to the data

- **Only `Annotation for …` tabs** are read; all other tabs are ignored.
- **Identical schema for every file** — only the fields above are emitted. Extra
  sheet columns (Measure_Type, S_Raw, S, M_0/1/cur, AT_Status, QA_Status,
  QA_Comment, Screenshot, raw Bbox_* columns) are not included.
- **Variant headers** are matched to the same field (e.g. a descriptive header
  like `(Always Default to "0")\nM_0` maps to `M_0`).
- **Clip_ID normalization** — bare numbers become `clip_<n>` (e.g. `18` → `clip_18`),
  a trailing `.0` is dropped (`clip_18.0` → `clip_18`), while decimals like
  `clip_22.1` and zero-padding like `clip_002` are preserved. Each rename is reported.
- **De-duplicates** keyframes identical on frame_idx + form + s_low + s_high +
  visibility + bbox.
- **Completeness** counts only the JSON fields. A clip is *complete* only if no
  required field is null and it has at least one keyframe.
- **Validation flags** (per client §10.4): null s-values, missing fields,
  anchors not in point form (`anchor_min ≠ anchor_max`), form vs s-value
  mismatch (`point` needs s_low = s_high; `interval`/`bound` need s_low < s_high),
  keyframes outside frame_range, placeholder clips (all keyframes the same frame),
  missing bbox, and a clip appearing in more than one tab.

## Deploy to Vercel (via GitHub)

1. Push this folder to a GitHub repository (`index.html` must keep that name).
2. In Vercel: **Add New → Project → import the repo → Deploy.** No build step —
   set the framework preset to **Other**; it's a static site.
3. Every push redeploys automatically.

To keep the page internal-only, enable **Settings → Deployment Protection**.

## Notes on clip handling

- **Full roster.** The clip list comes from the **Cliplist** tab, so every planned
  clip appears even if no one has annotated it yet. Un-annotated clips show as
  **not annotated** and carry metadata (category, video_uid, annotator, range)
  from Cliplist.
- **Clip IDs are plain numbers** — `clip_1`, `clip_40`, `clip_22.1` (no zero-padding).
- **Categories** are lowercased with underscores — `glass_container`, `cup_fill`.
- **Annotator** is shown on every clip row so issues can be traced to a person.
- **Performance.** Large workbooks (StateMeter files can exceed 150 MB because of
  embedded screenshots) are handled by stripping `xl/media/*` images in-browser
  before parsing — only the sheet data is read, so parsing takes ~2s instead of
  stalling or crashing.
- **Incomplete export.** Incomplete or not-annotated clips can still be exported,
  but a confirmation warns that their JSON will contain nulls or empty keyframes.
