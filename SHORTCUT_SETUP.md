# iOS Shortcut Setup Guide

The precise, step-by-step build for the **MFP Weekly Macros** iOS Shortcut (Part 1 of the
system). This pulls your weekly MyFitnessPal macro averages from Apple Health with a fixed
**Sunday–Saturday** window and copies them to your clipboard in one tap.

If you just want to install it, use the iCloud link in the [README](./README.md). This guide
is for building it by hand or understanding how it works.

---

## Prerequisites

1. **MyFitnessPal** installed and logging food daily
2. **MFP → Apple Health sync enabled:**
   - MFP app → More → Settings → Sharing & Privacy → HealthKit Sharing
   - Toggle ON: Calories In, Protein, Carbohydrates, Fat Total
3. **iPhone** (Apple Health / HealthKit is not available on iPad)

---

## Output

```
Weekly MFP (Sun-Sat):
Cal: 2492 | Pro: 186g | Carbs: 215g | Fat: 105g
```

One tap → clipboard → paste into your Claude chat (Part 2).

---

## Build it manually

Open the **Shortcuts** app on your iPhone and create a new shortcut with these actions in order.

### Part 1: Calculate the Sunday–Saturday date window

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

**Why steps 8 and 10 widen the window:** The Find Health Samples "is between" filter in iOS
Shortcuts is **exclusive on both endpoints** (undocumented Apple behavior). Widening by 1 day
on each side means the actual data captured is Sunday 12:00 AM through Saturday 11:59 PM.

### Part 2: Pull macros from Apple Health

Repeat this block **4 times** — once for each macro:

| Step | Action | Configuration |
|------|--------|--------------|
| 13 | **Find Health Samples** | Type: Dietary Energy (or Protein/Carbs/Fat Total). Start Date: **is between** `lastSunday` and `lastSaturday`. Group by: Day. Fill Missing: OFF. Limit: OFF |
| 14 | **Calculate Statistics** | Sum of Health Samples |
| 15 | **Calculate** | Divide Statistics by 7 |
| 16 | **Round Number** | Round Calculated Number to nearest integer |
| 17 | **Set Variable** | Name: `avgCal` (or `avgPro`, `avgCarbs`, `avgFat`) |

Repeat for:
- **Dietary Energy** → `avgCal`
- **Dietary Protein** → `avgPro`
- **Dietary Carbohydrates** → `avgCarbs`
- **Dietary Fat Total** → `avgFat`

### Part 3: Format output and copy

| Step | Action | Configuration |
|------|--------|--------------|
| 29 | **Text** | `Weekly MFP (Sun-Sat):` (new line) `Cal: [avgCal] \| Pro: [avgPro]g \| Carbs: [avgCarbs]g \| Fat: [avgFat]g` — insert variables as tags, not typed text |
| 30 | **Copy to Clipboard** | Input: Text from step 29 |
| 31 | **Show Notification** | "Weekly macros copied!" |

---

## Known quirks

- **"Is between" is exclusive on both endpoints** in Find Health Samples. The date window is
  intentionally widened by 1 day on each side to compensate. Skip this and you'll miss Saturday.
- **Mac Shortcuts has variable-passing bugs** — the Adjust Date action on Mac won't accept
  Dictionary Value as a variable input. Build on iPhone.
- **MFP only syncs data logged AFTER the HealthKit connection is established** — no historical
  backfill.
- **Format String `c`** (ISO day number) doesn't work reliably in Shortcuts. Use `EEEE`
  (full day name) with a text-keyed Dictionary instead.
- The output prints `Cal: Xg` with a "g" even though calories aren't grams — a harmless
  formatting quirk. The Claude skill strips it automatically.

---

## Credits

Built by Dan with help from Claude (Anthropic). The "is between" exclusivity workaround was
discovered through extensive trial and error and is not documented by Apple.
