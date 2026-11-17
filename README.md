# Farm Data Reader

A plain-Python reader for the daily mortality records of Aarambha Poultry Farm,
Chitwan, Nepal.

## Background

I am a 23-year-old layer farmer and a final-year BSc IT student at ISMT College,
Chitwan. My birds live in a shared four-lane house of roughly 6,800 birds —
lanes 1 and 2 are mine (about 3,400 birds), lanes 3 and 4 belong to my uncle.

I began recording daily farm data in Excel on 12 September 2026. Once I had
more than 50 days of records, I wrote this reader to get them out of the
spreadsheet and into code, the first step toward an early-warning model for
flock health.

## What it does

Reads the exported lane-level CSV and reports, in a single pass:

- how many data rows were read
- which dates and lanes had no usable death figure
- total deaths per lane
- the lane with the highest total deaths
- the worst single lane-day: lane, date and count
- lane 1's most recent bird count

## How to run it

- Python 3.10 or newer. No external libraries.
- Place the exported CSV beside the notebook as `aarambha_lanes.csv`.
- Open `reader.ipynb` and run the cells in order.

A 15-row sample (`sample_lanes.csv`) is included so the code runs without the
full dataset.

## What I learned

**2,340 phantom rows.** My first run reported 2,340 skipped rows when my data
is only 260 rows long. They were not blank lines printing the raw lines with
`repr()` showed pre-formatted template rows carrying a batch ID and a lane but
no date, which Excel exports because they genuinely have content. The date
column is what distinguishes a real record from a template. Guarding on an
empty date took the skipped count from 2,340 to 4.

**One `try` per risky line.** I first wrapped two conversions in one `try`
block. A row with a valid death count but a missing bird count failed on the
second line and was discarded whole losing data I had already successfully
read. Splitting them into separate blocks fixed it.

**Lane grain versus farm grain.** I was certain the highest single-lane death
count was higher than my code reported. It turned out the number I remembered
was a farm-wide daily total across all four lanes, not a single lane's figure.
Values from different grains of the same dataset are easy to confuse when
checking by eye.

**Three lines before the header.** The exported sheet writes a title, a
description and a blank line before the column names, so four reads are needed
before the data begins.

**Everything arrives as text.** Numbers read from a file are strings until
converted, and dates written as `YYYY-MM-DD` compare correctly as text which
is what makes "the latest date" findable without parsing.

**Every path out of a loop must advance the file**, or that path repeats the
same line forever.

## Limitations

- The notebook ships pointed at sample_lanes.csv. The figures quoted above come 
  from the full 260-row dataset, which is not published.
- Validated against Excel: L1 52, L2 56, L3 28, L4 24. Totals match exactly.
- Ties on the worst lane-day resolve to the earliest occurrence. L1 and L2 both
  recorded 23 deaths on different dates but only L1 is reported.
- Rows for the current day, not yet filled in, are indistinguishable from
  genuinely missing records. Both appear as skipped.
- Column positions are hard-coded. A change to the sheet's column order would
  break the reader silently.
- A comma inside the free-text `Note` field would shift every column after it,
  corrupting the bird count and mortality rate for that row.
- This reads mortality only. It does not diagnose anything a lane with high
  mortality means "inspect" the cause is a veterinarian's judgement.
- The full dataset is not published. Only the sample is included.

## Data

One row per lane per day.

| Column | Meaning |
|---|---|
| `Batch_ID` | the flock batch (A001 is the first flock) |
| `Date` | the date the record covers |
| `Age_Days` | flock age in days |
| `Lane_ID` | which lane of the house — L1 to L4 |
| `Deaths` | birds found dead that day |
| `Culls` | birds removed deliberately because they were sick or weak |
| `Transfers` | birds moved between lanes — changes a count without a death |
| `Note` | free text |
| `House_ID` | which house the lane belongs to |
| `Birds_Alive_End` | birds remaining in the lane at the end of the day |
| `Mortality_Rate` | deaths ÷ birds at start of day; exports as text with a `%` |

The reader ignores the stored mortality rate and computes its own from deaths
and bird counts.