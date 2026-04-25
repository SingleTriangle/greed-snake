# Contributing to Greed Snake

Thank you for your interest in contributing! This document explains how to set up a development environment, the workflow for submitting changes, and the standards we follow.

---

## Development Setup

1. **Fork the repository** on GitHub.
2. **Clone your fork:**

```bash
git clone https://github.com/YOUR_USERNAME/greed-snake.git
cd greed-snake
```

3. **Start a local server** (required for Web Audio to work properly):

```bash
python3 -m http.server 8088
# or
npx serve .
```

4. Open `http://localhost:8088` in your browser.

---

## Workflow

We use a standard **Fork → Feature Branch → Pull Request** workflow:

1. Create a branch from `main`:

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/your-bug-fix-name
```

2. Make your changes. The entire game lives in `index.html` — no build step required.
3. **Test manually** in the browser (auto-playback restrictions mean you must interact with the page before sounds play).
4. Commit your changes with a clear message:

```bash
git add .
git commit -m "feat: add super-snake mode"
```

5. Push to your fork:

```bash
git push origin feat/your-feature-name
```

6. Open a **Pull Request** against `main` on the upstream repo.

---

## Code Style

Since this is a single-file vanilla JS project, we follow simple, consistent conventions:

- **Indentation**: 2 spaces
- **No semicolons**: ASI-friendly style (leading semicolons on IIFEs are fine)
- **Variable naming**: `camelCase` for variables and functions, `SCREAMING_SNAKE_CASE` for constants defined at the top of `index.html`
- **Comments**: Explain *why*, not *what*. Non-obvious logic (e.g. the direction buffer, self-collision truncation) must have a comment; straightforward code should be left clean
- **Section headers**: Use the format `// === SECTION NAME ===` to divide the file, matching the structure in `CODE_DOCUMENTATION.md`

---

## Areas for Contribution

### Good first issues

- Add a mute toggle for audio
- Add a pause-on-tab-switch feature (`visibilitychange` event)
- Add touch/swipe controls for mobile
- Add a dark/light theme toggle
- Pre-render the background grid to an offscreen canvas for performance

### Medium effort

- Responsive canvas sizing (scale to fit viewport)
- Add more power-up types (e.g. invincibility shield, reverse controls)
- Add sound on/off per event type
- Save replay data to `localStorage` and show a "last game" ghost snake

### Advanced

- WebAssembly port for deterministic simulation
- Multiplayer via WebSocket
- Procedural level generation beyond fixed obstacles

---

## Reporting Bugs

Please include the following in your bug report:

- Browser and version
- Steps to reproduce
- Expected vs. actual behavior
- Console errors (if any)
- Screenshot or screen recording if visual

---

## Questions?

Open an issue on GitHub or start a discussion. We aim to respond within a few days.
