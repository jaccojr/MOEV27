# MOEV27: event app engine

One codebase that builds a companion app (PWA) for each event. An event is not
its own app. It is data (`event.json`) plus language files, and the engine
turns that into a static site.

Events planned for 2027: The Ride, Tour for Life, Duchenne Heroes, Africa Classic.

> Status: draft. Lines marked `TODO` need confirming against the real repo.

## How it fits together

| Piece | What it is |
|---|---|
| Engine (`index.template.html` + parts) | The app. Contains no event facts. |
| `event.json` | Everything about one event: stages, camps, times, texts. Format: `engine/schema/event-format-v0.md`. |
| Language files | Every word the app shows, per language. English is the reference language. |
| Build script | Assembles the engine and one event into a deployable folder. |
| Checkers | Two scripts that stop a broken event or incomplete translation from being built. |

Rule of thumb: **facts live in `event.json`, words live in the language files,
logic lives in the engine.** Do not put event facts or Dutch/English text in the
engine code.

## Layout

```
engine/
  web/       build.py, index.template.html, CSS/JS parts, sw.js, compare.html
  lang/      <layer>.<lang>.json files + check_lang.py
  schema/    event.schema.json, check_event.py, event-format-v0.md
  events/    one folder per event with its event.json     (TODO: confirm path)
  convert/   one-off converter from TRD-2026 data.js      (TODO: confirm path)
```

> TODO: the folder names above are taken from the build script (`web/`, `lang/`,
> `schema/`) and from how the files were delivered. Check them against the repo
> and correct this block.

## Build an event locally

Needs Python 3 and `pip install jsonschema`.

```
python3 engine/web/build.py <event.json> <weather-keymap.json | -> <out_dir>
```

Example (path is a placeholder):

```
python3 engine/web/build.py engine/events/trd2026/event.json - dist/trd2026
```

What it does, in order:
1. Fills the template with the CSS, Leaflet, route code and icons.
2. Copies `sw.js`, `compare.html` and the event's `event.json` into `<out_dir>`.
3. Copies only the language files for the languages the event offers.
4. Runs both checkers. If either fails, the build stops with a non-zero exit code.

The result in `<out_dir>` is a plain static site. Open `index.html` through any
static file server to test it.

## The two checkers

- `engine/schema/check_event.py <event.json>`: validates the event against the
  JSON Schema plus cross-checks the schema cannot do (for example that the
  default language is one of the offered languages).
- `engine/lang/check_lang.py [folder]`: for every language file, the keys must
  match English exactly, with the same `{placeholders}` and the same plural
  form. No empty values, no key in two layers.

Both print one problem per line and exit 1 when something is wrong.

## Languages

- Files are named `<layer>.<lang>.json`, for example `ui.en.json`,
  `sport-cycling.nl.json`, `weather.en.json`.
- English (`en`) is the base. Add a language by adding its file for every layer
  and listing it under `languages.offered` in the event.
- Facts (times, distances, coordinates) are typed once. Prose (what riders read)
  has one version per offered language.

## Add or change an event

1. Create or edit the event's `event.json`.
2. Run `check_event.py` on it.
3. Build it (see above) and open the result locally.
4. Commit. Do not edit generated output in `dist/` by hand.

## What this repo does not do yet

- **The build starts at `event.json`.** Turning the organiser's spreadsheet and
  GPX files into `event.json` is not automated here yet.
- **Photos, logos, climb charts and GPX are not copied by `build.py`.**
  TODO: decide where per-event assets live and how they reach the output.
- **Weather data** is fetched separately. The build only copies the keymap
  that tells the app how to read it. TODO: describe where `weather.json` comes
  from for each event.

## Deployment

TODO: fill in once decided.

- Host: Vercel (connected to this repo) or another static host.
- One site per event, each with its own build command, output folder and domain.
- Pushing to `main` should publish. Other branches should get a preview URL.

## Ownership

- Code and the app engine: Jacco.
- Event content (texts, routes, photos): the event organiser.
