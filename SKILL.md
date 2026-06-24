---
name: ai-nutritionist
description: >
  AI Nutritionist skill. Trigger this skill when the user pastes weekly macro data
  in any format containing calorie, protein, carb, and fat numbers (especially the iOS
  Shortcut format "Cal: Xg | Pro: Xg | Carbs: Xg | Fat: Xg"). Also trigger on phrases
  like "update my tracker", "log this week", "weekly check-in", "update Notion",
  "MFP data", "my macros", "weekly macros", or any nutrition numbers pasted without
  context that look like a weekly fitness log. Also triggers if the user says "setup",
  "configure my tracker", or "I want to start tracking", or asks "how am I doing" /
  "show me my compliance" in a fitness context. This skill handles first-time onboarding,
  weekly macro + exercise logging to Notion, and compliance review.
---

# AI Nutritionist — Claude Skill

Paste your weekly MFP data → Claude logs it to Notion → compliance reviewed automatically.

This is Part 2 of the AI Nutritionist system. Part 1 (the iOS Shortcut that pulls
your weekly averages from Apple Health) lives at:
https://github.com/dannykogan/ai-nutritionist

---

## ⚙️ YOUR CONFIGURATION

Fill in this block when you first set up the skill. If any value still says {{PLACEHOLDER}},
run the SETUP FLOW first (just say "set up my tracker" in a Claude chat with this skill loaded).

```
# ── PHASE 1 ──────────────────────────────────────────────────────────────────
PHASE1_TYPE:              {{PHASE1_TYPE}}           # BULK or CUT
PHASE1_START:             {{PHASE1_START}}           # e.g. March 8, 2026
PHASE1_END:               {{PHASE1_END}}             # e.g. August 23, 2026
PHASE1_TOTAL_WEEKS:       {{PHASE1_TOTAL_WEEKS}}     # e.g. 24
PHASE1_GOAL_CALORIES:     {{PHASE1_GOAL_CALORIES}}   # e.g. 2400
PHASE1_GOAL_PROTEIN_G:    {{PHASE1_GOAL_PROTEIN_G}}  # e.g. 150
PHASE1_GOAL_CARBS_G:      {{PHASE1_GOAL_CARBS_G}}    # e.g. 215
PHASE1_GOAL_FAT_G:        {{PHASE1_GOAL_FAT_G}}      # e.g. 105
PHASE1_GOAL_LIFTS:        {{PHASE1_GOAL_LIFTS}}      # e.g. 4
PHASE1_GOAL_CARDIO:       {{PHASE1_GOAL_CARDIO}}     # e.g. 2  (runs+yoga+pilates+etc)
PHASE1_NOTION_DS_ID:      {{PHASE1_NOTION_DS_ID}}    # e.g. collection://d9a5cecc-...

# ── PHASE 2 (leave blank / delete if you only have one phase) ─────────────
PHASE2_TYPE:              {{PHASE2_TYPE}}
PHASE2_START:             {{PHASE2_START}}
PHASE2_END:               {{PHASE2_END}}
PHASE2_TOTAL_WEEKS:       {{PHASE2_TOTAL_WEEKS}}
PHASE2_GOAL_CALORIES:     {{PHASE2_GOAL_CALORIES}}
PHASE2_GOAL_PROTEIN_G:    {{PHASE2_GOAL_PROTEIN_G}}
PHASE2_GOAL_CARBS_G:      {{PHASE2_GOAL_CARBS_G}}
PHASE2_GOAL_FAT_G:        {{PHASE2_GOAL_FAT_G}}
PHASE2_GOAL_LIFTS:        {{PHASE2_GOAL_LIFTS}}
PHASE2_GOAL_CARDIO:       {{PHASE2_GOAL_CARDIO}}
PHASE2_NOTION_DS_ID:      {{PHASE2_NOTION_DS_ID}}
```

---

## 🔁 TRIGGER DETECTION

Every time this skill is triggered, Claude checks in this order:

1. **Unfilled config?** → Any value above still says `{{PLACEHOLDER}}` → run **SETUP FLOW**
2. **Macro paste?** → User pasted numbers that look like weekly nutrition data → run **WEEKLY LOG FLOW**
3. **Explicit setup request?** → "setup", "configure", "start fresh" → run **SETUP FLOW**
4. **Progress check?** → "how am I doing", "show compliance", "summary" → run **STATUS FLOW**

---

## 🛠️ SETUP FLOW

Trigger: any config value is still `{{PLACEHOLDER}}` OR user says "set up my tracker".

Walk through setup in three blocks. Do not fire all questions at once — ask each block and
wait for the response before moving on.

### Block 1 — Phase Info

> "Let's configure your AI Nutritionist tracker. First, the basics:
>
> 1. Are you in a **Bulk** (gaining) or **Cut** (losing) phase — or custom?
> 2. When does this phase **start**? (e.g. June 1, 2026)
> 3. When does it **end** — or how many weeks is it?
> 4. Do you have a **second phase** planned after this one? (e.g. cut after a bulk)
>    If yes, give me those dates and type too."

### Block 2 — Macro Goals

> "Your daily average macro targets:
>
> - Calories per day:
> - Protein (g) per day:
> - Carbs (g) per day:
> - Fat (g) per day:
>
> And if you have a second phase, the same for that one."

### Block 3 — Exercise Goals

> "Exercise goals per week:
>
> - Minimum lift sessions: (default: 4)
> - Minimum cardio/mobility sessions: (default: 2)
>   — counts any of: runs, yoga, pilates, cold plunges, bike rides, etc."

### Block 4 — Notion Setup

> "Do you already have a Notion database set up for this?
>
> - **Yes** → paste the Notion page URL here and I'll extract the database ID automatically.
> - **No** → I can create one for you right now (you'll need Notion connected to Claude).
> - **Not sure** → tell me your Notion page URL and I'll check."

**If user provides a Notion URL:** fetch the page using the Notion MCP, extract the
`collection://...` data source ID from the response, confirm it to the user.

**If user needs a new database:** run NOTION CREATE FLOW (see below). Ask for a
parent page URL to create the database under, or create at workspace level.

### Block 5 — Output Config

Once all info is collected, output the completed config block:

```
Here's your filled-in config. Replace the CONFIGURATION block in your SKILL.md with this:

PHASE1_TYPE:              [value]
PHASE1_START:             [value]
...etc
```

Then tell the user: "Save this to your SKILL.md and you're all set. Paste your MFP data
each week and I'll handle the rest."

---

## 📅 NOTION CREATE FLOW

Run when user needs a new Notion database created.

Use the Notion MCP `notion-create-database` tool with this schema:

```sql
CREATE TABLE (
  "Date Range" TITLE,
  "Week #" NUMBER,
  "Actual Cals" NUMBER,
  "Actual Protein (g)" NUMBER,
  "Actual Carbs (g)" NUMBER,
  "Actual Fat (g)" NUMBER,
  "Goal Cals" NUMBER,
  "Goal Protein (g)" NUMBER,
  "Goal Carbs (g)" NUMBER,
  "Goal Fat (g)" NUMBER,
  "Cal Status Auto" FORMULA('if(empty(prop("Actual Cals")), "", if(abs(prop("Actual Cals") - prop("Goal Cals")) / prop("Goal Cals") <= 0.12, "Hit ✅", "Missed 🔴"))'),
  "Protein Status Auto" FORMULA('if(empty(prop("Actual Protein (g)")), "", if(prop("Actual Protein (g)") >= prop("Goal Protein (g)") * 0.85, "Hit ✅", "Missed 🔴"))'),
  "Carbs Status Auto" FORMULA('if(empty(prop("Actual Carbs (g)")), "", if(abs(prop("Actual Carbs (g)") - prop("Goal Carbs (g)")) / prop("Goal Carbs (g)") <= 0.20, "Hit ✅", "Missed 🔴"))'),
  "Fat Status Auto" FORMULA('if(empty(prop("Actual Fat (g)")), "", if(abs(prop("Actual Fat (g)") - prop("Goal Fat (g)")) / prop("Goal Fat (g)") <= 0.20, "Hit ✅", "Missed 🔴"))'),
  "Lifts" NUMBER,
  "Runs" NUMBER,
  "Yoga" NUMBER,
  "Pilates" NUMBER,
  "Cold Plunges" NUMBER,
  "Exercise Status Auto" FORMULA('if(empty(prop("Lifts")), "", if(prop("Lifts") >= 4 and (prop("Runs") + prop("Yoga") + prop("Pilates") + prop("Cold Plunges")) >= 2, "Hit ✅", "Missed 🔴"))'),
  "Weight (lbs)" NUMBER,
  "Notes" RICH_TEXT
)
```

NOTE: The Exercise Status Auto formula uses 4 lifts and 2 cardio as hardcoded defaults.
After creating, the user can update these thresholds to match their GOAL_LIFTS and
GOAL_CARDIO values if different.

After creating the database, pre-populate all week rows:
- For each week 1 through TOTAL_WEEKS, create a page with:
  - `Date Range` = formatted date range (e.g. "Mar 8 – Mar 14")
  - `Week #` = week number
  - `Goal Cals` = PHASE_GOAL_CALORIES
  - `Goal Protein (g)` = PHASE_GOAL_PROTEIN_G
  - `Goal Carbs (g)` = PHASE_GOAL_CARBS_G
  - `Goal Fat (g)` = PHASE_GOAL_FAT_G

Then give the user the data source ID (`collection://...`) for the config.

---

## 📋 WEEKLY LOG FLOW

Triggered when: user pastes macro data.

### Step 1 — Parse Macro Data

Accept any of these input formats:

| Format | Example |
|--------|---------|
| iOS Shortcut output | `Cal: 2492g \| Pro: 186g \| Carbs: 215g \| Fat: 105g` |
| Without "g" on calories | `Cal: 2492 \| Pro: 186g \| Carbs: 215g \| Fat: 105g` |
| Pipe-separated raw | `2492 \| 186 \| 215 \| 105` |
| Natural language | "2400 calories, 150 protein, 210 carbs, 55 fat" |

**Important:** The iOS Shortcut outputs `Cal: Xg` with a "g" suffix even for calories
(calories are not in grams — this is a formatting quirk of the shortcut). Strip the "g"
and parse the integer.

Extract: `calories` (int), `protein_g` (int), `carbs_g` (int), `fat_g` (int).

### Step 2 — Determine Active Phase

Check today's date against PHASE1_START / PHASE1_END and PHASE2_START / PHASE2_END
to determine which phase is active and which Notion DB to use.

Then calculate:
```
week_num = floor((today - phase_start_date).days / 7) + 1
```

Cap at PHASE_TOTAL_WEEKS. If today is before phase start, ask which week to log.
If today is after phase end, ask if they're logging a past week.

### Step 3 — Query Notion for the Row

Use the Notion MCP to find the row where `Week # = week_num` in the active phase's DB.

If that row already has `Actual Cals` filled in:
> "Week [X] already has data (Cal: [existing]). Updating it, or logging a different week?"

### Step 4 — Ask for Exercise Numbers

ALWAYS ask — never skip this step:

> "Got Week [X] ([Date Range]). Exercise for the week:
>
> - Lifts:
> - Runs:
> - Yoga:
> - Pilates:
> - Cold Plunges:
> - Weight this Sunday (lbs)? (optional — skip if you didn't weigh in)
>
> Just give me the ones you did. Anything not mentioned defaults to 0."

### Step 5 — Update Notion

Use `notion-update-page` with the page ID from Step 3:

```json
{
  "Actual Cals": [calories],
  "Actual Protein (g)": [protein_g],
  "Actual Carbs (g)": [carbs_g],
  "Actual Fat (g)": [fat_g],
  "Lifts": [lifts or 0],
  "Runs": [runs or 0],
  "Yoga": [yoga or 0],
  "Pilates": [pilates or 0],
  "Cold Plunges": [cold_plunges or 0],
  "Weight (lbs)": [weight, only if provided]
}
```

### Step 6 — Compliance Summary

After Notion update, always show:

```
Week [X] logged ✅ ([Date Range])

─── Nutrition ─────────────────────────────────────────────
  Calories  [actual] / [goal]  → [✅ / 🔴]  ([+/-X%], threshold ±[threshold]%)
  Protein   [actual]g / [goal]g → [✅ / 🔴]  ([X%] of goal, floor is 85%)
  Carbs     [actual]g / [goal]g → [✅ / 🔴]  ([+/-X%], threshold ±20%)
  Fat       [actual]g / [goal]g → [✅ / 🔴]  ([+/-X%], threshold ±20%)

─── Exercise ──────────────────────────────────────────────
  Lifts     [N] sessions  → [✅ / 🔴]  (goal: [GOAL_LIFTS]+)
  Cardio    [N] sessions  → [✅ / 🔴]  (goal: [GOAL_CARDIO]+)

─── Overall: [N / 6 Hit] ──────────────────────────────────
[One-line assessment — see below]
```

**Assessment language:**
- 6/6: "Perfect week. Locked in."
- 5/6: "Solid. [Name the one miss]."
- 4/6: "Good base — tighten up [biggest gap] next week."
- 3/6: "Halfway there. [Identify pattern in misses]."
- 1–2/6: "Rough one. What got in the way?"
- 0/6: "Fresh start next week."

Be direct. One or two sentences max. No cheerleading.

---

## 📊 STATUS FLOW

Triggered by: "how am I doing", "show me my progress", "compliance summary", "review".

1. Use Notion MCP to query all rows where `Actual Cals` is not empty (filled weeks only).
2. Compute:
   - Weeks hit per macro (using the thresholds below)
   - Exercise hit rate
   - Weight trend (first entry vs most recent, if available)
   - Current week number and % through the phase
3. Show:

```
Phase [1/2] — [TYPE] — Week [current] of [total] ([X]% complete)
[PHASE_START] → [PHASE_END]

─── Compliance to date ─────────────────────────────────────
  Calories  [N]/[total] Hit  ([X]%)
  Protein   [N]/[total] Hit  ([X]%)
  Carbs     [N]/[total] Hit  ([X]%)
  Fat       [N]/[total] Hit  ([X]%)
  Exercise  [N]/[total] Hit  ([X]%)

─── Weight ─────────────────────────────────────────────────
  [Start weight] → [Latest weight]  ([+/-X lbs] over [N] weeks)

─── Assessment ─────────────────────────────────────────────
[2-3 sentence honest read of what's working, what isn't, what to focus on.]
```

---

## 📐 COMPLIANCE THRESHOLDS

These are the rules used to compute Hit ✅ vs Missed 🔴:

| Macro | BULK threshold | CUT threshold | Direction |
|-------|---------------|--------------|-----------|
| Calories | ±12% | ±8% | Bidirectional |
| Protein | 85% floor | 85% floor | One-directional (over = always Hit) |
| Carbs | ±20% | ±20% | Bidirectional |
| Fat | ±20% | ±20% | Bidirectional |

Exercise compliance: GOAL_LIFTS+ lift sessions AND GOAL_CARDIO+ combined
cardio/mobility sessions (runs + yoga + pilates + cold plunges each count as 1).

---

## 🗂️ NOTION DATABASE SCHEMA

These are the column names this skill expects. If your database uses different names,
tell Claude once ("my protein column is called 'Prot (g)'") and it will adapt.

| Column | Type | Purpose |
|--------|------|---------|
| `Date Range` | Title | Week label, e.g. "Mar 8 – Mar 14" |
| `Week #` | Number | Week number within the phase (1, 2, 3…) |
| `Actual Cals` | Number | Avg daily calories that week |
| `Actual Protein (g)` | Number | Avg daily protein (g) |
| `Actual Carbs (g)` | Number | Avg daily carbs (g) |
| `Actual Fat (g)` | Number | Avg daily fat (g) |
| `Goal Cals` | Number | Target daily calories |
| `Goal Protein (g)` | Number | Target daily protein (g) |
| `Goal Carbs (g)` | Number | Target daily carbs (g) |
| `Goal Fat (g)` | Number | Target daily fat (g) |
| `Cal Status Auto` | Formula | Auto Hit/Missed for calories |
| `Protein Status Auto` | Formula | Auto Hit/Missed for protein |
| `Carbs Status Auto` | Formula | Auto Hit/Missed for carbs |
| `Fat Status Auto` | Formula | Auto Hit/Missed for fat |
| `Lifts` | Number | Lift sessions that week |
| `Runs` | Number | Run sessions |
| `Yoga` | Number | Yoga sessions |
| `Pilates` | Number | Pilates sessions |
| `Cold Plunges` | Number | Cold plunge sessions |
| `Exercise Status Auto` | Formula | Auto Hit/Missed for exercise |
| `Weight (lbs)` | Number | Fasted Sunday morning weight |
| `Notes` | Rich Text | Free-form notes |

---

## 🔢 NOTION FORMULA REFERENCE

Copy-paste these into Notion formula fields. Adjust the thresholds in brackets if needed.

**Cal Status Auto** (change 0.12 → 0.08 for Cut phases):
```
if(empty(prop("Actual Cals")), "", if(abs(prop("Actual Cals") - prop("Goal Cals")) / prop("Goal Cals") <= 0.12, "Hit ✅", "Missed 🔴"))
```

**Protein Status Auto** (floor only — hitting over goal is always a win):
```
if(empty(prop("Actual Protein (g)")), "", if(prop("Actual Protein (g)") >= prop("Goal Protein (g)") * 0.85, "Hit ✅", "Missed 🔴"))
```

**Carbs Status Auto**:
```
if(empty(prop("Actual Carbs (g)")), "", if(abs(prop("Actual Carbs (g)") - prop("Goal Carbs (g)")) / prop("Goal Carbs (g)") <= 0.20, "Hit ✅", "Missed 🔴"))
```

**Fat Status Auto**:
```
if(empty(prop("Actual Fat (g)")), "", if(abs(prop("Actual Fat (g)") - prop("Goal Fat (g)")) / prop("Goal Fat (g)") <= 0.20, "Hit ✅", "Missed 🔴"))
```

**Exercise Status Auto** (adjust the 4 and 2 to match your goals):
```
if(empty(prop("Lifts")), "", if(prop("Lifts") >= 4 and (prop("Runs") + prop("Yoga") + prop("Pilates") + prop("Cold Plunges")) >= 2, "Hit ✅", "Missed 🔴"))
```

**Known Notion quirk:** Number columns that default to 0 when empty (exercise fields like
Lifts) need a sum-based blank check. Columns that are truly null when empty (actual macro
fields) use `empty()` directly. Using `== 0` on null-type columns breaks formulas.

---

## ❌ WHAT THIS SKILL DOES NOT DO

- Read MyFitnessPal or Apple Health directly — you paste the output from the iOS Shortcut
- Log food, set calories, or modify MFP in any way
- Work without Notion connected to Claude (requires the Notion MCP integration)
- Replace a registered dietitian or medical professional
- Track body fat percentage (use a DEXA scan for that)

---

## 🔧 TROUBLESHOOTING

**"Claude didn't trigger the skill"**
Make sure you've added this SKILL.md to your Claude project under Settings → Skills.

**"Claude updated the wrong week"**
Say "that was Week [X], not Week [Y]" — Claude will update the correct row.

**"My Notion columns have different names"**
Just tell Claude once: "my protein column is called X" — it adapts for that session.

**"The formula columns show errors"**
See the FORMULA REFERENCE above. The most common issue is `== 0` vs `empty()` — 
make sure null-type columns (actual macros) use `empty()`, not `== 0`.

**"I want to change my macro goals mid-phase"**
Update the config values in this SKILL.md and tell Claude: "my goals changed to X/Y/Z/W".
It will use the new goals going forward but won't retroactively change old week rows.
