# Turkey Check

A live fact-checker web app, hosted on GitHub Pages (repo SosoYoo/turkey-check).

## Architecture
- `index.html` is the entire app (HTML, CSS and JS in one file). No build step.
- Interaction: press-and-hold the mic button, speak, let go. The speech goes to a fact-check call and the result appears as a card (true / false / misleading / unverified / not fact-checkable).
- Speech recognition: the browser's built-in Web Speech API (Chrome).
- Fact-checking: Claude API with web search, called directly from the browser using the user's own key. If `PROXY_URL` is set, a free-trial mode calls the Cloudflare Worker (`worker.js`) instead.
- Sounds: `false.mp3` / `true.mp3` next to `index.html` if present, otherwise built-in synthesized sounds.
- Settings and keys live in the browser's localStorage, per address.

## Rules
- Never commit an API key or any secret. Keys are typed into the app or stored as Cloudflare secrets.
- After changing `index.html`, bump the version number in the footer ("Turkey Check vNN").
- Before finishing a change, check that the script has no syntax errors and that every element id the code uses exists in the HTML.
- Test locally with `py -m http.server 8000`, then open http://localhost:8000 (opening the file directly makes Chrome ask for the mic every time).
- Keep explanations short and the UI simple. This is used at family dinners.
