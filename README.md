# 💬 Catch Me Up

**Missed the group chat? Get the recap.**

Drop in a WhatsApp chat export and get a fun, structured recap: the plans that were made, what got decided, the drama, every time someone tagged you, and a to-do list of what you need to do now.

live link: https://catch-me-up.netlify.app/

## Features

- **Upload or paste**: WhatsApp `.txt` or `.zip` exports (Android and iPhone formats), or plain `Name: message` text.
- **Structured recap**: headline, TL;DR, vibe and chaos meter, mentions that need a reply, plans with time, place and status, decisions, drama with a spice level, best moment, and a personal to-do list.
- **Three recap styles**: Bestie 😌, Spicy 🌶️ and Just facts 📋.
- **Local stats**: top yappers, busiest hour, top emoji, how often you came up. Computed in the browser, no AI needed.
- **Copy or share**: the recap is formatted to paste straight back into WhatsApp.
- **Privacy first**: the chat is parsed in your browser and only the trimmed transcript is sent to the model. Nothing is stored.
- **Light and dark mode**, responsive down to small phones, keyboard accessible, and respects reduced-motion settings.

## Tech stack (intentionally minimal)

| Part | Choice |
| --- | --- |
| Frontend | One `index.html`: plain HTML, CSS and JavaScript, no build step |
| Backend | One serverless function: `api/recap.js` |
| AI | Google Gemini (free tier) or Anthropic Claude, chosen by which API key you set |
| Hosting | Vercel (free) |

No frameworks, no database, no npm dependencies.

## Deploy in 5 minutes

1. **Get an API key.** Either a free Gemini key from [Google AI Studio](https://aistudio.google.com/apikey) or an Anthropic key from the [Claude Console](https://console.anthropic.com/).
2. **Push to GitHub.**
   ```bash
   git init
   git add .
   git commit -m "Catch Me Up: GenAI group chat recaps"
   git branch -M main
   git remote add origin https://github.com/<you>/catch-me-up.git
   git push -u origin main
   ```
3. **Import on Vercel.** Go to [vercel.com/new](https://vercel.com/new), pick the repo, and leave the framework preset as **Other**.
4. **Add your key** under *Environment Variables*: `GEMINI_API_KEY` (or `ANTHROPIC_API_KEY`).
5. **Deploy.** Open the link and press *Try with a sample chat*.

To change the model, add a `MODEL` environment variable (for example `gemini-2.5-flash`). Model names change over time, so check your provider's docs if a default stops working.

### Run locally

```bash
npm i -g vercel
cp .env.example .env    # add your key
vercel dev              # opens http://localhost:3000
```

Opening `index.html` directly in a browser also works for the UI. The sample chat then shows a saved example recap, since there's no server to call.

## How it works

```
Browser                                         Serverless function            LLM
──────────────────────────────────────────      ────────────────────────       ─────────
1. Parse export (regex, multi-line, iOS/Android)
2. Detect date order (dd/mm vs mm/dd)
3. Filter system lines, deleted msgs, media
4. Keep the last N messages, cap at 60k chars
5. Compute stats locally                ──POST──▶ 6. Prompt with schema,  ──▶  7. JSON recap
                                                     tone, reader's name
9. Render recap, copy and share       ◀──JSON──── 8. Validate + normalize ◀──
```

Engineering decisions worth talking about in interviews:

- **Structured output, not free text.** The model returns JSON in a fixed schema, which the server validates and clamps (array lengths, string sizes, enum values) before the UI ever sees it.
- **Prompt-injection defense.** The transcript is wrapped in tags and the model is told to treat it as data. Chats often contain text like "ignore previous instructions".
- **Grounding rules.** The model must not invent times, places or quotes, and must say "not decided" instead of guessing.
- **Split work between code and AI.** Counting, emoji frequency, busiest hour and date parsing are done deterministically in JavaScript. The LLM only does what needs language understanding. That is cheaper, faster and always correct for the stats.
- **Privacy by design.** Parsing happens client-side, nothing is stored, and the API key never reaches the browser.
- **XSS safety.** Every model-generated string is HTML-escaped before rendering.

## Make it stand out further (ideas for v2)

- **Evaluation.** Hand-label 20 to 30 real or synthetic chats with the correct plans, decisions and mentions. Measure precision and recall per section and the hallucination rate, and compare models and prompts. Put the numbers in this README.
- **Long chats.** Map-reduce summarization: chunk chats of 5,000+ messages, summarize each chunk, then merge.
- **"Since I last spoke" mode.** Start the recap from your last message automatically.
- **Telegram or Discord bot** that recaps a group on command.
- **Tamil, Hindi or Hinglish recaps** for code-mixed chats.

## Resume bullets (replace with your own numbers)

- Built **Catch Me Up**, a GenAI web app that turns long WhatsApp group chats into structured recaps (plans, decisions, mentions, drama) using LLM structured outputs; deployed on Vercel with a serverless backend.
- Wrote a client-side parser for Android and iOS chat exports handling multi-line messages, locale date formats and system messages, keeping user chats private and out of storage.
- Hardened the LLM pipeline with schema validation, output normalization, prompt-injection guards and grounding rules; reached __% mention-detection recall on a hand-labeled set of __ chats.

## Project structure

```
catch-me-up/
├── index.html        # the whole frontend
├── api/recap.js      # serverless LLM endpoint
├── sample-chat.txt   # example WhatsApp export for testing
├── vercel.json       # function timeout
├── package.json
├── .env.example
└── README.md
```

## License

MIT
