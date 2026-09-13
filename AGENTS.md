# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Add durable project-specific notes here as they are discovered through real work.
- `chrome-devtools-axi` does not work for real measurement in this environment (open succeeds,
  snapshot/eval/screenshot fail or no-op). For real-browser QA (e.g. verifying `index.html`
  layout/overflow at specific viewports), use Chrome for Testing at
  `/home/leah/.cache/chrome-for-testing/chrome/<version>/chrome-linux64/chrome` via
  `puppeteer-core` (`npm install puppeteer-core` in a scratch dir; it is not preinstalled).
  The host has no system `libnss3`/`libnspr4`, so the chrome binary fails with
  `error while loading shared libraries: libnspr4.so`; fix with a userspace-only shim:
  `apt-get download libnss3 libnspr4 && dpkg-deb -x <pkg>.deb <shimdir>` for each, then launch
  puppeteer with `env: { LD_LIBRARY_PATH: '<shimdir>/usr/lib/x86_64-linux-gnu' }`. No root/sudo
  needed. A CSS grid item's default `min-width: auto` can force a grid track wider than its
  container at large mobile font sizes (a "grid blowout"); guard grid children with
  `min-width: 0` (see `.hero-grid > *` in `index.html`) and check for it with real per-viewport
  `scrollWidth` vs `clientWidth` measurement, not visual guesswork.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
