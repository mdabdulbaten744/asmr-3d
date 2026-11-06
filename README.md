# Blender Video Factory

Upload a zip -> click **Run workflow** -> download an MP4 from the run's **Artifacts**.
Everything (unzip, Blender install, render, ffmpeg encode, cleanup) is automatic.

## One-time setup
1. Create the repo.
2. Create `.github/workflows/render.yml` (paste the provided file).
3. Put this `README.md` in the root.

## Every video
1. Delete the old zip from the repo root (or tick **delete_zip_after** on the previous run).
2. Upload the new zip to the **root** (it must contain `main.py`).
3. Actions -> **Render Blender Video** -> Run workflow -> pick quality.
4. Open the finished run -> Artifacts -> `video-N` (the MP4). `render-log-N` holds the Blender log.

If several zips exist, the most recently committed one is used (or type a name in `zip_name`).
Each run starts on a fresh VM and re-unpacks into an empty `job/` folder, so nothing carries over.

Quality: `draft` 854x480 (quick test) / `standard` 1280x720 / `high` 1920x1080.
Hosted runners have a 6-hour limit; use `standard` first, `high` only for final renders.

## Contract for any AI asked to make a new zip
Give the AI this text:

> Create a zip for my Blender GitHub-Actions pipeline. Blender 4.2 LTS, Python API only, Cycles CPU, headless.
> - Zip root must contain `main.py` and `job.json` (extra .py modules are allowed; `main.py` must add its own folder to `sys.path`).
> - Run as `blender -b --python main.py -- --job-dir D --out O --quality draft|standard|high`. Parse these args after `--`.
> - Fully procedural: no internet, no downloaded HDRIs/textures/models. Use Principled BSDF node setups, procedural textures, modifiers, bmesh.
> - Build the scene, keyframe the animation, then call `bpy.ops.render.render(animation=True)` with `scene.render.filepath = O/frames/frame_` (PNG, frames start at 1). Do NOT encode video; the workflow does it.
> - Honour `--quality`: draft 854x480 ~20 samples, standard 1280x720 ~64, high 1920x1080 ~112. Cycles, adaptive sampling, OpenImageDenoise, motion blur on.
> - Length 6-12 s at fps from `job.json`. Keep a standard render under ~90 minutes.
> - `job.json`: `{"title":"...","output_name":"snake_case_name","fps":24,"grain":3,"vignette":true,"crf":15}`.
> - Realism checklist: real-world scale, bevelled edges, imperfect shapes, roughness variation, bump/micro detail, SSS for organic material, area-light key/rim/fill, shallow depth of field, slight camera motion, contact shadows, believable physics timing. Calibrate exposure automatically (low-res probe renders) instead of guessing light power.
> - Do not run or test anything; just deliver the zip. Write errors-safe code (try/except around optional extras).

## Troubleshooting
- "main.py not found": zip the *contents*, not a parent folder of folders (one level of nesting is tolerated).
- Black/white video: open `render-log-N`, look for `auto-exposure` lines.
- Blender download failed: re-run; the workflow tries three mirrors and caches Blender after success.
