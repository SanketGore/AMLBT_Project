# Pilot: 20 NIFTY 50 companies, Screener.in data

Before Prowess access was available, we ran the full pipeline on 20 non-financial NIFTY 50 companies, using annual accounts exported from Screener.in (FY2017–FY2026, consolidated) and weekly prices from Yahoo Finance. The notebook is saved with the output cells of its Colab run (fingerprint `cd8ace47...`).

| Walk-forward, 2022–2026 (100 predictions) | Out-of-sample R² |
|---|---:|
| Ridge | 0.827 |
| Random Forest | 0.762 |
| XGBoost | 0.785 |
| MLP | 0.866 |

As in the main study, the MLP beat Ridge, last year's P/B was the strongest benchmark, and mispricing did not predict returns (mean MLP IC −0.16). With 20 companies these differences are not reliable, which is why the paper reports the Prowess study.

**Data.** The 20 Screener `.xlsx` exports (one per company, named by NSE symbol) are not included because Screener's terms restrict redistribution; `data/raw/MANIFEST.csv` holds their SHA-256 checksums. The notebook downloads weekly prices from Yahoo Finance on its first run (`yfinance`, `<SYMBOL>.NS` and `^NSEI`, 2016-01-01 to 2026-11-14) and freezes them with a checksum. A fresh download can differ slightly from ours if Yahoo has revised its history.
