# Weightlifting-Stat-Tracking
## Personal Lifting Stat Tracker
A comprehensive strength tracking spreadsheet designed to monitor personal progress, benchmark lifts, and generate analytics-ready datasets.

## Overview 
This project is a personal weightlifting tracking system built in Google Sheets.
It tracks individual lift performance, bodyweight, and percentiles relative to other lifters.
The tracker automatically calculates current estimates, personal records, progress over time, and lift scores based on data from strengthlevel.com.
All data is converted into a tidy format suitable for external analytics tools like Tableau.

## Core Features
### 1.Training Log (Raw Input)
- Central sheet where users enter day-to-day lifts.
- Individual lift pages automatically pull data from the log.
- Designed for simplicity: user inputs → formulas transform → dashboard and lift pages update. 

<img width="3199" height="1308" alt="Spencer Weightlifting Stat Tracking - TrainingLog" src="https://github.com/user-attachments/assets/3750aa74-d843-4596-96c6-d9270b4a9f1b" />
  
### 2. Body Weight Page
- Users log their weight periodically.
- Uses LINEST() regression to predict weight trends.
- Automatically interpolates weight on training days:
- Finds the closest weigh-in before and after a lift date.
- Uses the nearest weight or average when between two dates.
- Bodyweight is used for relative strength and percentile calculations.

<img width="2786" height="1268" alt="Spencer Weightlifting Stat Tracking - Weight" src="https://github.com/user-attachments/assets/7a6ca3b2-3193-4ba7-930f-90b6e1ed88af" />

### 3. Individual Lift Pages (e.g., Bench, Squat, Deadlift)
Each lift page shows:
- PR (all-time max)
- Current max
- Estimated current max
- Relative PR vs. bodyweight
- Relative current
- Progress and percent progress
- Two visual graphs:
Absolute lift progress
Relative lift progress
- A lift score based on strengthlevel.com percentile tables

<img width="2447" height="2135" alt="Spencer Weightlifting Stat Tracking - Bench" src="https://github.com/user-attachments/assets/17fab4ba-4500-4b85-922d-8baa9bb5b794" />

### 4. Dashboard
- Provides a high-level snapshot:
- Explanations of features
- Arm and leg strength indices
- Lift score graphs
- 1000-lb club progress
- Summary of PRs and changes over time

<img width="3226" height="1929" alt="Spencer Weightlifting Stat Tracking - Dashboard" src="https://github.com/user-attachments/assets/29e91d76-e340-4909-bcc7-780cce337d65" />

### 5. Tidy Data Export
All metrics are transposed and converted into "long format" using query functions.
Each row contains: date, bodyweight, lift type, and metrics.
Ideal for Tableau

<img width="3300" height="2550" alt="Spencer Weightlifting Stat Tracking - TableauData" src="https://github.com/user-attachments/assets/ffb39f38-7796-45aa-a7b3-ad31781da6f5" />

# Example Formulas Used
### Merge lift datasets
```=QUERY({ArmsDataTidy!A2:G; LegsDataTidy!A2:G}, "select * where Col1 is not null", 0)'```

### Weight regression
```=LINEST(filter(G9:G,G9:G <> ""),filter(D9:D,D9:D <> ""),true,true)```

### Pull training log values dynamically
```
=MAP(
  SEQUENCE(30),
  LAMBDA(i,
    LET(
      val, OFFSET(TrainingLog!$D$5, 0, (i-1)*5),
      IF(ISNUMBER(val), val, "")
    )
  )
)
```
### Interpolated bodyweight on lift dates
```
=IF(Dashboard!A2="", "",
  LET(
    before, XLOOKUP(Dashboard!A2, Weight!D2:D, Weight!G2:G,, -1),
    after,  XLOOKUP(Dashboard!A2, Weight!D2:D, Weight!G2:G,, 1),
    IF(OR(ISNA(after), after=""), before, AVERAGE(before, after))
  )
)
```
### Sorting dynamic tables
```
=IFNA(
  SORT(K9:N16,
    IF(C1="PRs", 2,
    IF(C1="Scores", 3,
    IF(C1="% Progress", 4, 1))),
  false),
"")
```
### Create tidy lift dataset
```
=QUERY({
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Bench",)), TransposedArms!C2:F;
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Military Press",)), TransposedArms!G2:J;
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Single Arm Rows",)), TransposedArms!K2:N;
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Dumbell Curls",)), TransposedArms!O2:R;
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Pullups",)), TransposedArms!S2:V;
  TransposedArms!A2:A, TransposedArms!B2:B, ARRAYFORMULA(IF(ROW(TransposedArms!A2:A),"Dips",)), TransposedArms!W2:Z
},
"select Col1, Col2, Col3, Col4, Col5, Col6, Col7 where Col1 is not null",
0)
```

### 5. Tableau Example Graphs
- Hypothetical data was used to create the following visualizations
<img width="800" height="600" alt="Captura de Pantalla 2025-11-26 a la(s) 21 52 05" src="https://github.com/user-attachments/assets/73818535-6aa3-47ec-8110-053870f2479f" />

<img width="800" height="800" alt="Captura de Pantalla 2025-11-26 a la(s) 21 45 07" src="https://github.com/user-attachments/assets/c4047875-8893-4109-a64f-8dd0cc2b096c" />


## Why This Tracker Is Useful
- Removes manual calculations
- Automatically tracks long-term performance
- Accounts for bodyweight changes
- Compares normalized performance between lifters
- Produces analytics-ready datasets
