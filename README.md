# freight-rate-ml-assessment-1-

\# Freight Rate Prediction



Predicts the dollar price of 12,000 truckload shipments (Nov to Dec 2025) and a 31-day price chart for one fixed lane. Final model: LightGBM trained on Jan to Oct 2025 loads, with an outlier-resistant loss and day-context features ("recipe D").



\## Quick start for reviewers (about 5 minutes)



Requires Python 3.10 or newer. Windows PowerShell:



```powershell

git clone https://github.com/phoenixstein/freight-rate-ml-assessment-1-.git

cd freight-rate-ml-assessment-1-

python -m venv .venv

.\\.venv\\Scripts\\Activate.ps1

python -m pip install -r requirements.txt

python src\\train\_final.py

python score.py --predictions validation\_predictions.csv --december-predictions data/december\_chart\_inputs.csv

```



macOS/Linux: activate with `source .venv/bin/activate` and use `/` in paths. Run every command from the repo root.



Expected result: `train\_final.py` finishes in about a minute, then `score.py` prints

`Validated 12,000 final predictions.` and `Validated 31 fixed December predictions.`, and creates `scorer\_results/candidate\_december.png`. The random seed is fixed (42), so results are reproducible.



\## Submission files



| File | What it is |

|---|---|

| `validation\_predictions.csv` | 12,000 predictions (`load\_id,predicted\_rate`) |

| `data/december\_chart\_inputs.csv` | December rows with `predicted\_rate` filled in |

| `scorer\_results/candidate\_december.png` | Chart produced by `score.py` |

| `report/report.docx` | Approach, validation, business case |



\## How the model was chosen (all numbers from this repo)



\- \*\*Split:\*\* time-based. Every month from Apr to Oct is predicted using only earlier months (rolling exams), because the real task is forecasting Nov to Dec. A random split would leak the future.

\- \*\*Model race (Jul-Oct exams, average miss):\*\* median-per-mile baseline MAE $254 (MAPE 11.4%), quote x distance $305 (14.8%), Ridge $143 (6.4%), random forest $132 (5.9%), LightGBM \*\*$130 (5.8%)\*\*. LightGBM and random forest were too close to call; LightGBM was chosen for steadiness and speed (3 s vs 53 s).

\- \*\*Improvements:\*\* an outlier-resistant (L1) loss plus day-context features gave about 10% lower MAE over seven months (146 vs 162 dollars). Predicting the correction to the quote and tuning settings were tried and rejected. The pre-agreed strict rule (better in 5 of 7 months) selected the baseline recipe; recipe D was better in 4 months, tied in 2 and worse in April (cold start), so it was chosen as a documented judgment call. The two recipes differ by 1.9% on average on the 12,000 loads.

\- \*\*Where the errors are:\*\* the worst 1% of loads cause 96% of squared error. Ordinary loads (98%) are predicted within about 3.5% (MAPE); rare spike loads are not predictable from the provided columns.

\- \*\*Price range:\*\* expected price x \[0.986, 1.070]. Leave-one-month-out coverage was 87%, 36%, 98% and 91% (target 80%), so it is uneven, with August weakest.



## Business value (simulated broker, Jul to Oct, 19,194 loads)

Each guesser prices every load using only earlier months. The broker adds a 5% margin and the customer accepts up to 10% above the true cost (illustrative assumptions, tested across 9 combinations).

| Price guesser | Profit per ordinary load | Net profit incl. unusual loads |
|---|---|---|
| Gut feel (median $/mile) | $8 | -$661k |
| Posted quote x distance | -$54 | -$1,841k |
| Basic model (A) | $60 | +$306k |
| Final model (D) | **$68** | **+$449k** |

The final model was the best guesser in 7 of 9 assumption sets. About 350 unusual loads lose about $825k under every method. See `report/report.docx` for the full discussion.



\## Data-quality handling (all in code, raw files never edited)



Negative weights made positive (292 train, 145 validation); blank weights filled by trailer-type median; blank `market\_index` filled by same-day median; "was blank" flags added; city names not used as features (8 cities appear only in validation), coordinates and distance are used instead; fill values are learned from training rows only.



\## Reproduce the full analysis (optional, run in this order)



`src/check\_setup.py`, `data\_quality.py`, `baseline.py`, `compare\_models.py`, `error\_analysis.py`, `spike\_profile.py`, `improve.py`, `stress\_test.py`, `risk\_range.py`, `business\_simulation.py`. Shared code is in `common.py`, `runner.py` and `recipes.py`. Logs are in `notes/`, tables in `results/`.



\## Limitations



Spotter's metric is not shown to candidates, so MAE, MAPE, RMSE and R2 are reported. Rare spike loads (about 2% of loads) cause most dollar error and are not predictable from the provided columns. Only 10 months of history exist, so December seasonality cannot be learned directly; December chart inputs use each day's median market values from `validation.csv`. October was inspected after the first improvement round, so no untouched exam remains in this data.



## Note on AI assistance

I used Claude as a tutor and code assistant. I ran every script, checked every output and can explain each step.

