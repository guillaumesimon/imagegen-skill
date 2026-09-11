---
name: imagegen
description: "Generate or edit images via fal.ai, from text or from reference images, with optional Bria background removal and single-call contact sheets. Asks the user to pick model, format, quality and background before generating. Use when the user needs illustrations, icons, mockups, placeholder or asset images, a transparent cutout, a contact sheet, or wants an existing image edited or restyled."
---

# Image Generation (fal.ai)

Two modes, three models, one optional cutout step. Nothing is generated before the user has
picked the model, the format, the quality where it applies, and the background treatment.

## 1. Resolve the API key (first, never ask twice)

Use this prefix on every call:

```bash
FAL_KEY="${FAL_KEY:-$(security find-generic-password -s fal.ai -a FAL_KEY -w 2>/dev/null)}"
```

Precedence: `FAL_KEY` in the environment, then the macOS Keychain when the shell runs on a
Mac. If both come back empty, ask the user for the key once, export it for the rest of the
session, and point them at the one-time setup below so they are not asked again:

```bash
# macOS, stores the key in the login Keychain (prompts for the value, nothing in shell history)
security add-generic-password -U -s fal.ai -a FAL_KEY -w

# any platform, in the shell profile
export FAL_KEY="..."
```

(`-U` must come before `-w`, otherwise `-w` swallows it as the password value)

Never print the key, never echo it into a project file, never commit it, never write it into
generated code or config. A 401 means the key is wrong, revoked, or out of credit.

## 2. Network reality check

Some sandboxed environments block outbound access to `fal.run`: the CONNECT comes back 403
regardless of the key. Anthropic-hosted cloud sandboxes are one such case, so this skill only
generates from an environment with direct network access, such as a local terminal. If a call
fails that way, say the environment cannot reach fal.ai, do not retry, do not blame the key,
and do not ask the user for a key.

## 3. Model catalogue

| # | Model | Text-to-image | Edit / reference | Price per output image |
|---|---|---|---|---|
| 1 | Muse Image (Meta) | `meta/muse-image/text-to-image` | `meta/muse-image/edit` | $0.01 |
| 2 | Nano Banana 2 (Google) | `fal-ai/nano-banana-2` | `fal-ai/nano-banana-2/edit` | $0.08 at 1K (0.5K ×0.75, 2K ×1.5, 4K ×2) |
| 3 | Grok Imagine Pro (xAI) | `xai/grok-imagine-image/quality/text-to-image` | `xai/grok-imagine-image/quality/edit` | $0.05 at 1k, $0.07 at 2k (+$0.01 per input image on edit) |

Post-processing: Bria RMBG 2.0, `fal-ai/bria/background/remove`, $0.018 per image.

Endpoint URL is always `https://fal.run/<endpoint id>`.

This order is the user's own ranking, lowest tier first. Keep it in every list you show and
never reorder it. Cost scales with `num_images`, so generate 1 unless the user asked for
variations.

## 4. Ask before generating — mandatory, as multiple choice

Never infer the model, the aspect ratio, the resolution or the background treatment from the
subject matter, and never fall back to an API default. Each unanswered question is blocking.
Use the AskUserQuestion tool when the session has it, otherwise short numbered lists.

### Round 1 — model, aspect ratio, background (one single round)

**Model.** Always ask, listing the three in the order of section 3, each with its price, and
mark Muse Image as the recommended default:

1. Muse Image — $0.01, fastest and cheapest, strong at in-image text, charts and QR codes
2. Nano Banana 2 — $0.08 at 1K, best for photoreal detail and complex composition, up to 4K
3. Grok Imagine Pro — $0.05 at 1k, alternative look, extreme wide ratios

Never collapse this to a single suggestion, and never skip it because the request sounds
important or expensive-looking.

**Aspect ratio.** Offer 3 or 4 options that all three models support, each labelled with what
it is for:

- `1:1` square, icon or avatar
- `16:9` landscape, hero, banner, slide
- `9:16` portrait, mobile, story
- `4:3` or `3:2` classic photo framing

Fit the options to the request rather than always listing the same four, and include the one
you would have picked so a single click is enough.

**Background.** Two options:

- Keep the background as generated
- Remove the background, transparent PNG cutout (+$0.018, one extra step via Bria)

The answer changes the prompt, so it has to come before generating, not after. If the user
chooses removal, add "subject isolated on a plain flat background, clean silhouette, no
shadow cast on the ground" to the prompt: a busy background makes the matting worse. If they
keep the background, describe the scene as usual.

### Round 2 — quality, only when the chosen model has it

The resolution options depend on the model, so ask after round 1, and only for models 2 and
3. Present the real cost, since resolution is the main cost multiplier:

- Nano Banana 2: `0.5K` ($0.06), `1K` ($0.08), `2K` ($0.12), `4K` ($0.16)
- Grok Imagine Pro: `1k` ($0.05), `2k` ($0.07)

Muse Image has no resolution parameter: skip round 2 entirely, say so rather than inventing
an option, and if the user then needs 2K or more, offer to switch to Nano Banana 2.

### Shortcuts

If the user already answered any of these, in this message or earlier in the conversation,
reuse that answer instead of asking again. When you genuinely cannot ask (unattended or
scheduled run), use Muse Image, `1:1`, and keep the background, then state the assumptions in
your reply.

## 5. Validate the format BEFORE generating

Aspect ratio support differs per model. Check the chosen value against the chosen model before
spending a call.

| Model | `aspect_ratio` values | `resolution` | `num_images` | `output_format` (default) |
|---|---|---|---|---|
| Muse Image | `21:9` `16:9` `4:3` `3:2` `1:1` `2:3` `3:4` `9:16` `9:21` | not supported | 1-10 | `jpeg` `png` `webp` (webp) |
| Nano Banana 2 | `auto` `1:1` `16:9` `9:16` `21:9` `4:3` `3:2` and extremes through `4:1`/`1:8` | `0.5K` `1K` `2K` `4K` (1K) | 1-4 | `png` `jpeg` `webp` (png) |
| Grok Imagine Pro | `2:1` `20:9` `19.5:9` `16:9` `4:3` `3:2` `1:1` `2:3` `3:4` `9:16` `9:19.5` `9:20` `1:2`, plus `auto` on edit | `1k` `2k` (1k) | 1-4 | `jpeg` `png` `webp` (jpeg) |

Rules:

- If the chosen ratio is not supported by the chosen model, offer the nearest supported ratio
  first, and name the model that would do it exactly as the alternative. Do not switch models
  on your own.
- Confirm the target file path and extension, and make `output_format` match the extension.
- When background removal is requested, set `output_format` to `png` and use a `.png` target
  path: the cutout carries an alpha channel, which jpeg cannot hold.
- Always confirm with the user before firing a generation. Each call costs money.

## 6. Mode A — text to image

```bash
curl -s https://fal.run/meta/muse-image/text-to-image \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"...","aspect_ratio":"1:1","num_images":1,"output_format":"png"}' \
  > /tmp/fal.json
```

Models 2 and 3 take the same shape, with `resolution` added:

```bash
-d '{"prompt":"...","aspect_ratio":"16:9","resolution":"2K","num_images":1,"output_format":"png"}'
```

## 7. Mode B — image to image (reference)

Edit endpoints take `image_urls`, a list. Accepts HTTP(S) URLs or `data:` URLs, so a local
reference file needs no upload. Limits: Muse 1-10 images, Nano Banana 2 up to 4, Grok up to 3.

Build the data URL (pick the right base64 flag for the platform):

```bash
# macOS
REF="data:image/png;base64,$(base64 -i ref.png | tr -d '\n')"
# Linux
REF="data:image/png;base64,$(base64 -w0 ref.png)"
```

Then send it, building the JSON with python3 so the payload is escaped correctly:

```bash
python3 - "$REF" <<'EOF' > /tmp/body.json
import json, sys
print(json.dumps({
    "prompt": "...",
    "image_urls": [sys.argv[1]],
    "aspect_ratio": "1:1",
    "num_images": 1,
    "output_format": "png",
}))
EOF

curl -s https://fal.run/meta/muse-image/edit \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  --data-binary @/tmp/body.json > /tmp/fal.json
```

Match the MIME type in the data URL to the real file type (`image/jpeg`, `image/webp`).
Base64 inflates payloads by about a third, so downscale a reference over ~5 MB first.

Use mode B, not mode A, whenever the user supplies an image, points at one in the project,
or asks to restyle, extend, clean up or vary something that already exists.

## 8. Contact sheets — one image, one call, never assembled

A contact sheet (also called a grid, a sheet, a character sheet, a turnaround, a set of poses,
a mood board, "N views on one page") is ONE generated image whose prompt describes the grid.
It is a single API call at the normal per-image price.

Never build one by generating several images and stitching them together. Specifically:

- Do not raise `num_images` and montage the results.
- Do not call the endpoint N times and combine the outputs.
- Do not use ImageMagick, `montage`, PIL, ffmpeg, HTML or any other local compositing.
- Do not lay the cells out yourself in any form.

The reason is that assembled cells are independently generated, so the subject drifts between
them: different face, different proportions, different lighting. A single generation keeps one
consistent subject across the whole sheet, which is the entire point of asking for a sheet.

Ask how many cells and the grid shape if the user has not said, then write the layout into the
prompt explicitly. Name the grid, the count, the gutters, and what varies from cell to cell:

```
Contact sheet, 3x3 grid of 9 cells, thin even white gutters, uniform studio lighting and
identical subject across every cell, each cell showing <the thing that varies: a distinct
pose / angle / colorway / expression>, plain neutral background, consistent scale and framing.
```

Three things to get right:

- **Count and grid.** State both, and make them agree (9 cells means 3x3, not "about nine").
  Beyond roughly 12 cells the per-cell detail collapses; say so and suggest fewer cells or a
  second sheet rather than delivering mush.
- **Aspect ratio.** Match the grid: `1:1` for square grids, `16:9` or `3:2` for wide strips,
  `9:16` for tall ones. This overrides the usual ratio question, so explain the link when
  offering the options.
- **Resolution.** Each cell only gets a fraction of the pixels, so a sheet needs more
  resolution than a single image. Recommend Nano Banana 2 at `2K` or `4K` for sheets of 6
  cells or more, and say plainly that Muse Image, with no resolution control, gives small
  cells on a dense sheet.

Background removal does not apply to a contact sheet: Bria mattes one subject per image, not
one per cell. If the user asks for both, say so and offer either a sheet with a plain
background, or separate single generations that can each be cut out.

## 9. Optional step — remove the background (Bria)

Run this only when the user chose removal in section 4. Bria RMBG 2.0,
`fal-ai/bria/background/remove`, $0.018 per image, returns a PNG with alpha.

It takes a single `image_url`, not a list. Feed it the fal-hosted URL the generation
returned, so nothing is uploaded or re-encoded:

```bash
GEN_URL=$(python3 -c 'import json;print(json.load(open("/tmp/fal.json"))["images"][0]["url"])')

python3 - "$GEN_URL" <<'EOF' > /tmp/rmbg.json
import json, sys
print(json.dumps({"image_url": sys.argv[1]}))
EOF

curl -s https://fal.run/fal-ai/bria/background/remove \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  --data-binary @/tmp/rmbg.json > /tmp/cut.json
```

The response is `{"image":{"url":...}}`, singular, not the `images` list the generation
endpoints return. Download that one:

```bash
CUT_URL=$(python3 -c 'import json;print(json.load(open("/tmp/cut.json"))["image"]["url"])')
curl -sL "$CUT_URL" -o assets/out.png
```

The cutout is the deliverable, so save it to the path the user asked for. Keep the
background version too, as `<name>-bg.png` beside it, so a failed matting does not mean
paying for the generation twice. With several generated images, run Bria once per image and
bill accordingly.

If the matting is visibly wrong (halo, missing limb, hole in the subject), say so rather than
delivering it silently: the usual fix is regenerating with a plainer background, not
re-running Bria on the same input.

## 10. Save the result

When no background removal was requested, the generation response is
`{"images":[{"url":...,"width":...,"height":...}]}`. Download it:

```bash
URL=$(python3 -c 'import json;print(json.load(open("/tmp/fal.json"))["images"][0]["url"])')
mkdir -p "$(dirname assets/out.png)" && curl -sL "$URL" -o assets/out.png
```

If the JSON has no `images` key, print the response body: it carries the API error. A 401
means the key is wrong or revoked, a 422 means a parameter value is outside the tables above.

After saving, report the path, the model used, the ratio and resolution, whether the
background was removed, and the total cost including the Bria step.

## 11. Prompt tips

- Name the style explicitly: minimalist, flat design, 3D render, watercolor, photographic.
- State colors rather than implying them.
- For a cutout, ask for a plain flat background and no ground shadow, and avoid transparent,
  translucent or wispy subjects (glass, smoke, loose hair) which matte badly.
- For edits, describe only what should change; these models keep the rest by default.
- For a sheet, put the grid, the cell count and what varies per cell in the prompt, and ask
  for consistent lighting, scale and framing across cells.
- Muse Image handles text, charts and QR codes in-image well. Nano Banana 2 and Grok Imagine
  Pro are the ones to pick for photoreal detail, complex composition, or many reference
  images at once.