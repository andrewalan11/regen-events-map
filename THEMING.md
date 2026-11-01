# Theming the Events & Gatherings map

This guide is for people who fork this repo and want their own look.
The map is plain HTML, CSS and JavaScript. There is no build step.

## What a theme is

A theme has two parts:

1. **One CSS block in `theme.css`**, named `:root[data-theme="name"]`. It sets colour, font and shape tokens (CSS custom properties).
2. **One entry in `window.SITE_CONFIG.themes`** at the top of `index.html`. It gives the theme a label and names the map tiles it uses.

The two names must match exactly.

```js
themes: {
  earthy: { label: 'Earthy', tiles: 'cartoDarkSplit', tilesLite: 'cartoDark' },
  dark:   { label: 'Dark',   tiles: 'cartoDark' }
}
```

Two themes ship with the repo:

- `earthy` - the default. Moss ground, turquoise glass panels, gold edges.
- `dark` - the original look, value for value.

Theme colours live in CSS. The JavaScript holds no theme colours. It only reads a token when it needs a literal value (the browser `theme-color` and the printable route page). A few static literals remain in `index.html`: the initial `<meta name="theme-color">`, the favicon colours, and a small inline fallback `<style>` with earthy values that shows only if `theme.css` fails to load.

## How theme.css is organised

`theme.css` has three layers:

- **(a) Primitives** - optional palette names such as `--moss-900`. Only the theme block that defines them uses them. App CSS never does.
- **(b) Semantic roles** - the contract. All app CSS uses these. Examples: `--bg`, `--text`, `--accent`.
- **(c) Component tokens** - optional. The base `:root` block derives each one from the semantic roles. A theme may override any of them.

The base `:root` block (no `data-theme`) holds fallbacks and derivations. Set every (b) semantic token in your theme. The base fallbacks are dark-navy values, so any you skip will mix dark colours into your theme. Only (c) component tokens are safe to leave out; they derive from your (b) values.

## Add a theme, step by step

1. **Copy the template.** Start from `themes/template.css`. It lists every token the earthy theme sets, with earthy values and a one-line comment each.
2. **Rename it.** Replace `mytheme` with your name. Use lower case and no spaces, for example `forest`.
3. **Set your palette.** Edit section (a) primitives first. Most other tokens point at them.
4. **Fine-tune.** Adjust section (b) and (c). Delete any (c) line you do not need; the base block supplies a default.
5. **Put it on the page.** Pick one:
   - Paste the block(s) into `theme.css`, after the earthy block and before the "Month plumbing" section. Then bump `theme.css?v=` in `index.html` (see below).
   - Or save it as `themes/forest.css` and add `<link rel="stylesheet" href="themes/forest.css?v=1">` in `index.html`, right after the `theme.css` link. Bump that `?v=` on every change too.
6. **Register it.** Add an entry to `SITE_CONFIG.themes`. Put a comma after the entry above yours: the shipped `dark: {...}` entry has no trailing comma, and a missing comma breaks the whole page.
   ```js
   forest: { label: 'Forest', tiles: 'cartoDarkSplit', tilesLite: 'cartoDark' }
   ```
   The label is what the theme picker shows. The picker lists every theme in this object, in order.
7. **Try it.** Open the page with `?theme=forest` in the URL. If you still see Earthy, the name is missing from `SITE_CONFIG.themes` or does not match the CSS block name.

## Token reference

You rarely need this; the template has every token with comments.

Every token below is defined in `theme.css`. "Base" means the base `:root` block sets a default, so a theme may skip it.

### Ground and surfaces
| Token | Use |
|---|---|
| `--bg` | Page ground behind everything |
| `--surface` | Main panel surface; also the browser `theme-color` |
| `--surface-2` | Raised surface: cards, popups |
| `--surface-sunk` | Sunk fields: search, inputs |
| `--surface-rail` | Side rails |
| `--hover` | Hover background |

### Lines and controls
| Token | Use |
|---|---|
| `--border` | Panel borders |
| `--line` | Hairlines and dividers |
| `--control` | Button / control fill |
| `--control-hover` | Control fill on hover |
| `--control-border` | Control outline |
| `--input-border` | Input outlines (base: `--line`) |

### Text
| Token | Use |
|---|---|
| `--text-strong` | Headings, emphasis (base: derived from `--text`) |
| `--text` | Body text |
| `--text-2`, `--text-3`, `--text-4` | Secondary, tertiary, quaternary text (base: derived) |
| `--text-muted` | Muted labels |
| `--text-dim`, `--text-faint` | Dimmer and faintest text (base: derived) |
| `--placeholder` | Placeholder text (base: `--text-muted`) |

### Accents and status
| Token | Use |
|---|---|
| `--accent`, `--accent-hover`, `--on-accent` | Main accent, its hover, and text on it |
| `--accent-2`, `--on-accent-2` | Second accent and text on it |
| `--accent-3` | Third accent |
| `--star` | Starred events (base: `--accent-2`) |
| `--warn`, `--on-warn` | Warning fill and text on it |
| `--danger`, `--on-danger` | Danger / error fill and text on it |
| `--success` | Success |
| `--info` | Info |
| `--tba` | Events whose dates are TBA |
| `--link`, `--link-2` | Links and secondary links |
| `--focus` | Keyboard focus ring |

### Overlays and shadows
| Token | Use |
|---|---|
| `--scrim`, `--scrim-strong` | Modal backdrops |
| `--shadow` | Base shadow colour |
| `--glass-shadow` | Panel and popup shadow |

### Fonts
| Token | Use |
|---|---|
| `--font-ui` | UI font stack |
| `--font-display` | Headings font stack (base: `--font-ui`) |
| `--font-map` | Map controls font (base: `--font-ui`) |

### Month urgency colours
Events are coloured by how soon they start. Month slot 0 is the current month (hottest). Slot 11 is the furthest.

| Token | Use |
|---|---|
| `--urg-hot`, `--urg-cold` | The two ends of the gradient. Set these two. |
| `--month-0` ... `--month-11` | One colour per slot. Base mixes them from `--urg-hot` to `--urg-cold` in oklch. Pin any single stop to fix contrast. |
| `--month-past` | Months already over |
| `--month-none` | Events with no usable month |
| `--on-month` | Default text on a month colour |
| `--on-month-0` ... `--on-month-11` | Text on each slot (base: `--on-month`) |
| `--on-month-past`, `--on-month-tba`, `--on-month-none` | Text on past, TBA and no-month colours |
| `--month-ink-lift` | How far to mix a month colour toward `--text` when it is used as text (0% = raw) |
| `--month-chip-alpha` | Background opacity of month tag chips |
| `--month-badge-text` | Fixed text colour on popup month badges. Base `initial` = use each slot's `--on-month-N`. |
| `--month-dot-text` | Fixed number colour on the route page dots. Only `dark` sets it; if set (to anything), it replaces the per-slot `--on-month-N` there. |

### Map
| Token | Use |
|---|---|
| `--map-bg` | Map ground before tiles load |
| `--map-filter` | CSS `filter` on the base-tile pane only. Costly when panning. |
| `--map-tint` | Colour layer between base tiles and labels |
| `--map-tint-blend` | `mix-blend-mode` of the tint layer |
| `--map-ctrl-bg`, `--map-ctrl-text`, `--map-ctrl-hover`, `--map-ctrl-border` | Zoom buttons |
| `--map-attr-bg`, `--map-attr-text`, `--map-attr-link` | Attribution strip. Keep it readable. |

### Pins and clusters
| Token | Use |
|---|---|
| `--pin-ring`, `--pin-ring-width` | Ring round each pin |
| `--pin-shadow`, `--pin-shadow-hover` | Pin shadow, normal and hover |
| `--pin-gloss` | Layer over the pin colour. `linear-gradient(transparent, transparent)` = flat. |
| `--pin-label`, `--pin-label-hover`, `--pin-label-shadow`, `--pin-label-bg` | Pin captions |
| `--pin-muted` | Pins of past events |
| `--pin-star-badge-bg`, `--pin-star-badge-text`, `--pin-star-badge-edge` | Star badge on pins |
| `--cluster-bg`, `--cluster-text` | Cluster bubble fill and count |
| `--cluster-ring`, `--cluster-ring-width`, `--cluster-ring-inner`, `--cluster-ring-hover` | Cluster rings |
| `--cluster-dot-stroke` | Outline of month dots on clusters |
| `--cluster-shadow` | Cluster shadow |

### Popups
| Token | Use |
|---|---|
| `--popup-bg`, `--popup-text` | Map popup background and text |
| `--popup-glass` | Translucent popup wrapper (where `backdrop-filter` works) |
| `--popup-border` | Popup border, as a full `border` shorthand |
| `--popup-close-bg`, `--popup-close-text` | Popup close button |
| `--tooltip-arrow` | Popup / tooltip arrow |

### Layout chrome
| Token | Use |
|---|---|
| `--bar-bg` | Header row. Gradient allowed. |
| `--filter-bg` | Filter row. Gradient allowed. |
| `--header-edge` | Line under the header and filter rows |
| `--title-text` | Site title |
| `--dock-bg`, `--dock-edge` | Docked columns (timeline right, online left). Gradient allowed. |
| `--timeline-bg` | Timeline column |
| `--panel-bg-solid` | Side panels, modals, menus, dock tabs |
| `--panel-bg` | Translucent panel variant, used only where `backdrop-filter` works |
| `--panel-backdrop` | `backdrop-filter` for panels. `none` = off. |
| `--panel-edge` | Panel border |
| `--row-sep` | List row separators |

### Timeline month bars
Leave all four unset to colour each bar with its month colour (the dark look).

| Token | Use |
|---|---|
| `--tl-month-band` | Bar background. Gradient allowed. |
| `--tl-month-band-text` | Bar text |
| `--tl-month-accent` | Width of a month-colour edge on the bar |
| `--tl-month-past-text` | Bar text for past months |

### Header and controls
| Token | Use |
|---|---|
| `--brand-mark`, `--brand-glass` | Logo mark stroke and fill |
| `--tagline` | Tagline under the title |
| `--search-bg`, `--search-edge`, `--search-edge-hover`, `--search-ring` | Search box |
| `--kbd-bg`, `--kbd-text`, `--kbd-edge` | Keyboard hints |
| `--tg-bg`, `--tg-edge`, `--tg-text`, `--tg-icon`, `--tg-hover` | Toggle groups, icon buttons, segmented controls |
| `--tg-on-bg`, `--tg-on-edge`, `--tg-on-text`, `--tg-on-icon` | Pressed toggles (panel or dock open) |
| `--pill-on-bg`, `--pill-on-text`, `--pill-on-border` | Region pills, selected |
| `--pill-off-text`, `--pill-off-border` | Region pills, off |
| `--menu-icon` | Menu icons |

### Badges and call-to-action
| Token | Use |
|---|---|
| `--badge-bg`, `--badge-text` | Count badges |
| `--badge-star-bg`, `--badge-star-text` | Star badges |
| `--badge-info-bg`, `--badge-info-text` | Info badges |
| `--cta-bg`, `--cta-text`, `--cta-hover`, `--cta-glow`, `--cta-sheen` | Main call-to-action button |

### Internal - do not set
`--c`, `--c-on` and `--c-ink` are set by the "Month plumbing" rules from `data-month`. Use them in CSS if you need the current element's month colour, but do not set them in a theme.

### Also set in a theme block
`color-scheme` (a normal CSS property, not a token). Use `dark` or `light` so native scrollbars and form widgets match.

## Map tiles

Each theme names its tiles in `SITE_CONFIG.themes`.

**`tiles`** is a preset name or a custom object.

| Preset | What it loads |
|---|---|
| `cartoDark` | CARTO `dark_all`: one layer, labels baked in |
| `cartoDarkSplit` | CARTO `dark_nolabels` in the base pane, then the tint pane, then `dark_only_labels` in a separate unfiltered pane |
| `cartoLight` | CARTO `light_all` |
| `cartoVoyager` | CARTO `voyager` |

A missing or unknown preset falls back to `cartoDark`.

A custom object:

```js
tiles: {
  url: 'https://tile.example.org/{z}/{x}/{y}.png',
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  options: { maxZoom: 19 },               // any Leaflet tileLayer options
  labels: {                               // optional: a labels-only layer above the tint
    url: 'https://tile.example.org/labels/{z}/{x}/{y}.png',
    attribution: '',
    options: {}
  }
}
```

Notes:

- `--map-filter` applies to the base pane only. With a split preset (base + labels), labels stay crisp. With a one-layer preset, the filter also hits the baked-in labels.
- Always keep the attribution. OpenStreetMap and CARTO (or your provider) require it.

## Lite mode on phones, tablets and touch screens

Filters on map tiles are slow to pan on weaker devices. So the page has a lite mode.

- `SITE_CONFIG.liteQuery` decides which devices are "lite". Default: `'(max-width: 900px), (pointer: coarse)'`. Keep `liteQuery` set: without it the tile code falls back to `'(max-width: 900px)'` while the page bootstrap marks every device `data-tiles="full"`, so the two disagree.
- On those devices `<html>` gets `data-tiles="lite"` (otherwise `full`).
- The page then loads the theme's **`tilesLite`** preset if it has one. If not, it uses `tiles`.
- Style lite mode in CSS with `:root[data-theme="name"][data-tiles="lite"]`. Earthy uses this to drop the filter and use a cheaper plain tint. See the second block in `themes/template.css`.
- Crossing the breakpoint (resize, rotation, plugging in a mouse) swaps the tiles without a reload.

## The `?v=` cache-buster rule

GitHub Pages caches files for about 10 minutes. Browsers cache longer.

`index.html` loads `theme.css?v=` plus the current value (`s28` at the time of writing). **Bump the `?v=` value on every release that changes `theme.css`.** Any new value works. This link in `index.html` is the only place it is set. Do the same for any extra theme file you link.

If you skip this, visitors may get new HTML with an old `theme.css`, and new tokens will be missing.

## Fonts

Use system font stacks only. No web fonts: no Google Fonts, no font CDNs. They add tracking and slow the first paint.

- UI: `system-ui, -apple-system, "Segoe UI", Roboto, Ubuntu, "Noto Sans", sans-serif`
- Display (earthy): `"Iowan Old Style", "Palatino Linotype", Palatino, "Book Antiqua", Georgia, "Noto Serif", serif`

If you must have one exact face, self-host an OFL-licensed font file inside the repo.

## CARTO key - forks need their own

CARTO basemaps need an API key. The key in `SITE_CONFIG.cartoKey` is locked by Referer to `andrewalan11.github.io`. It will not work on your fork.

- Get a free non-commercial key at dashboard.basemaps.carto.com and put it in `cartoKey`.
- Or use a different tile provider with a custom `tiles` object (see above).

While you are there: the Supabase project in `SITE_CONFIG` also belongs to this site. Bring your own.

## Set the default theme

In `SITE_CONFIG`:

```js
defaultTheme: 'earthy',                           // a theme name, or 'auto'
autoThemes: { dark: 'earthy', light: 'earthy' },  // what 'auto' resolves to
storageKey: 'events-map',                         // prefix for localStorage keys
```

- `defaultTheme` is used on a first visit. Set it to `'auto'` to follow the device's light / dark setting, via `autoThemes`.
- A visitor's choice is saved in localStorage under `<storageKey>-theme` and wins on later visits. Change `storageKey` in your fork so you do not share saved state with this site.
- A `?theme=name` URL parameter wins over both.
- An unknown name falls back to `defaultTheme`, then to the first theme listed. An unknown `?theme=` value also skips the visitor's saved choice: once the URL gives a value, localStorage is not read.
- Browsers without `color-mix()` get the `dark` theme, because it uses the most literal values. Keep a `dark` entry in `SITE_CONFIG.themes` if you care about old browsers.

## Test locally

1. Serve the folder, for example `python3 -m http.server 8000`, and open `http://localhost:8000/?theme=forest`.
2. Expect broken tiles on localhost. The CARTO key is Referer-restricted, so a localhost page gets 403 tiles or a watermark. The Supabase data may also fail. The UI colours still show, so you can check panels, pills, pins and popups.
3. To see real tiles, use your own CARTO key (allow `localhost` for it), or a custom tile provider, or test on your published GitHub Pages site.
4. Check both widths: desktop, and a narrow window or phone (lite mode, phone layout at 900px and below).
5. Check contrast: month pill text, the attribution strip, and focus rings.
