# Notion Database Setup Guide

This guide walks you through setting up the Notion tracking database for the MFP Muscle
Protocol. You can either let Claude create it automatically (recommended) or build it manually.

---

## Option A — Automatic Setup (Recommended)

If you have Notion connected to Claude, just say "set up my tracker" after loading the
SKILL.md. Claude will walk you through configuration and create the database with all
columns, formulas, and week rows pre-populated.

Requirements:
- Claude Pro or Team subscription
- Notion connected via the Notion integration in Claude settings

---

## Option B — Manual Setup

### Step 1: Create a new database in Notion

1. Open Notion and navigate to where you want the tracker
2. Type `/database` → select "Table — Full page"
3. Name it something like `🏋️ Phase 1 BULK Tracker (Mar 8 – Aug 23, 2026)`

---

### Step 2: Add columns in this order

Delete the default columns and add these:

| Column Name | Type | Notes |
|-------------|------|-------|
| `Date Range` | Title | Already exists — just rename it |
| `Week #` | Number | Week number within the phase |
| `Actual Cals` | Number | Format: Number |
| `Actual Protein (g)` | Number | |
| `Actual Carbs (g)` | Number | |
| `Actual Fat (g)` | Number | |
| `Goal Cals` | Number | Your daily calorie target |
| `Goal Protein (g)` | Number | Your daily protein target |
| `Goal Carbs (g)` | Number | Your daily carbs target |
| `Goal Fat (g)` | Number | Your daily fat target |
| `Cal Status Auto` | Formula | See formulas below |
| `Protein Status Auto` | Formula | See formulas below |
| `Carbs Status Auto` | Formula | See formulas below |
| `Fat Status Auto` | Formula | See formulas below |
| `Lifts` | Number | Lift sessions per week |
| `Runs` | Number | |
| `Yoga` | Number | |
| `Pilates` | Number | |
| `Cold Plunges` | Number | Or swap for any other activity |
| `Exercise Status Auto` | Formula | See formulas below |
| `Weight (lbs)` | Number | Optional — Sunday fasted weigh-in |
| `Notes` | Text | Free-form notes |

---

### Step 3: Add the formulas

Click the formula column → Edit formula → paste the expression:

**Cal Status Auto** (±12% for Bulk, change to 0.08 for Cut):
```
if(empty(prop("Actual Cals")), "", if(abs(prop("Actual Cals") - prop("Goal Cals")) / prop("Goal Cals") <= 0.12, "Hit ✅", "Missed 🔴"))
```

**Protein Status Auto** (floor only — over is always a Hit):
```
if(empty(prop("Actual Protein (g)")), "", if(prop("Actual Protein (g)") >= prop("Goal Protein (g)") * 0.85, "Hit ✅", "Missed 🔴"))
```

**Carbs Status Auto** (±20%):
```
if(empty(prop("Actual Carbs (g)")), "", if(abs(prop("Actual Carbs (g)") - prop("Goal Carbs (g)")) / prop("Goal Carbs (g)") <= 0.20, "Hit ✅", "Missed 🔴"))
```

**Fat Status Auto** (±20%):
```
if(empty(prop("Actual Fat (g)")), "", if(abs(prop("Actual Fat (g)") - prop("Goal Fat (g)")) / prop("Goal Fat (g)") <= 0.20, "Hit ✅", "Missed 🔴"))
```

**Exercise Status Auto** (4+ lifts AND 2+ cardio/mobility — adjust numbers to match your goals):
```
if(empty(prop("Lifts")), "", if(prop("Lifts") >= 4 and (prop("Runs") + prop("Yoga") + prop("Pilates") + prop("Cold Plunges")) >= 2, "Hit ✅", "Missed 🔴"))
```

---

### Step 4: Pre-populate week rows

For each week in your phase, create a row with:
- **Date Range** (Title): the week's date range, e.g. "Mar 8 – Mar 14"
- **Week #**: 1, 2, 3... through your total number of weeks
- **Goal columns**: pre-fill with your targets (Goal Cals, Goal Protein, etc.)

Leave all Actual columns blank until you log that week.

For a 12-week phase with weekly rows, this takes about 10 minutes to set up manually.
For 24 weeks, let Claude do it.

---

### Step 5: Find your Notion Data Source ID

Claude needs the data source ID (not the page ID from the URL) to update your tracker.

**Easiest method:** In a Claude chat with Notion connected, paste this:

> "Can you fetch my Notion page at [your-tracker-page-url] and give me the collection data source ID?"

Claude will fetch the page and return something like:
```
<data-source url="collection://d9a5cecc-85f0-4fce-9ca0-2ab08ace42bc">
```

The string `collection://d9a5cecc-85f0-4fce-9ca0-2ab08ace42bc` is your
`NOTION_DS_ID` — paste it into your SKILL.md configuration.

---

## Phase 2 Database (Optional)

If you're running a two-phase protocol (e.g. Bulk → Cut), create a second database
following the same steps. The only things that change are:
- The title (e.g. `🔥 Phase 2 CUT Tracker`)
- The Cal Status Auto formula threshold (change 0.12 → 0.08 for Cut)
- Different Goal column values (your cut macro targets)

Use the second database's data source ID for `PHASE2_NOTION_DS_ID` in your SKILL.md.

---

## Known Formula Quirks

**`== 0` vs `empty()`:** Notion handles null vs zero differently across column types.
- Number columns that auto-initialize to 0 (exercise fields: Lifts, Runs, etc.) need a
  sum-based null check if used in formulas
- Number columns that are truly null when untouched (Actual Cals, Actual Protein, etc.)
  work correctly with `empty()`
- Using `== 0` on a null-type column breaks the formula — it will show "Hit" on blank rows

If your status columns show results on empty rows, check which blank-detection method
your column type needs.

---

## Troubleshooting

**Formula shows "Hit" on empty rows:**
The formula is using `== 0` when it should use `empty()`. Replace the blank check.

**"Goal column is showing as a formula result instead of a number":**
Make sure Goal Cals etc. are set to type **Number**, not Formula.

**"Claude updated the wrong row":**
Tell Claude the correct week number and it will fix it.

**"I want to add a column Claude doesn't know about":**
Just tell Claude the column name once per session — it will include it in the update.
