# MFP Muscle Protocol

Track your body-comp progress for **free**. One tap on your iPhone grabs your weekly
MyFitnessPal averages → paste into Claude → your Notion tracker updates itself, compliance
and all.

No Zapier. No paid APIs. No spreadsheets. ~2 minutes a week.

```
MyFitnessPal → Apple Health → iPhone Shortcut → paste into Claude → Notion updates itself
```

---

## Your week (once it's set up)

1. **Tap the Shortcut** on your iPhone — it copies your week's macros.
2. **Paste into Claude.**
3. **Tell Claude your workouts** when it asks. Done.

That's the whole routine. Everything below is the one-time setup.

---

## Setup — about 10 minutes, once

You'll need an iPhone, MyFitnessPal (free), Notion (free), and Claude Pro. Do these two
steps once and you're set.

<details>
<summary><b>① Add the iPhone Shortcut</b> (2 min)</summary>

<br>

First, make sure MyFitnessPal is sending data to Apple Health:
**MFP app → More → Settings → Sharing & Privacy → HealthKit Sharing** → toggle ON
Calories, Protein, Carbohydrates, Fat.

Then tap to install:

> 📲 **[Install the iOS Shortcut](https://www.icloud.com/shortcuts/76a4640f53734668bb8712ecb7f2f7e9)**

That's it. Tapping the Shortcut copies your latest Sunday–Saturday averages, like:
`Cal: 2492 | Pro: 186g | Carbs: 215g | Fat: 105g`

*Want to build it by hand or see how it works? → [SHORTCUT_SETUP.md](./SHORTCUT_SETUP.md)*

</details>

<details>
<summary><b>② Add the Claude skill + your Notion tracker</b> (~8 min)</summary>

<br>

**a. Connect Notion to Claude** — Claude.ai → Settings → Connectors → Notion → Connect.

**b. Add the skill** — download **[`mfp-muscle-protocol.zip`](https://github.com/dannykogan/mfp-weekly-macros-shortcut/raw/main/mfp-muscle-protocol.zip)** (downloads directly, no GitHub account needed), then in Claude.ai go to **Settings → Capabilities → Skills → Upload skill** and pick the zip.

**c. Tell Claude to set you up** — in a chat with the skill on, say:

> "Set up my muscle protocol tracker"

Claude asks for your phase dates, macro goals, and workout goals, then builds your Notion
tracker for you (or connects to one you already have). One conversation and you're done.

*Prefer to build the Notion database by hand? → [NOTION_SETUP.md](./NOTION_SETUP.md)*

</details>

---

<details>
<summary>What you'll need (and what it costs)</summary>

<br>

| Requirement | Cost | Notes |
|-------------|------|-------|
| iPhone | — | The Shortcut is iPhone-only |
| MyFitnessPal | Free | Log food daily |
| Apple Health | Free | Comes with iPhone; MFP syncs to it |
| Notion | Free | Personal plan is fine |
| Claude | Pro (~$20/mo) | Needs the Notion connector + skills |

</details>

<details>
<summary>How Claude grades each week (compliance rules)</summary>

<br>

| Macro | Bulk | Cut | Direction |
|-------|------|-----|-----------|
| Calories | ±12% | ±8% | Bidirectional |
| Protein | 85% floor | 85% floor | Over = always a Hit |
| Carbs | ±20% | ±20% | Bidirectional |
| Fat | ±20% | ±20% | Bidirectional |

Workouts: 4+ lifts **and** 2+ cardio/mobility sessions per week (runs, yoga, pilates,
cold plunges each count as one). All of this is adjustable in your skill.

</details>

<details>
<summary>Running a Bulk then a Cut? (multi-phase)</summary>

<br>

The skill handles up to two phases — e.g. a 24-week Bulk followed by a 12-week Cut — each
with its own goals, Notion database, and thresholds. It figures out which phase you're in
from today's date automatically. Single-phase users just ignore this.

</details>

<details>
<summary>FAQ</summary>

<br>

**Does this cost anything?**
The Shortcut and Notion are free. Claude Pro is ~$20/month. Nothing else.

**Does it work on Android?**
The Shortcut is iPhone-only, but the Claude + Notion half works anywhere. Android users can
recreate the Shortcut with the Android Health app + Tasker and paste the same format.

**Do I need MyFitnessPal Premium?**
No — the free tier is fine. The system reads Apple Health, not MFP directly.

**My Health data looks wrong.**
Check MFP → Health sync (Settings → Sharing & Privacy → HealthKit Sharing). Data only syncs
for days logged *after* you turned the connection on.

**Can I use a different food app?**
Yes — anything that writes nutrition to Apple Health works (Cronometer, MyMacros+, etc.).

**I missed a week.**
Tell Claude the week number and paste the data — it fills the right row no matter the date.

**Can I share my tracker?**
Yes. Each person runs their own setup, but you can share view-only access to a Notion tracker.

**I want to track other activities (biking, swimming…).**
Add a column to your Notion database and tell Claude its name — it'll start including it.

</details>

<details>
<summary>What's in this repo</summary>

<br>

| File | What it is |
|------|-----------|
| `mfp-muscle-protocol.zip` | The skill, packaged for one-click upload to Claude.ai |
| `SKILL.md` | The skill source — same content, for reading/editing |
| `SHORTCUT_SETUP.md` | Step-by-step iOS Shortcut build guide |
| `NOTION_SETUP.md` | Manual Notion database setup guide |

</details>

---

MIT licensed — do whatever you want with it. Built by Dan with help from Claude.
The iOS "is between" date-filter workaround was found through a lot of trial and error and
isn't documented by Apple.
