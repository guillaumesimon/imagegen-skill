# 🎨 imagegen

A Claude skill for making images with [fal.ai](https://fal.ai). Illustrations, icons, mockups,
placeholders, product shots, transparent cutouts, contact sheets — all the everyday asset stuff
you'd rather not leave your editor for. ✨

It's opinionated, and that's the whole point. The skill never picks for you behind your back. It
asks which model, which ratio, and whether you want the background gone — with the price of every
option on the table — before it spends a single cent. 💸

## 🧰 What you get

### 🪜 A five-rung model ladder

Same order every time, prices always visible, the cheap one is the default.

| # | Model | Price per image | Reach for it when |
|---|---|---|---|
| 1 | 🐣 Muse Image (Meta) | $0.01 | Default. Fast, cheap, weirdly good at text, charts and QR codes inside the image |
| 2 | 🍌 Nano Banana 2 (Google) | $0.08 at 1K → $0.16 at 4K | Photoreal detail, busy composition, or you just want 4K |
| 3 | 🤖 Grok Imagine Pro (xAI) | $0.05 at 1k, $0.07 at 2k | A different look, or very wide ratios |
| 4 | 👑 GPT Image 2.5 Flare (OpenAI) | $0.053 at `high`, $0.006 → $0.21 across the slider | It has to be right the first time. Tops both arena leaderboards, does transparent natively, eats 16 reference images |
| 5 | 🖨️ Reve 2.1 | $0.25 flat | Poster-size, native 4K, and the best in-image typography of the bunch |

Rungs 1–3 are the everyday ladder. 4 and 5 are the premium tier — the skill won't push them on
you unless the job actually calls for it. 🙅

### 🖼️ Text to image, and image to image

Up to 16 reference images depending on the model. Local files go inline as data URLs, so nothing
gets shoved into some bucket first. On Reve you can even point at a specific reference from the
prompt with `<frame>1</frame>` — handy for "the jacket from frame 1 on the model in frame 2". 🧥

### ✂️ Background removal, two ways

Ask for a cutout and you get one:

- 🆓 **GPT Image 2.5** does it natively (`background: "transparent"`). No second call, no extra
  cost, and hair and thin edges survive way better.
- 🩹 **Everything else** goes through Bria RMBG 2.0 (+$0.018). The version *with* the background
  gets saved right next to it, so a bad matte doesn't mean paying for the generation twice.

### 🧩 Contact sheets that actually work

Ask for a grid, a character sheet or a turnaround and you get **one** generated image whose prompt
describes the grid. Not nine images glued together. 🚫🪡

Stitched cells drift — different face, different proportions, different light — which defeats the
entire reason you wanted a sheet. One generation, one consistent subject.

### 🛡️ Format checked before you pay

Aspect ratio support isn't the same across models, and GPT Image 2.5 doesn't even speak
`aspect_ratio` — it wants named `image_size` values. The skill validates your pick against the
model you chose and offers the nearest supported value, instead of eating a 422 or quietly
falling back to something you never asked for. 🎯

## ✅ Before you start

- A [fal.ai API key](https://fal.ai/dashboard/keys). Pay per image, no subscription. Figure one cent
  for a quick Muse draft, twenty-five for a Reve poster. 🪙
- `curl` and `python3`. Both already on macOS and basically every Linux box.
- Real network access. Sandboxes that block `fal.run` can't run this, no matter how valid your key
  is. 🚧

## 📦 Install

### Claude Code

```bash
git clone https://github.com/guillaumesimon/imagegen-skill ~/.claude/skills/imagegen
```

Already keep the repo somewhere else? Only one file matters:

```bash
mkdir -p ~/.claude/skills/imagegen
cp SKILL.md ~/.claude/skills/imagegen/SKILL.md
```

Restart Claude Code — skills are read at startup. 🔄

Want it in one project only instead of everywhere? Use `<project>/.claude/skills/imagegen/`.

### Claude app, Cowork, claude.ai

Add it as an account skill from the skills panel in settings, pasting this repo's `SKILL.md`.

⚠️ Heads up: account skills and Claude Code's local skills directory are two separate worlds.
Installing in one doesn't install in the other. Use both? Install in both.

## 🔑 Your key

The skill checks these in order and only bothers you if both come up empty.

**Environment variable**, works anywhere:

```bash
export FAL_KEY="your-key"   # ~/.zshrc, ~/.bashrc, or a project .env
```

**macOS Keychain**, nothing in plain text on disk, nothing in your shell history:

```bash
security add-generic-password -U -s fal.ai -a FAL_KEY -w
```

It prompts for the value and echoes nothing back. Check it landed:

```bash
security find-generic-password -s fal.ai -a FAL_KEY -w
```

🚨 Don't put your key in `SKILL.md`. It's a plain text file that gets committed, shared, and loaded
straight into the model's context. Environment or Keychain, nowhere else.

## 💬 Using it

Just ask. The skill picks up on requests for images, illustrations, icons, mockups, assets,
cutouts and sheets.

```
Generate a hero image for the landing page, abstract gradient, dark theme
```

It'll ask for the model, the ratio and the background treatment, then the quality tier if your
model has one, then generate and tell you the path, the settings and what it cost.

A few more:

```
Make me an app icon, minimalist, gradient blue background with a white geometric shape
```

```
Take public/references/bottle.png and restyle it as a studio product shot on white
```

```
I need a contact sheet, 3x3, of nine poses of the same character
```

```
Generate a product photo of the sneaker and cut out the background
```

```
Design an A2 event poster, heavy typography, with Reve
```

Spell it out up front and it skips the questions entirely: ⚡

```
Generate a 16:9 banner with GPT Image 2.5 at high, keep the background
```

## 💰 What it costs

Per output image, before resolution multipliers:

- 🐣 Muse Image: $0.01
- 🍌 Nano Banana 2: $0.08 at 1K. Multipliers: 0.5K ×0.75, 2K ×1.5, 4K ×2
- 🤖 Grok Imagine Pro: $0.05 at 1k, $0.07 at 2k, plus $0.01 per input image when editing
- 👑 GPT Image 2.5 Flare at 1024²: `low` $0.006, `medium` $0.013, `high` $0.053, `xhigh` $0.094,
  `max` $0.21. At 4K: $0.011 / $0.026 / $0.10 / $0.18 / $0.40. Edits bill the input images and a
  long prompt on top.
- 🖨️ Reve 2.1: $0.25, one price, always native 4K
- ✂️ Bria background removal: $0.018 (free on GPT Image 2.5, which does it itself)

Fun fact: GPT Image 2.5 at its default `high` costs **less** than Nano Banana 2 at 1K while sitting
above it on both leaderboards. The scary $0.21 number is the `max` tier. 🤯

A contact sheet is one image, so it costs the same as a single generation no matter how many cells.
Resolution matters more on a sheet though — each cell only gets a slice of the pixels. 🔍

## 🚧 Known limits

- Images only, no video. 🎥❌
- Background removal handles one subject per image, so it can't cut out each cell of a contact
  sheet separately — native transparent mode included.
- Past roughly 12 cells, per-cell detail on a sheet falls apart.
- Muse Image and Reve 2.1 have no resolution control. Muse is therefore a bad pick for dense
  sheets; Reve is the opposite problem, you pay for 4K whether you need it or not.
- Reve is a text-to-image pick. It ranks below Nano Banana 2 at editing, so don't reach for it to
  retouch things.
- Prices and endpoint names come from fal.ai and they move. If a parameter that used to work starts
  throwing 422, check the model pages. 📉

## 🛠️ Making it yours

`SKILL.md` is the entire skill. One file, plain Markdown. Sensible things to tweak:

- Swap models or reorder the ladder in section 3, then fix the price tables in sections 3, 4 and 5
  to match.
- Delete the mandatory questions in section 4 if you'd rather it just decided. That single change
  does the most to alter how the skill feels. 🎚️
- Replace the default aspect ratio options with the ones you actually use.
- Point the background removal step at a different matting model in section 9.

## 🙏 Credits

Models are served by [fal.ai](https://fal.ai): Muse Image by Meta, Nano Banana 2 by Google, Grok
Imagine by xAI, GPT Image 2.5 by OpenAI, Reve 2.1 by Reve, RMBG 2.0 by Bria. This repo is just the
skill definition.

Built by [Guillaume Simon](https://guillaumesimon.app). 👋
