# From Pixels to Patterns

A small browser tool that turns an uploaded image into a playable Strudel pattern.

Live demo  
https://taraalcira.github.io/Pixels2Patterns/

## What this is

This project takes a single image and extracts a few simple visual features (colour, brightness, contrast).  
Those values are turned into an affective description (editable text).  
That text and the extracted features then control how a short generative Strudel pattern is built and played in the embedded REPL.

Everything runs in the browser. No backend needed.

## How to use it

1. Open the demo link (or open `index.html` locally).
2. Upload an image.
3. Click **Analyze image → text**  
   This draws the image onto a canvas, computes HSV + contrast, and fills an editable “affective description”.
4. Optionally edit:
   - Root, mode, density, instrument
   - The affective text (keywords influence the mapping)
5. Click **Apply text → REPL**
6. Press **Play** inside the Strudel REPL.

## What gets extracted from the image

The image is resized into a small canvas preview (160×160) and sampled for speed.

- **Hue** (average hue angle in degrees)
- **Sat** (average saturation, 0–1 shown as %)
- **Val** (average brightness/value, 0–1 shown as %)
- **Contrast** (rough luminance standard deviation, normalised to ~0–1)

Contrast is estimated from luma values:
`Y = 0.2126 R + 0.7152 G + 0.0722 B`  
Then the standard deviation of `Y` is computed and divided by 255.

## How the music is generated

### Step 1: Visual → affect labels
A simple rule set converts features into labels:

- warm / cool (from hue)
- dark / mid / bright (from val)
- low / medium / high saturation
- low / medium / high contrast

From these, three higher level labels are estimated:

- **valence**: positive / neutral / negative
- **arousal**: low / medium / high
- **texture**: soft / neutral / sharp

### Step 2: Affect text is editable
The textarea is meant to be part of the process.

If the text includes keywords such as:
- **positive** / **negative**
- **calm** / **energetic**
- **soft** / **sharp**

…those override the labels. This makes the tool co-creative, instead of fully automatic.

### Step 3: Labels + controls → Strudel parameters

- **Hue → root + default mode**
  - A fixed mapping from hue ranges to root notes and major/minor defaults.
- **Valence → mode**
  - positive → major, negative → minor, neutral → hue default
- **Arousal → density**
  - low → low density, high → high density
- **Texture (contrast-based) → instrument**
  - soft → piano, sharp → gm_lead_6_voice, neutral → kalimba
- **Markov chain → melody**
  - A first order Markov process generates a sequence of scale degrees
  - Density controls the sequence length (8, 16, or 32)
  - Random sampling means the melody can change each time, even for the same image

### About tempo and speed
This version avoids controlling global tempo (no `setcps` in generated code).
Instead, the pattern uses relative speed:

- `fast(x)` when the pattern should feel faster
- `slow(1/x)` when it should feel slower

Right now the speed factor is mainly driven by arousal. (This can be extended to use brightness again if desired.)

## Running locally

This is a single HTML file.

Option 1  
Just open `index.html` in a browser.

Option 2 (recommended if file input acts weird)  
Serve it locally:

```bash
python3 -m http.server 8000
```

Then open:  
http://localhost:8000

## Notes and limitations

- The analysis is global and average-based. Two very different images can still sound similar if their colour statistics are similar.
- The Markov chain is intentionally small and simple. It adds variation, but it is not meant to learn musical style.
- The Strudel REPL is embedded via `@strudel/embed`.

## Credits

- Strudel embed: https://www.npmjs.com/package/@strudel/embed  
- Project: Tara Splint & Luca Pouw
