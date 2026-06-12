# AGENTS.md -- Root Guidance

**One-sentence purpose:** ComfyUI custom node that interrogates booru-style tags from images using Waifu2x WD 1.4 ONNX tagger models.

**Status:** Actively maintained (forked/custom clone of `pythongosssss/ComfyUI-WD14-Tagger`)
**Primary Audience:** ComfyUI custom node developers, AI agents maintaining this node

## What This Repo Provides

- **WD14 Tagger Node:** ComfyUI node (`WD14Tagger|pysssss`) accepting image tensors, returning booru tag strings.
- **Right-click Quick Tag:** Context menu option on any image-displaying node (LoadImage, SaveImage, PreviewImage) to interrogate without wiring a node.
- **Auto Model Download:** ONNX model and CSV tag files download on first use from HuggingFace.
- **Multi-model Support:** 11 known models from SmilingWolf, plus user-provided `.onnx` + `.csv` pairs in `models/`.
- **Core Philosophy:** Minimal dependencies (`onnxruntime` only), CPU-first inference, runtime model caching.

**Core Value for Agents:** Single-node custom node with clear entry points, no build step, no tests to maintain. Safe to edit Python and JS directly.

## Key Concepts & Data Flows

- **Node Execution Flow:** ComfyUI calls `WD14Tagger.tag()` -> converts tensor to PIL Image -> calls async `tag()` -> downloads model if missing -> runs ONNX inference -> filters tags by threshold -> returns formatted string.
- **Model Storage:** Models live in `models/` (relative to node dir). Each model is a `.onnx` + matching `.csv` pair. Directory is gitignored.
- **Config Chain:** `pysssss.user.json` overrides `pysssss.json`. Settings under `"settings"` key merge into `defaults` dict in `wd14tagger.py`.
- **Web Extension Flow:** `web/js/wd14tagger.js` registers via `app.registerExtension()` -> hooks `beforeRegisterNodeDef` -> adds status tag handler and right-click menu option.
- **API Endpoint:** `/pysssss/wd14tagger/tag` route in `wd14tagger.py` serves the right-click quick-tag feature, reading images from ComfyUI's output/input/temp dirs.

## Constraints & Hard Rules

- **Runtime:** Python 3.x (whatever ComfyUI ships with). No version pinning beyond `onnxruntime`.
- **Never:**
  - Hand-edit files in `models/` -- they are downloaded artifacts.
  - Commit `models/` or `__pycache__/` (both in `.gitignore`).
  - Edit `pysssss.user.json` -- it is a user-local override file.
- **Always:**
  - Keep `requirements.txt` and `pyproject.toml` dependencies in sync.
  - Preserve the ComfyUI node registration pattern: `NODE_CLASS_MAPPINGS`, `NODE_DISPLAY_NAME_MAPPINGS`, `__all__` exports in `__init__.py`.
  - Model names in `pysssss.json` must match actual HuggingFace repo slugs.
- **Style:** Follow existing code style (no linter configured). Python uses snake_case, JS uses camelCase. No type hints currently.
- **Verification:** Load ComfyUI with the node installed. Confirm node appears under `image -> WD14 Tagger`. Test with a sample image.

## Key Files & Responsibilities

| Category | Key Files / Folders | Role |
|----------|---------------------|------|
| **Entry Point** | `__init__.py` | ComfyUI custom node bootstrap; imports from pysssss and wd14tagger; exposes `WEB_DIRECTORY` |
| **Core Logic** | `wd14tagger.py` | `WD14Tagger` class, async `tag()` function, model download, `/pysssss/wd14tagger/tag` API route |
| **Shared Utils** | `pysssss.py` | Logging, config loading, file downloads, JS install/linking, async helpers, node status updates |
| **Configuration** | `pysssss.json` | Model registry URLs, default settings (thresholds, providers, HF endpoint) |
| **Web Extension** | `web/js/wd14tagger.js` | Frontend extension: status tags on nodes, right-click "WD14 Tagger" menu item |
| **Dependencies** | `requirements.txt`, `pyproject.toml` | `onnxruntime` dependency; Comfy registry metadata |
| **Generated / Runtime** | `models/` | Downloaded ONNX models and CSV files (do not edit, gitignored) |
| **User Override** | `pysssss.user.json` | User-local config override (not committed, same shape as `pysssss.json`) |
| **CI** | `.github/workflows/publish_action.yml` | Publishes to Comfy registry on push to `main` when `pyproject.toml` changes |
| **Guidance** | `README.md`, this `AGENTS.md` | Human README + agent startup guide |

## Common Tasks & Routing

- **Add a new model** -> Add entry to `pysssss.json` models dict with HuggingFace URL -> Verify node dropdown includes new model name.
- **Change default settings** -> Edit `"settings"` block in `pysssss.json` -> Restart ComfyUI -> Confirm defaults apply in node UI.
- **Fix tag formatting** -> Edit `tag()` function in `wd14tagger.py` (lines ~94-107) -> Test with batched images in ComfyUI.
- **Modify right-click menu behavior** -> Edit `web/js/wd14tagger.js` `getExtraMenuOptions` callback (~line 108) -> Reload ComfyUI frontend.
- **Fix model download** -> Edit `download_model()` in `wd14tagger.py` (~line 110) and `download_to_file()` in `pysssss.py` -> Test with cleared `models/` directory.
- **Update dependencies** -> Edit `requirements.txt` AND `pyproject.toml` `[project] dependencies` -> Verify both list the same packages.
- **Debug node not loading** -> Check `__init__.py` import chain: `pysssss.init()` must return True -> Check `onnxruntime` is installed -> Check console for `(pysssss:WD14Tagger)` log output.

## Agent Instructions (Always Follow)

- Start every session by referencing this file and the repo's core purpose.
- The node registration contract (`NODE_CLASS_MAPPINGS`, `NODE_DISPLAY_NAME_MAPPINGS`, `__all__` in `__init__.py`) must not be broken.
- `models/` is a runtime directory -- never add model files to git.
- Config changes in `pysssss.json` require ComfyUI restart to take effect.
- There are no tests; verification means loading the node in ComfyUI and running it against an image.
- When suggesting changes: specify exact files, line ranges, and how to verify in ComfyUI.

## Gotchas & Migration Notes

- `pysssss.py` is a shared utility module pattern from pythongosssss's nodes -- do not remove functions used by other extensions unless confirmed unused.
- The JS file imports from relative ComfyUI paths (`../../../scripts/app.js`) -- these must match ComfyUI's web directory structure.
- `wait_for_async()` in `pysssss.py` handles both running and non-running event loops -- do not simplify without testing both code paths.
- Model inference creates a new `InferenceSession` per call -- no model caching between calls (performance consideration if optimizing).
- The `HF_ENDPOINT` env var allows HuggingFace mirror/proxy routing for regions where HF is blocked.

**Completion Standard:** Do not claim work is done without stating:
- Changes made and files affected
- How to verify in ComfyUI (node loads, tag output matches expectation)
- Any remaining gaps or follow-up steps
