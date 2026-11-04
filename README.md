# Album cover lights for Home Assistant

Colors two lights from the cover art of the music that's playing. A blue cover
gives blue light, a red one gives red. The right light is shifted 30° around the
color wheel so the pair looks like a deliberate gradient, not two identical bulbs.

Built with a Lyngdorf amplifier playing Tidal and two Philips Hue bulbs behind
the TV, but it works with any media player that exposes cover art and any two
color-capable lights.

## What it does

- Takes the dominant color of the current album cover and applies it to both lights.
- Updates on every track change.
- Seven effects: **Static, Breathing, Wave L/R, Color swap, Hue drift, Sparkle, Heartbeat**.
- Brightness, effect speed and a **warm-white cut-off** are sliders on your dashboard.
- Covers that are nearly neutral (white, gray) fall back to warm white.
  The cut-off slider controls what counts as "neutral". Higher means more covers go warm white.
- Very dark covers (black or nearly black) get a dim glow instead, set by the
  **dark cover brightness** slider, so a dark album keeps a dark room.
- Remembers how the lights were before it started and puts them back when you
  switch it off, when the music stops for a minute, or when the TV turns on.
- Starts only when you switch it on. It never takes over your lights by itself.

## Requirements

- Home Assistant 2024.10 or newer (the `triggers:` / `actions:` / `action:` syntax).
- A media player that has `entity_picture` while playing (cover art).
- Two color lights. If your lights have built-in effects (Hue does), the script switches them off so they don't override the colors.
- The built-in **Color Extractor** integration (below).

## Setup

### 1. Color Extractor and allowed URLs

Add this to `configuration.yaml`, then restart Home Assistant:

```yaml
color_extractor:

homeassistant:
  allowlist_external_urls:
    - http://127.0.0.1:8123
```

The second part is easy to miss. Without it, `color_extractor.get_color` is not allowed to
fetch the cover from your own Home Assistant, even though it's a local address.

### 2. Packages

If you don't use packages yet, add this to `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

(Merge it with the `homeassistant:` block above, so you only have one.)

Create a `packages` folder next to `configuration.yaml` and copy `package.yaml`
into it as `album_cover_lights.yaml`.

### 3. Replace the placeholders

In `album_cover_lights.yaml`, use find & replace for these four names:

| Placeholder | Replace with |
|---|---|
| `media_player.YOUR_PLAYER` | your media player |
| `light.YOUR_LIGHT_LEFT` | the left light |
| `light.YOUR_LIGHT_RIGHT` | the right light |
| `media_player.YOUR_TV` | your TV, or delete the TV lines if you don't want that |

### 4. Check and restart

Developer tools → YAML → **Check configuration**, then restart Home Assistant.

After the first start, set the sliders once: brightness about 70, speed 3 to 4,
cut-off 10, dark cover brightness 5. They keep their value across restarts.

### 5. Dashboard card (optional)

Edit dashboard → Add card → Manual, and paste `dashboard_card.yaml`
(replace `media_player.YOUR_PLAYER` in it too).

## Usage

1. Start some music.
2. Switch on **Album cover lights**.
3. Pick an effect and adjust the sliders. Changes apply right away.
4. Switch it off to get your previous lighting back.

## Tuning

- **Gray or blue covers show warm white:** lower the **warm-white cut-off**. A cover
  with 16% saturation only gets color when the cut-off is below 16.
- **Dark covers are too bright or too dim:** change **dark cover brightness** (1 to 30%).
- **Colors look too vivid:** the automation raises saturation to at least 75% so
  muted covers still look good. Change the `75` in the two `hs_color` lines to taste.
- **Hue offset between the lights:** change the `+ 30` (it appears in the automation
  and once in the script call).
- **Only one color per cover:** Color Extractor returns a single dominant color for the whole
  image, so the brown field and red flags of a busy cover won't show up.

## Troubleshooting

- *"Color extractor failed" or nothing happens:* check the `allowlist_external_urls`
  line above and the logs. Open Developer tools → Actions → `color_extractor.get_color`
  with your cover URL to test it by hand.
- *Lights don't restore:* the automation stores their state in a temporary scene called
  `scene.album_lights_before`. If Home Assistant restarts mid-session, switch the
  automation off and on once.
- *Effects stop when changing tracks:* expected. The effect restarts with the new colors.

## Files

| File | What it is |
|---|---|
| `package.yaml` | Helpers, effect script and automation. |
| `dashboard_card.yaml` | A ready-made dashboard card. |

## Feedback

Found a problem or made an improvement? Open an issue or a pull request.
