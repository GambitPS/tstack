---
name: codex-image
description: Generate or edit images with Codex. Use whenever a task needs a generated image (concept art, mockup art, icons, textures, reference sheets) instead of drawing it in code.
---

# Codex Image

Claude does not generate images. Codex does, through its built-in image tool,
so every generated image goes through the `codex` CLI. Claude writes the
prompt, runs Codex, and judges the result.

## Run it

Put the prompt on stdin and pass `-` as the prompt argument. This matters with
a reference image: `-i <ref>` takes the next word as another image path, so a
prompt written after it is swallowed and the run fails with "No prompt
provided via stdin".

```bash
codex exec -s read-only --skip-git-repo-check -i ref.png - < prompt.txt > run.log 2>&1
```

- `-s read-only`: Codex only needs to make the image, not touch files.
- `-i <path>` (repeatable): reference images for style, layout or the thing to
  edit. Leave it out for a fresh image.
- Ask for one image per run, and say what it is for (game sprite, UI icon,
  marketing still) and the size or aspect ratio you need.
- Wrap the call in `timeout 600`. A run that times out produced nothing.

The image lands in `~/.codex/generated_images/<session>/`. Take the newest
`.png` path printed in the run's output and copy it where the work needs it.
Never read a path you did not see printed; a stale image from an earlier run
looks like success.

## Batches

For more than a few images, write the jobs to a file (key, prompt) and run a
small worker pool that calls the command above per job, skips keys that
already have output, retries a failed job once, and logs each run. Keep the
pool small (about 4) and respect any limit your instructions set on heavy
local jobs. Check the first few results before you spend the rest of the
batch.

## Judge the result

Look at every image before you use it. Check it against the request and the
reference, not just that a file exists. When an image is wrong, change the
prompt and say what was wrong; do not rerun the same prompt hoping for luck.

Generated images are drafts and references. If the project has rules about
what art may ship (real assets over generated scenes, a style kit, license
limits), they win.
