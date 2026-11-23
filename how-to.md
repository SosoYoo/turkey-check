# Turkey Check: moving to Claude Code and pushing to GitHub

This guide gets the Turkey Check project running in Claude Code on a Windows PC, so Claude can edit the files, test them locally and push straight to GitHub. No more downloading and re-uploading `index.html`.

The commands below are the standard ones, but installers change. If something doesn't match what you see, check the official install page at https://code.claude.com/docs/en/setup.

## What's in the project

| File | What it is |
|---|---|
| `index.html` | The whole app: page, styles and logic in one file. GitHub Pages serves it. |
| `worker.js` | Optional Cloudflare Worker that holds your Claude key and enforces the free-trial limits. |
| `false.mp3`, `true.mp3` | Optional custom sounds. The app falls back to built-in sounds if they're missing. |
| `how-to.md` | This file. |

Live site: `https://sosoyoo.github.io/turkey-check/`
Repo: `https://github.com/SosoYoo/turkey-check`

## 1. One-time setup

### Install Git and the GitHub CLI
Open PowerShell and run:

```powershell
winget install --id Git.Git -e
winget install --id GitHub.cli -e
```

Close PowerShell and open a new window so the new commands are found. Then tell Git who you are:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Use the email tied to your GitHub account, or the private `noreply` address from GitHub's email settings if you'd rather not expose your real one.

### Log in to GitHub
```powershell
gh auth login
```
Choose **GitHub.com**, then **HTTPS**, then **Login with a web browser**, and follow the prompts. This is what lets Claude push for you.

### Install Claude Code
You need a Claude Pro, Max, Team or Enterprise plan, or a Console account. The free claude.ai plan doesn't include Claude Code.

```powershell
irm https://claude.ai/install.ps1 | iex
```

Open a new terminal and check it worked:

```powershell
claude --version
```

## 2. Get the project onto your computer

```powershell
cd C:\Users\<you>\Documents
gh repo clone SosoYoo/turkey-check
cd turkey-check
```

Copy your newest `index.html` (and `worker.js`, `false.mp3`, `true.mp3`) into this folder, replacing the old ones. Then check what changed:

```powershell
git status
```

## 3. Add a CLAUDE.md so Claude knows the project

Create a file named `CLAUDE.md` in the folder with the text below. Claude Code reads it at the start of every session.

```markdown
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
```

## 4. Everyday workflow

1. In the project folder, start Claude Code:
   ```powershell
   claude
   ```
2. Ask for a change in plain words, for example: *"Make the false sound louder and bump the version."*
3. Test it locally. In a second terminal window in the same folder:
   ```powershell
   py -m http.server 8000
   ```
   Then open `http://localhost:8000` in Chrome. Paste your API key into the app's settings the first time, since each address has its own settings.
4. When it looks right, ask Claude: *"Commit this and push to GitHub."* It will run roughly:
   ```powershell
   git add -A
   git commit -m "Describe the change"
   git push
   ```
5. Wait 1 to 2 minutes, then open the live site. Check that the footer shows the new version. If it doesn't, close the tab and reopen it, since browsers cache the old copy.

Claude Code asks before running commands. Approving `git` commands as you go is fine. Always glance at what's about to be committed.

## 5. Keep secrets out of the repo

The repo is public, so anything committed is visible to everyone, including bots that scan for API keys.

- Don't paste your key into any file in the folder.
- If you ever commit a key by mistake, **revoke it in the Anthropic Console right away** and make a new one. Deleting the file later doesn't remove it from Git history.
- Add a `.gitignore` file with these lines as a safety net:
  ```
  .env
  *.key
  .dev.vars
  .wrangler/
  ```

## 6. Optional: the free-trial Worker

This lets visitors use the app without their own key. You pay, within the limits you set.

### One-time setup
```powershell
npm install -g wrangler
wrangler login
wrangler kv namespace create COUNTS
```
You need Node.js installed (`winget install --id OpenJS.NodeJS.LTS -e`). The last command prints an `id`. Put it into a file named `wrangler.toml` in the project folder:

```toml
name = "turkey-check"
main = "worker.js"
compatibility_date = "2025-01-01"

[[kv_namespaces]]
binding = "COUNTS"
id = "PASTE_THE_ID_HERE"

[vars]
PER_DEVICE_DAILY = "15"
GLOBAL_DAILY = "150"
```

Store your key as a secret. It prompts you to paste it and never writes it to a file:
```powershell
wrangler secret put ANTHROPIC_KEY
```
Use a fresh key made just for this Worker.

### Deploy and connect
```powershell
wrangler deploy
```
The output shows the Worker's URL, like `https://turkey-check.<name>.workers.dev`. Ask Claude Code: *"Set PROXY_URL in index.html to this URL, bump the version, and push."*

Later, to change the prompt or limits, edit `worker.js` or `wrangler.toml` and run `wrangler deploy` again. If the system prompt in `index.html` changes, update the copy in `worker.js` too, since they must match.

### Protect your budget
- In the Anthropic Console, set a **workspace spend limit** and keep **auto-reload off**.
- Rough cost is 1.5 to 2 cents per check, so `GLOBAL_DAILY = 150` caps the worst case at about $3 a day.
- To shut the free trial off immediately, delete the Worker, or remove the `ANTHROPIC_KEY` secret.

## 7. Troubleshooting

| Problem | Fix |
|---|---|
| `claude` isn't recognized | Open a new terminal. Run `claude doctor`. |
| `git push` asks for a password | Run `gh auth login` again, and choose HTTPS. |
| Chrome asks for the mic every time | You're opening the file directly. Use `localhost:8000` or the GitHub site. |
| Site still shows an old version | Wait 2 minutes, then close and reopen the tab. Check the **Actions** tab in the repo for a green check. |
| Custom sound isn't used | Open the app's settings and read the status line under the test buttons. It lists what it tried. |
| "Add your API key first" | The address has no key saved. Paste yours into the app's settings. |

## Quick checklist

- [ ] Git, GitHub CLI and Claude Code installed
- [ ] `gh auth login` done
- [ ] Repo cloned and your newest files copied in
- [ ] `CLAUDE.md` and `.gitignore` added
- [ ] Local test at `localhost:8000` works
- [ ] First push succeeded and the footer version updated on the live site
- [ ] (Optional) Worker deployed, spend limit set
