# MFP Muscle Protocol

A free, zero-subscription body composition tracking system. Pull your weekly MyFitnessPal
averages with one tap on iPhone, paste them into Claude, and your Notion tracker
auto-updates with compliance formulas calculated.

No Zapier. No paid APIs. No subscriptions beyond what you're probably already using.

---

## How it works

```
MyFitnessPal  ──sync──▶  Apple Health  ──Shortcut──▶  Clipboard
                                                           │
                                                           ▼
                                               Paste into Claude chat
                                                           │
                                                    SKILL.md loaded
                                                           │
                                                           ▼
                                              Notion tracker updated
                                           (compliance auto-calculated)
```

---

## What you need

| Requirement | Cost | Notes |
|-------------|------|-------|
| iPhone | — | iOS Shortcut runs on iPhone only |
| MyFitnessPal | Free | Log food daily |
| Apple Health | Free | Comes with iPhone; MFP syncs to it |
| Notion | Free | Personal plan is fine |
| Claude | Pro ($20/mo) | Needs Notion integration + skills |

---

## Part 1 — iOS Shortcut

Pulls your Sunday–Saturday weekly averages from Apple Health and copies them to your
clipboard in one tap.

**Output format:**
```
Cal: 2492 | Pro: 186g | Carbs: 215g | Fat: 105g
```

### Setup

**Prerequisites:**
1. MyFitnessPal installed and logging food daily
2. MFP → Apple Health sync enabled:
   - MFP app → More → Settings → Sharing & Privacy → HealthKit Sharing
   - Toggle ON: Calories In, Protein, Carbohydrates, Fat Total

**Install the Shortcut:**

> 📲 [Download iOS Shortcut — tap to install on iPhone](REPLACE_WITH_ICLOUD_LINK)

Prefer to build it yourself, or want to understand how the Sun–Sat window works?
See the precise step-by-step build in **[SHORTCUT_SETUP.md](./SHORTCUT_SETUP.md)**.

**Key technical detail:** iOS Shortcuts' "is between" filter for Health samples is exclusive
on both endpoints (undocumented behavior). The shortcut widens the date window by one day on
each side to compensate — full explanation in [SHORTCUT_SETUP.md](./SHORTCUT_SETUP.md).

---

## Part 2 — Claude Skill

Accepts the pasted macro data, determines the current week in your phase, asks for
exercise numbers, updates your Notion tracker, and shows a compliance summary.

### Setup

**Step 1: Connect Notion to Claude**
- Claude.ai → Settings → Integrations → Notion → Connect
- Authorize access to your workspace

**Step 2: Add the skill**

Download [`SKILL.md`](./SKILL.md) from this repo. In Claude.ai, open your project
(or create one), go to Settings → Skills, and upload it.

**Step 3: Run first-time setup**

In a Claude chat with the skill loaded, say:
> "Set up my muscle protocol tracker"

Claude will ask for your phase dates, macro goals, exercise goals, and Notion database
info. At the end it outputs a filled-in config block — paste it into your SKILL.md.

**Step 4: Create your Notion database** (if you don't have one)

Either let Claude create it automatically during setup, or follow the manual guide in
[`NOTION_SETUP.md`](./NOTION_SETUP.md).

---

## Weekly Workflow

Every Sunday (or whenever your shortcut data is ready):

1. **Run the iOS Shortcut** — tap it from the Shortcuts app or Home Screen widget
2. **Paste the output** into your Claude chat (the one with the skill loaded)
3. **Give exercise numbers** when Claude asks (lifts, runs, yoga, pilates, cold plunges)
4. **Done** — Notion updates, compliance reviewed

Total time per week: ~2 minutes.

---

## Compliance Rules

| Macro | Bulk threshold | Cut threshold | Direction |
|-------|---------------|--------------|-----------|
| Calories | ±12% | ±8% | Bidirectional |
| Protein | 85% floor | 85% floor | Over = always Hit |
| Carbs | ±20% | ±20% | Bidirectional |
| Fat | ±20% | ±20% | Bidirectional |

Exercise: 4+ lift sessions AND 2+ cardio/mobility sessions per week (runs, yoga,
pilates, cold plunges each count as 1).

All thresholds are configurable in your SKILL.md.

---

## Multi-Phase Support

The skill supports up to two phases (e.g. a 24-week Bulk followed by a 12-week Cut).
Each phase has its own:
- Macro targets
- Notion database
- Compliance thresholds

The skill auto-detects which phase is active based on today's date.

---

## Files in this repo

| File | What it is |
|------|-----------|
| `README.md` | This file — full system overview |
| `SKILL.md` | The Claude skill template — download and configure |
| `SHORTCUT_SETUP.md` | Precise step-by-step iOS Shortcut build guide (Part 1) |
| `NOTION_SETUP.md` | Manual Notion database setup guide |

---

## FAQ

**Does this cost anything?**
The shortcut itself is free. Claude Pro is ~$20/month. Notion free tier is sufficient.
No other subscriptions needed.

**Does it work on Android?**
Part 1 (iOS Shortcut) is iPhone only. Part 2 (Claude + Notion) works anywhere.
Android users could adapt Part 1 by building a similar automation with the Android
Health app and a shortcut tool like Tasker — then paste the output in the same format.

**Does MyFitnessPal Premium matter?**
No. You only need the free MFP tier. The system reads from Apple Health, not MFP directly.

**What if I log in MyFitnessPal but the Health data looks wrong?**
Verify MFP → Health sync is enabled (Settings → Sharing & Privacy → HealthKit Sharing).
Data only syncs for days logged after the connection was established.

**Can I track macros I enter manually in Apple Health instead of MFP?**
Yes. The shortcut reads Apple Health regardless of the source. Any app that writes
nutrition data to HealthKit works — MFP, Cronometer, MyMacros+, etc.

**What if I miss a week of logging?**
Tell Claude the week number and paste the data — it will fill in the correct row
regardless of the current date.

**Can I share the Notion database with someone else?**
Yes. Each person needs their own Claude setup (with the skill configured to their own
Notion DB), but you can share view-only access to someone's Notion tracker if you want.

**I want to add more activity types (bike rides, swimming, etc.)**
Add a Number column to your Notion database and tell Claude the column name. It will
include it in future updates. Update the Exercise Status Auto formula to count it.

---

## License

MIT — do whatever you want with it.

## Credits

Built by Dan with help from Claude (Anthropic).

The "is between" date boundary workaround for iOS Shortcuts was discovered through
extensive trial and error and is not documented by Apple.
