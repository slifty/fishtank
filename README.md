# 🐠 Fish Tank 🐠

A calm, interactive aquarium that lives in your browser. Tap to feed the fish,
add new creatures, plant seaweed, and watch the tank drift through day and
night. Built to be friendly for little fingers on a tablet, and to make a nice
ambient screensaver on a big screen.

## Play

Open **`index.html`** in any modern browser — that's the whole app. There is no
build step, no install, and no dependencies.

The live version is published to GitHub Pages automatically on every push to
`main` (see [Deploy](#deploy)).

## What you can do

- **Tap the water** to drop food where you tapped — fish swim over to eat.
- **Tap a fish** to toggle an info card showing its name, age, and traits.
- Use the toolbar along the bottom to:
  - 🍤 feed the fish
  - 🐟 add a fish &nbsp; 🦀 add a crab &nbsp; 🐴 add a seahorse
  - 🌱 plant seaweed
  - 🌗 change the day/night setting — cycle, always day, or always night
  - 📜 open the "what's new" changelog
  - 🔊 toggle sound

The toolbar and counter fade away after a few seconds of stillness
(screensaver mode) and reappear the moment you move the mouse or touch the
screen.

## Features

- Procedurally generated fish, each with its own colour, size, and traits
  (glow, bubbles, horns, whiskers, eyelashes, and more).
- Fish relationships, hunger, and the occasional predator.
- A full day/night cycle with a drifting sky, stars, and a moon after dark.
  Glowing fish shine brightest at night.
- Growing seaweed, crabs, seahorses, and a castle for fish to hide in.
- Gentle ambient sound (off until you turn it on).

## Deploy

Hosted on **GitHub Pages**. The workflow in
[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) uploads the repo
root and publishes it on every push to `main`. To deploy, just merge to `main`.

## Development

Everything lives in a single file, **`index.html`** — markup, CSS, and the
canvas-driven simulation in vanilla JavaScript. To work on it, open the file in
a browser and reload as you edit. See [`AGENTS.md`](AGENTS.md) for a tour of how
the code is organised.
