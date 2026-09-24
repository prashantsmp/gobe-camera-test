# GoBe Camera Test

A single-page browser diagnostic that checks the camera, screen, touch input and internet connection on a GoBe device. The tester opens the page on the device, runs the tests, saves a text report and uploads it to GitHub.

- **Live page:** https://prashantsmp.github.io/gobe-camera-test/
- **Repo:** https://github.com/prashantsmp/gobe-camera-test (public; GitHub Pages serves `main` from `/`)
- **Uploaded reports:** https://github.com/prashantsmp/gobe-camera-test/tree/main/reports

## Stack deviation (intentional)

This project is **one self-contained static HTML file**: inline CSS and JS, no framework, no build step and no backend. It deliberately does not follow the DAK defaults (Vue, FastAPI, the design system, PROJECT.md or CONVENTIONS.md, Docker).

**Why:** it's a throwaway field-test tool. It has to load on any device from a single URL, and a static page on GitHub Pages is the simplest way to do that. GitHub Pages also serves it over HTTPS, which browsers require before they allow camera access (`getUserMedia`).

Keep it as a single file unless the user explicitly asks to change the approach.

## Files

| Location | File | Notes |
|---|---|---|
| This folder (local, **not** a git repo) | `gobe-camera-test.html` | Source of truth for editing |
| GitHub repo | `index.html` | A copy of `gobe-camera-test.html` |
| GitHub repo | `reports/README.md` | Placeholder file. GitHub needs at least one file in a folder before its upload page will open there |
| GitHub repo | `reports/*.txt` | Uploaded test reports |

## Page structure (`gobe-camera-test.html`, ~440 lines)

The UI panels are Camera controls, Last photo, Touch test, Results, Log and Report. Large buttons (72px tall) make the page easy to use on a touch screen, and the styling is a dark theme.

The script sections are marked with `// ---------- N. name ----------` comments:

1. **environment:** browser and screen information
2. **cameras:** list the cameras, then start and stop one (resolution picker, mirror toggle)
3. **take photo:** full-frame and cropped JPEG captures, each with Save and Open links
4. **resolution sweep:** tries each resolution in turn
5. **restart stress test:** restarts the camera 10 times to check that it releases cleanly
6. **internet:** `no-cors` fetches to `gstatic.com/generate_204` and `cloudflare.com/cdn-cgi/trace`, with a 6-second timeout
7. **touch:** multi-finger tap pad
8. **report:** Copy report, Save report .txt (the filename is timestamped, e.g. `gobe-test-report-<ISO>.txt`), Upload report and View uploaded reports

## Report upload flow

**Upload report** is a plain link to `https://github.com/prashantsmp/gobe-camera-test/upload/main/reports`, which is GitHub's own upload page. The tester chooses the saved `.txt` file and taps **Commit changes**. This requires a GitHub login with write access to the repo.

- **No tokens:** never put a GitHub token in the page. The page is public, so anyone could read the token and use it to write to the repo.
- **If testers without GitHub access need to upload,** the agreed alternatives are:
  - a fine-grained token (write access to this repo's contents only) that each tester enters once and that is stored in `localStorage`;
  - a Cloudflare Worker relay that holds the token.

  Ask the user before building either one.

## Deploying a change

1. Edit `gobe-camera-test.html` in this folder.
2. Clone the repo to the scratchpad, copy the file over as `index.html`, then commit and push to `main`:
   ```bash
   gh repo clone prashantsmp/gobe-camera-test <scratchpad>/gobe-camera-test
   cp gobe-camera-test.html <scratchpad>/gobe-camera-test/index.html
   # commit, then push
   ```
   The scratchpad is cleared between sessions, so clone fresh each time.
3. GitHub Pages rebuilds in about a minute. To check that the change is live, run `curl` against the live page with a cache-busting query string (for example `?v=1`) and grep for the new markup.
4. Keep the local file and the repo's `index.html` identical.

## Conventions

- **Commits:** conventional commit messages (`feat:`, `fix:` and so on).
- **Asking first:** publishing and pushing are outward-facing, so confirm with the user before any new repo, visibility change or anything else that goes beyond a routine update.
- **Testing:** there is no automated test suite. To verify a change, open the live page in a browser (`open <url>`); the camera works only over HTTPS or on `localhost`.
