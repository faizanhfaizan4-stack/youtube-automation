# GitHub Actions secrets for the YouTube pipeline

These values live in **GitHub repo → Settings → Secrets and variables → Actions → "New repository secret"**.
Never paste them in chat, never commit them to the repo. The workflow reads them
only as `${{ secrets.NAME }}` and maps them to environment variables.

## Required for a full generation run

| Secret name | What it is | Where to get it |
|---|---|---|
| `GEMINI_API_KEY` | Primary LLM key (free tier). The pipeline auto-fails-over through Groq → Cerebras → OpenRouter free models when quota hits. | https://aistudio.google.com/apikey |
| `YOUTUBE_TOKENS_JSON` | **base64** of your local `config/tokens.json`, created once by `npm run walkthrough` on any machine. Encodes the OAuth refresh token so the runner can upload. Generate with: `base64 -w0 config/tokens.json` | Your own machine, after one-time OAuth |
| `YOUTUBE_CLIENT_SECRETS_JSON` | **base64** of `config/client_secrets.json` (Google Cloud OAuth client). `base64 -w0 config/client_secrets.json` | Google Cloud Console → APIs & Services → Credentials → OAuth client ID |

You need at least **one** LLM key. `OPENAI_API_KEY` or `OPENROUTER_API_KEY` also
work as the primary; the workflow checks all three.

## Recommended (free tiers)

| Secret name | What it is | Where to get it |
|---|---|---|
| `GROQ_API_KEY` | Free-tier LLM failover | https://console.groq.com |
| `CEREBRAS_API_KEY` | Free-tier LLM failover | https://cloud.cerebras.ai |
| `PEXELS_API_KEY` | Free stock B-roll footage | https://www.pexels.com/api/ |
| `PIXABAY_API_KEY` | Free stock B-roll footage (backup source) | https://pixabay.com/api/docs/ |
| `DISCORD_WEBHOOK_URL` | Success/failure alerts for each run | Discord channel → Settings → Integrations → Webhooks |
| `AMAZON_AFFILIATE_TAG` | Your Amazon Associates tag, e.g. `yourtag-21` (auto-appends to Amazon.in links) | https://affiliate-program.amazon.in |
| `AFFILIATE_LINKS` | Extra affiliate URLs, comma-separated (first two go in the affiliate comment) | Your affiliate dashboards |
| `COMPETITOR_CHANNELS` | Comma-separated YouTube channel IDs for zero-quota trend discovery, e.g. `UCxxxx,UCyyyy` | Channel page URLs |

## If a secret is missing

- Missing **LLM key or YouTube OAuth** → the scheduled run degrades gracefully to
  `npm run pipeline:dry-run` (validates the environment, generates/uploads nothing).
  Fix by adding the secrets above.
- Missing **Pexels/Pixabay** → the pipeline warns and falls back to AI/local visuals.
- Missing **Discord webhook** → no chat alerts; the pipeline's own notifier is also silent.
- Missing **affiliate tag/links** → video still publishes, just without revenue links.

## Refreshing YouTube tokens

OAuth refresh tokens can be revoked or expire. If uploads start failing with auth
errors, re-run `npm run walkthrough` locally, re-encode `config/tokens.json` with
`base64 -w0`, and update the `YOUTUBE_TOKENS_JSON` secret. Nothing else changes.

## Free-tier budget

This repo is **public**, so GitHub Actions minutes are unlimited and free —
the twice-daily schedule costs nothing regardless of run length. (On a private
repo the allowance would be ~2,000 min/month: 2 runs/day × ~20–40 min × 30 days
≈ 1,200–2,400 min/month — too tight. Keep this repo public.)
