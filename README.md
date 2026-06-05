# Quest Board

A three-part system for tabletop RPG campaigns:

1. **Discord Bot** (`bot.py`) – reads the **#quest-board** channel with `!quests`,
   parses raw text via **Anthropic API** into structured JSON, and commits the
   result to `quests.json` via **GitHub Contents API**.
2. **GitHub Pages** (`index.html`) – loads `quests.json` and displays quests as
   fantasy parchment cards (dark theme, Cinzel / Crimson Pro), with filters by
   length and badges for level & duration.
3. **`.env`** – all secrets and configuration in one place.

After each successful commit, the bot posts the website URL to the channel.

```
Discord #quest-board  ──!quests──▶  bot.py  ──Anthropic──▶  JSON
                                       │
                                       └──GitHub API──▶ quests.json ──▶ GitHub Pages (index.html)
```

---

## Files

| File               | Purpose                                          |
|--------------------|--------------------------------------------------|
| `bot.py`           | The Discord bot (run locally)                    |
| `requirements.txt` | Python dependencies                              |
| `.env.example`     | Template for your configuration                  |
| `index.html`       | The website (hosted on GitHub Pages)             |
| `quests.json`      | Data source (overwritten by the bot)             |
| `.gitignore`       | Protects `.env` from accidental commits          |

---

## Requirements

- **Python 3.10+** (for `|` type hints in the code)
- A **Discord server** where you can manage roles & channels
- An **Anthropic account** with API credits
- A **GitHub account**

---

## Step 1 – GitHub Repository & Pages

1. Create a **new public repository** (e.g., `quest-board`).
   > GitHub Pages is free only for public repos. Private repos require GitHub Pro.
2. Upload `index.html` and `quests.json` to the **root** of the repo
   (via web upload or `git push`). `bot.py`, `.env`, etc. should **not** be in the repo.
3. Enable Pages: **Settings → Pages → Build and deployment**
   → *Source:* **Deploy from a branch** → *Branch:* `main` / `(root)` → **Save**.
4. After ~1 minute, GitHub shows the URL at the top, typically:
   ```
   https://<your-user>.github.io/<your-repo>/
   ```
   Open it – you should see the two sample quests.

> The bot automatically derives this URL from `GITHUB_REPO`. If your URL differs
> (e.g., custom domain), set it in `.env` under `SITE_URL`.

---

## Step 2 – GitHub Personal Access Token

The bot needs write access to `quests.json`.

**Recommended (Fine-grained Token):**

1. **GitHub → Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**
2. *Repository access:* **Only select repositories** → select your quest repo.
3. *Permissions → Repository permissions → Contents:* **Read and write**.
4. Generate the token and **copy immediately** → paste in `.env` as `GITHUB_TOKEN`.

*(Alternatively, use a classic token with the `repo` scope.)*

---

## Step 3 – Anthropic API Key

1. Log in to **console.anthropic.com**.
2. **Settings → API Keys → Create Key**.
3. Copy the key → paste in `.env` as `ANTHROPIC_API_KEY`.

> Default model is `claude-sonnet-4-20250514`. Change it via `ANTHROPIC_MODEL`
> in `.env`.

---

## Step 4 – Set Up Discord Bot

### 4a. Create Application & Bot

1. **discord.com/developers/applications → New Application** → enter a name.
2. Left sidebar **Bot** → **Reset Token** → copy token → paste in `.env` as `DISCORD_TOKEN`.
3. Under **Privileged Gateway Intents**, enable:
   - ✅ **Message Content Intent** (required – bot needs to read message text)
   - ✅ **Server Members Intent** (for role verification)

### 4b. Invite Bot to Your Server

1. Left sidebar **OAuth2 → URL Generator**.
2. *Scopes:* ✅ `bot`
3. *Bot Permissions:* ✅ `Read Messages/View Channels`, ✅ `Read Message History`,
   ✅ `Send Messages`, ✅ `Embed Links`
4. Open the generated URL → invite bot to your server.

### 4c. Get IDs & Set Role

1. In Discord, enable **User Settings → Advanced → Developer Mode**.
2. Right-click **#quest-board → Copy Channel ID** → paste in `.env` as
   `QUEST_CHANNEL_ID`.
3. Create (or choose) a role that can run `!quests`, and enter its **exact name**
   in `.env` as `ADMIN_ROLE` (default: `GameMaster`). Assign this role to yourself.

---

## Step 5 – Set Up & Run Bot Locally

```bash
# Change to project folder
cd unseen-servant

# Virtual environment (recommended)
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up configuration
cp .env.example .env               # Windows: copy .env.example .env
#  -> open .env in an editor and fill in all values

# Start bot
python bot.py
```

When you see `Logged in as …` in the terminal, the bot is running. It must be
running for `!quests` to work.

---

## Usage

1. Players / GM post quest descriptions to **#quest-board**
   (one quest per message works best).
2. A member with the admin role types **`!quests`**.
3. The bot reports its progress (reading → parsing → committing) and posts
   an embed with **website URL** and **commit link**.
4. GitHub Pages rebuilds (~1–2 min), then quests go live.
   The site auto-bypasses browser cache.

---

## Data Format (`quests.json`)

```json
{
  "generated_at": "2026-01-01T12:00:00+00:00",
  "count": 1,
  "quests": [
    {
      "title": "Road to Drakkenheim",
      "level": "5-9",
      "length": "Short",
      "description": "…",
      "reward": "",
      "hooks": [{ "text": "…" }, { "text": "…" }],
      "characters": ["Sylvara the Amethyst Dragon", "…"],
      "progression": ["Unmaking ++", "Divine Influence ++"],
      "notes": "This adventure unlocks 2 additional quests.",
      "tags": ["Ruins", "Dragon", "…"]
    }
  ]
}
```

Field reference:

| Field         | Content |
|---------------|---------|
| `title`       | Quest title |
| `level`       | Recommended character level, often a range like `5-9` |
| `length`      | Always `Short` / `Medium` / `Long` (normalized for filters) |
| `description` | Quest description, close to original |
| `reward`      | Reward – **only if explicitly stated**, else `""` (won't display) |
| `hooks`       | Array of quest paths/options as objects with `text` field; **can be empty** |
| `characters`  | Array of important NPCs, factions, quest givers |
| `progression` | Campaign states like `"Unmaking ++"`, `"Divine Influence --"`; shown as green ▲ / red ▼ in frontend |
| `notes`       | Meta notes (P.S., story branching, "unlocks …") or empty string |
| `tags`        | Array of 3–6 thematic keywords **derived by the model** (not in original) |

Empty fields (empty string `""` or empty array `[]`) are normalized by the bot.
Duplicate identical quests get merged into one entry during parsing.

---

## Configuration (`.env`)

| Variable            | Required | Description                                        |
|---------------------|:--------:|---------------------------------------------------|
| `DISCORD_TOKEN`     |    ✅    | Bot token from Developer Portal                   |
| `ANTHROPIC_API_KEY` |    ✅    | API key from console.anthropic.com                |
| `GITHUB_TOKEN`      |    ✅    | PAT with *Contents: Read and write*               |
| `GITHUB_REPO`       |    ✅    | Target in format `user/repo`                      |
| `GITHUB_FILE_PATH`  |          | Target file (default: `quests.json`)              |
| `QUEST_CHANNEL_ID`  |    ✅    | ID of #quest-board                                |
| `ADMIN_ROLE`        |          | Role that can run `!quests` (default: `GameMaster`) |
| `GITHUB_BRANCH`     |          | Branch to commit to (default: `main`)             |
| `ANTHROPIC_MODEL`   |          | Model (default: `claude-sonnet-4-20250514`)       |
| `MESSAGE_LIMIT`     |          | Number of channel messages to read (default: `100`) |
| `SITE_URL`          |          | Only set if Pages URL differs from standard       |

---

## Troubleshooting

| Problem | Cause / Solution |
|---|---|
| Bot doesn't respond to `!quests` | **Message Content Intent** enabled in portal? Bot online? Right channel? |
| "This command is reserved for role …" | Your account lacks the `ADMIN_ROLE`, or the name in `.env` doesn't match exactly. |
| "Quest channel not found" | Wrong `QUEST_CHANNEL_ID` or bot can't see the channel (permissions). |
| GitHub error `404` | Wrong `GITHUB_REPO`/`GITHUB_FILE_PATH`, or token lacks repo access. |
| GitHub error `401/403` | Token expired or missing *Contents: write*. |
| Website stays empty | Pages still building (1–2 min wait), or `quests.json` not in repo root. |
| `Missing environment variables …` | `.env` incomplete – compare with `.env.example`. |

---

## Security

- `.env` contains secrets and is in `.gitignore` – **never commit it**.
- Give `!quests` only to trusted roles, since each call costs API credits and
  modifies the repo.
- Use a fine-grained token with access **only** to this one repo.
