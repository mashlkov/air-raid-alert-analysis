# Daily Air Raid Alert Duration — Kyiv Oblast

Educational historical time-series experiment; **not a warning system or personal safety advice**.

## Question
Can the daily recorded duration of air raid alerts in Kyiv Oblast be forecast one day ahead using the previous seven calendar days? Compare a seven-day mean baseline to a Random Forest regressor.

## Source and reproducibility
Historical volunteer dataset: https://github.com/Vadimkin/ukrainian-air-raid-sirens-dataset (file `datasets/volunteer_data_en.csv`). Frozen CSV snapshot acquired **2026-10-11 12:18:26 Kyiv time**, SHA256 `e403ece29651a1ffe674b1fb6afa72264c76070d6ff57686432f9c07f78fb4f0`, **115,989 rows** across all regions. The CSV must be supplied separately as `air_raid_source_snapshot.csv`. The notebook rejects any other snapshot rather than silently changing its results.

In Google Colab, upload `air_raid_source_snapshot.csv` in the Files panel (to the notebook's working directory), open `air_raid_analysis_clean.ipynb`, and run **Runtime → Run all**. The notebook writes `test_predictions.csv`. For local execution, install packages with `pip install -r requirements.txt` and run the notebook in the same directory as the CSV. Colab files are temporary; re-upload the CSV after a runtime reset.

## Processing
UTC timestamps are converted to `Europe/Kyiv`. Intervals are split at local midnight. Duplicate records, overlaps, and malformed intervals are inspected. Missing calendar days are left as unknown rather than zero. Records marked `naive=True` contain estimated 30-minute durations; days affected by those records are excluded from target evaluation. The experiment uses dates from 2024-01-01 to 2026-10-09, with the last 30 calendar days (2026-09-10 through 2026-10-09) held out for final testing.

## Modelling
Seven lag features (`lag_1` through `lag_7`) refer to the seven *previous calendar days*. An observation is evaluated only when these features and its target are available. The baseline is their arithmetic mean. A Random Forest (100 trees, max depth 4, minimum leaf size 5, random seed 42) is validated chronologically on the most recent 20% of eligible development samples, then refit on all development samples before held-out evaluation. No random shuffling.

## Recorded results (from user's original Colab execution, reproduced after restart)
| Evaluation | Random Forest MAE | 7-day baseline MAE |
| --- | ---: | ---: |
| Validation (46 samples) | 66.11 minutes | 67.33 minutes |
| Final test (22 samples) | **61.22 minutes** | 64.84 minutes |

The tests in the cleaned notebook have **not yet been executed**; those numbers are from the original notebook runs and should be confirmed by a clean run.

## Limitations
The original feed can be revised or published with delay, and it does not provide event publication timestamps. Therefore the historical backtest assumes the prior day's final duration was available by the time of each prediction; this cannot be fully established and is a remaining potential source of real-world leakage. An absence of records does not establish absence of alerts. Excluding `naive`-affected days and requiring complete lag histories introduces sample selection bias. Reported errors are over 22 eligible dates rather than all 30 test dates. Results cannot be used for safety decisions.
