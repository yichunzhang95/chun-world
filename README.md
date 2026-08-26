# chun-world

A personal site. One page, one file, no build step.

## Run it

```bash
node serve.js
```

Then open http://localhost:8124. Or just double-click `index.html` — it works from the filesystem too.

To deploy, upload `index.html` and the `media/` folder to any static host. Nothing to compile.

## Swap the photos

Four image slots. Replace a photo by **overwriting the file** — no HTML editing:

| File | Where it appears |
|---|---|
| `media/bali.jpg` | Who I am |
| `media/work.jpg` | What I do |
| `media/training.jpg` | How I Spend My Time |
| `media/desk.jpg` | Get in touch |

Any slot whose file is missing renders as an empty bordered frame reading "drop a photo at …", which looks intentional rather than broken.

Portrait or square crops work best; they're displayed at 4:5. Change the captions under each photo in `index.html` (search for `figcaption`).

### If you drop in new photos

Your untouched uploads are kept in **`media/originals/`** — nothing was destroyed. The versions in `media/` went through three steps, and new photos need the same treatment:

1. **Rotation.** `bali.jpg` and `desk.jpg` came off the RX100 with upright pixels but an EXIF `orientation=lower-left` flag. Browsers apply that flag, so both photos rendered on their side. Fixed by stripping the EXIF block entirely.
2. **Resize.** Originals were 5–8 MB each (27 MB total). Now 1400px on the long edge at quality 72 — 828 KB total, still sharp at 2× DPR since the frames render 300px wide.
3. **`object-position`.** `training.jpg` is 16:9 landscape in a 4:5 frame, so a centred crop cut the runner in half. It carries an inline `object-position:64% center`. Any landscape photo dropped into a slot will need the same nudge.

The scripts that did this are in the scratchpad, or just ask and I'll rerun them.

`2026_0523_Run_Club_60.jpg` is in `media/` but unused — there are only four slots. It's the strongest "creator at work" frame, so it's a candidate to replace `work.jpg` if you prefer it.

## Edit the copy

All prose is drafted from `../Content Dashboard/chun-voice.md`. Search `index.html` for **`CONFIRM`** — three HTML comments mark every line I inferred rather than got from you:

1. **Hero line** — "The only measure of a good life is how much you enjoy it." You said this one sticks with you but isn't final.
2. **Who I am** — the middle paragraphs are drafted from the voice guide. Swap in the two or three facts you actually want a stranger to know.
3. **What I do** — client names are deliberately left out until you say which are okay to list publicly.

The page has **four navigable sections** — About, Work, Now, Contact. Three navs list them and the JS section-spy keeps all of them in sync:

| Nav | Shown | Labels |
|---|---|---|
| `<nav id="topnav">` floating pill bar | 1000px and up | About, Work, Now, Contact |
| `<nav id="rail">` left rail | 1200px and up, alongside the bar | Who I am, What I do, Where I'm at, Say hi |
| `<nav id="drawer">` hamburger drawer | below 1000px | About, Work, Now, Contact |

The rail keeps its longer, more descriptive labels; the bar and drawer use the short ones. Align them if you'd rather they matched. The rail only appears at 1200px because `body{padding-left:132px}` reserves the gutter it sits in — the two go together, so don't remove one without the other.

If you add or remove a section, update **all three** nav lists *and* the `sections` selector in the script.

**How I Spend My Time** is a subsection *inside* What I do, not its own section — that's why the navs show four entries and stay on "Work" through both blocks. Four items, each ending in an outbound link with a ↗ arrow: Strava, UNBRKN., Instagram, the unbreakable mind. Link text and URL both live in the `<a class="go">` on each item.

**Where I'm at** is a five-row label/value list — Building, Training, Learning, Where, Focused on. Edit the `<dt>`/`<dd>` pairs; add or drop rows by copying a `.now-row` block.

## Favicon

A cream "C" in Fraunces on the palette's deep ink. Three files at the repo root:

| File | Used for | Ink coverage |
|---|---|---|
| `favicon-16.png` | Browser tab | 12.1% |
| `favicon-32.png` | Browser tab | 11.5% |
| `apple-touch-icon.png` | iOS home screen | 8.2% |

**Every size is rendered natively — none is downscaled from a master**, and that's the whole trick. Fraunces is a high-contrast serif, so area-averaging a 512px render down to 16px washes its hairline strokes into the background: the first attempt measured 3 light pixels out of 256, an effectively blank square. Each file is instead drawn at its target size, with the small ones using a heavier weight (600) and tighter fill so the strokes stay thick enough to read. That's deliberate optical sizing — if you regenerate these, don't "simplify" it by scaling one master to all three.

If you do regenerate them, **measure the result** rather than trusting that the file looks like a PNG. Both failure modes here produced structurally valid PNGs of exactly the right dimensions while being visually blank. Load the file into a canvas and count pixels above ~140 luminance; anything under ~5% means the mark has disappeared.

## How it's put together

- Palette and type come from your existing chun.mov link page (Fraunces + Space Mono, amber, warm paper), inverted to light.
- Scroll behaviour: reveal-on-enter via one `IntersectionObserver` (fires once, then unobserves), navs that track the active section, a top bar that tightens up once the hero is behind you, a scroll-progress bar, sticky marginal photos, and a slow hero drift.
- **Nav clicks run an eased scroll in JS** (`easeOutQuart`, 600–1300ms by distance), with wheel or touch cancelling it mid-flight. Duration is the `Math.min(1300, Math.max(600, …))` line — the single knob if the pace needs tuning.
- **`scroll-behavior` must stay `auto`.** The animation calls `window.scrollTo` once per frame; with `smooth` set, each of those calls starts its own native animation toward the target, so the page trails the values being set and every jump feels rubber-banded. That combination is what made the scroll feel delayed, and it is easy to reintroduce by "helpfully" adding `scroll-behavior: smooth` back.
- The curve is ease-**out**, deliberately. An ease-in-out covers under 1% of the distance in its first 10% of time, so the page looks frozen right after the click. Ease-out is at 34% by then.
- The bar's minimise is driven by its own `IntersectionObserver` on the hero, **not** by the scroll handler. That handler is rAF-throttled, so in a background tab it stalls and the bar would freeze in whichever state it was last in.
- `prefers-reduced-motion: reduce` collapses all of it to instant. With JavaScript disabled the page renders fully visible rather than blank.
- Contrast is AA-clean: body 15.9:1, secondary text 5.0:1, amber labels 5.25:1. `--amber` is decorative only; `--amber-deep` is the text-safe accent. If you change the palette, keep that split.
