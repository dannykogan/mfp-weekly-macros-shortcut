# MFP Weekly Macros — iOS Shortcut

A free iOS Shortcut that pulls your weekly MyFitnessPal macro averages (calories, protein, carbs, fat) from Apple Health with a fixed **Sunday–Saturday** window. One tap → clipboard → paste anywhere.

## Why this exists

MyFitnessPal's API is private and not available to individual users. The built-in CSV export requires a Premium subscription and manual calculation. This shortcut uses Apple Health as a bridge — MFP syncs daily nutrition data to HealthKit automatically, and this shortcut reads it with proper date boundaries.

## What it does

- Calculates the most recently completed **Sunday–Saturday** week regardless of when you run it
- Pulls daily totals for **Calories, Protein, Carbs, and Fat** from Apple Health
- Averages each macro across 7 days
- Copies a formatted summary to your clipboard

Example output:
```
Weekly MFP (Sun-Sat):
Cal: 2492 | Pro: 186g | Carbs: 215g | Fat: 105g
```

## Prerequisites

1. **MyFitnessPal** installed and logging food daily
2. **MFP → Apple Health sync enabled:**
   - MFP app → More → Settings → Sharing & Privacy → HealthKit Sharing
   - Toggle ON: Calories In, Protein, Carbohydrates, Fat Total
3. **iPhone** (Apple Health / HealthKit is not available on iPad)

## How to install

### Option A: iCloud link (easiest)
[Download shortcut via iCloud link](#) *(replace with your iCloud share link)*

### Option B: Build it manually

Open the **Shortcuts** app on your iPhone and create a new shortcut with these actions in order:

#### Part 1: Calculate the Sunday–Saturday date window

| Step | Action | Configuration |
|------|--------|--------------|
| 1 | **Current Date** | No configuration needed |
| 2 | **Format Date** | Date Format: Custom, Format String: `EEEE` |
| 3 | **Set Variable** | Name: `Formatted Date`, Input: Formatted Date from step 2 |
| 4 | **Dictionary** | See day-name lookup table below |
| 5 | **Get Dictionary Value** | Get Value for `Formatted Date` in `Dictionary` |
| 6 | **Adjust Date** | Subtract `Dictionary Value` days from `Current Date` |
| 7 | **Adjust Date** | Get Start of Day from Adjusted Date |
| 8 | **Adjust Date** | Add 1 day to Adjusted Date |
| 9 | **Set Variable** | Name: `lastSaturday`, Input: Adjusted Date |
| 10 | **Adjust Date** | Subtract 7 days from `lastSaturday` |
| 11 | **Adjust Date** | Get Start of Day from Adjusted Date |
| 12 | **Set Variable** | Name: `lastSunday`, Input: Adjusted Date |

**Dictionary (step 4):**

| Key | Type | Value |
|-----|------|-------|
| Monday | Number | 2 |
| Tuesday | Number | 3 |
| Wednesday | Number | 4 |
| Thursday | Number | 5 |
| Friday | Number | 6 |
| Saturday | Number | 0 |
| Sunday | Number | 1 |

**Important note on the "is between" filter:** The Find Health Samples "is between" filter in iOS Shortcuts is **exclusive on both endpoints**. That's why steps 8 and 10 widen the window by 1 day on each side — so the actual data captured is Sunday 12:00 AM through Saturday 11:59 PM.

#### Part 2: Pull macros from Apple Health

Repeat this block **4 times** — once for each macro:

| Step | Action | Configuration |
|------|--------|--------------|
| 13 | **Find Health Samples** | Type: Dietary Energy (or Protein/Carbs/Fat Total). Start Date: **is between** `lastSunday` and `lastSaturday`. Group by: Day. Fill Missing: OFF. Limit: OFF |
| 14 | **Calculate Statistics** | Sum of Health Samples |
| 15 | **Calculate** | Divide Statistics by 7 |
| 16 | **Round Number** | Round Calculated Number to nearest integer |
| 17 | **Set Variable** | Name: `avgCal` (or `avgPro`, `avgCarbs`, `avgFat`) |

Repeat for:
- **Dietary Protein** → `avgPro`
- **Dietary Carbohydrates** → `avgCarbs`
- **Dietary Fat Total** → `avgFat`

#### Part 3: Format output and copy

| Step | Action | Configuration |
|------|--------|--------------|
| 29 | **Text** | `Weekly MFP (Sun-Sat):` (new line) `Cal: [avgCal] \| Pro: [avgPro]g \| Carbs: [avgCarbs]g \| Fat: [avgFat]g` — insert variables as tags, not typed text |
| 30 | **Copy to Clipboard** | Input: Text from step 29 |
| 31 | **Show Notification** | "Weekly macros copied!" |

## Known quirks

- **"Is between" is exclusive on both endpoints** in Find Health Samples. The date window is intentionally widened by 1 day on each side to compensate.
- **Mac Shortcuts has variable-passing bugs** — the Adjust Date action on Mac won't accept Dictionary Value as a variable input. Build on iPhone.
- **MFP only syncs data logged AFTER the HealthKit connection is established** — no historical backfill.
- **Format String `c`** (ISO day number) doesn't work reliably in Shortcuts. Use `EEEE` (full day name) with a text-keyed Dictionary instead.

## Use case: automated fitness tracking with Notion

This shortcut was built as part of a 24-week body composition tracking protocol. The workflow:

1. Log food daily in MyFitnessPal
2. Run this shortcut weekly (Sunday or Monday)
3. Paste the output into a Claude chat
4. Claude pushes the data into a Notion fitness tracker database with auto-calculating compliance formulas

## Cost

**$0.** No paid apps, no API keys, no subscriptions.

## License

MIT — do whatever you want with it.

## Credits

Built by Dan with help from Claude (Anthropic). The "is between" exclusivity workaround was discovered through extensive trial and error.
