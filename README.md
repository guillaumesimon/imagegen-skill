# imagegen

A Claude skill for making images with [fal.ai](https://fal.ai). Illustrations, icons, mockups,
placeholders, product shots, transparent cutouts, contact sheets — the everyday asset stuff you
don't want to leave your editor for.

It's opinionated, and that's the point. The skill never picks for you silently. It asks which
model, which aspect ratio, and whether you want the background gone, with the price of every
option on the table, before it spends a cent.

## What you get

**A cheapest-first model ladder.** Three models, always in the same order, always with prices.
The cheap one is the default.

| # | Model | Price per image | Reach for it when |
|---|---|---|---|
| 1 | Muse Image (Meta) | $0.01 | Default. Fast, cheap, surprisingly good at text, charts and QR codes inside the image |
| 2 | Nano Banana 2 (Google) | $0.08 at 1K, up to $0.16 at 4K | You need photoreal detail, busy composition, or 4K |
| 3 | Grok Imagine Pro (xAI) | $0.05 at 1k, $0.07 at 2k | You want a different look, or very wide ratios |

**Text to image, and image to image.** Up to 10 reference images depending on the model. Local
files go inline as data URLs, so nothing gets uploaded to some bucket first.

**Background removal when you want it.** Ask for a cutout and the image gets piped through Bria
RMBG 2.0 (+$0.018) for a transparent PNG. The version with the background is saved next to it,
so a bad matte doesn't mean paying for the generation twice.

**Contact sheets that actually work.** Ask for a grid, a character sheet or a turnaround and you
get *one* generated image whose prompt describes the grid. Not nine images glued together.
Stitched cells drift — different face, different proportions, different light — which defeats
the whole reason you wanted a sheet. One generation, one consistent subject.

**Format checked before you pay.** Aspect ratio support isn't the same across models. The skill
validates your pick against the model you chose and offers the nearest supported value, instead
of eating a 422 or quietly falling back to something you didn't ask for.

## Before you start

- A [fal.ai API key](https://fal.ai/dashboard/keys). Pay per image, no subscription. Figure
  $0.01 to $0.16 an image depending on model and resolution.
- `curl` and `python3`. Both already there on macOS and most Linux boxes.
- Real network access. Sandboxed environments that block `fal.run` can't run this, no matter
  how valid your key is.

## Install

### Claude Code

```bash
git clone https://github.com/guillaumesimon/imagegen-skill ~/.claude/skills/imagegen
```

Already keep the repo somewhere else? Only one file matters:

```bash
mkdir -p ~/.claude/skills/imagegen
cp SKILL.md ~/.claude/skills/imagegen/SKILL.md
```

Restart Claude Code — skills are read at startup.

Want it in one project only instead of everywhere? Use `<project>/.claude/skills/imagegen/`.

### Claude app, Cowork, claude.ai

Add it as an account skill from the skills panel in settings, pasting this repo's `SKILL.md`.

Heads up: account skills and Claude Code's local skills directory are two separate worlds.
Installing in one doesn't install in the other. Use both? Install in both.

## Your key

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

Don't put your key in `SKILL.md`. It's a plain text file that gets committed, shared, and loaded
straight into the model's context. Environment or Keychain, nowhere else.

## Using it

Just ask. The skill picks up on requests for images, illustrations, icons, mockups, assets,
cutouts and sheets.

```
Generate a hero image for the landing page, abstract gradient, dark theme
```

It'll ask for the model, the ratio and the background treatment, then the resolution if your
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

Spell it out up front and it skips the questions entirely:

```
Generate a 16:9 banner with Nano Banana 2 at 2K, keep the background
```

## What it costs

Per output image, before resolution multipliers:

- Muse Image: $0.01
- Nano Banana 2: $0.08 at 1K. Multipliers: 0.5K ×0.75, 2K ×1.5, 4K ×2
- Grok Imagine Pro: $0.05 at 1k, $0.07 at 2k, plus $0.01 per input image when editing
- Bria background removal: $0.018

A contact sheet is one image, so it costs the same as a single generation no matter how many
cells. Resolution matters more on a sheet though — each cell only gets a slice of the pixels.

## Known limits

- Images only, no video.
- Background removal handles one subject per image, so it can't cut out each cell of a contact
  sheet separately.
- Past roughly 12 cells, per-cell detail on a sheet falls apart.
- Muse Image has no resolution control, which makes it a bad pick for dense sheets.
- Prices and endpoint names come from fal.ai and they move. If a parameter that used to work
  starts throwing 422, check the model pages.

## Making it yours

`SKILL.md` is the entire skill. One file, plain Markdown. Sensible things to change:

- Swap models or reorder the ladder in section 3, then fix the price tables in sections 3 and 4
  to match.
- Delete the mandatory questions in section 4 if you'd rather it just decided. That single
  change does the most to alter how the skill feels.
- Replace the default aspect ratio options with the ones you actually use.
- Point the background removal step at a different matting model in section 9.

## Credits

Models are served by [fal.ai](https://fal.ai): Muse Image by Meta, Nano Banana 2 by Google, Grok
Imagine by xAI, RMBG 2.0 by Bria. This repo is just the skill definition.

Built by [Guillaume Simon](https://guillaumesimon.app).
