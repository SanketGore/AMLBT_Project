# Data: what we used and how to rebuild it

All model inputs came from **CMIE ProwessIQ**, downloaded on the GIM campus on 18 November 2026 under the institute's licence. The licence does not permit redistribution, so the nine files are not in this repository. This page records exactly what they contain so that:

- a user with Prowess access can download the same files and verify them against [`prowess_raw/MANIFEST.csv`](prowess_raw/MANIFEST.csv) (SHA-256), and
- anyone else can rebuild comparable files from public sources and run the notebook without changing its code.

## 1. Universe

- **Companies:** the constituents of the NIFTY 200 index as listed in November 2026, excluding banks, NBFCs, insurers, asset managers, exchanges and other financial companies, whose balance sheets make P/B a different quantity. That gives 147 companies; [`universe.csv`](universe.csv) lists each with its NSE symbol, ISIN, NSE basic industry, listing date, the sector we assigned and the number of formation years it contributes to the study.
- **Two further exclusions** are made in the notebook: Groww (Billionbrains Garage Ventures, a stock broker) and Paytm (One 97 Communications, a payments company), which Prowess files as non-financial.
- **Study sample:** 1,102 company-years from 141 companies, formation years 2017–2025, after the filters in Section 4.
- **Survivorship:** because the list is today's index, companies that left the NIFTY 200 before 2026 are missing. The ₹5,000 crore size rule reduces but does not remove this bias.

## 2. The nine files

| File | Content | Period | Used |
|---|---|---|---|
| `prowess_C_2010_2018.txt` | Abridged annual financials, consolidated | FY2012–FY2018 | yes |
| `prowess_C_2019_2026.txt` | Abridged annual financials, consolidated | FY2019–FY2026 | yes |
| `prowess_S_2010_2018.txt` | Abridged annual financials, standalone | FY2011–FY2018 | yes |
| `prowess_S_2019_2026.txt` | Abridged annual financials, standalone | FY2019–FY2026 | yes |
| `prowess_px_adjclose_monthly.txt` | NSE adjusted closing price, last trading day of each month (₹) | Jan 2010–Oct 2026 | yes |
| `prowess_px_mcap_monthly.txt` | NSE market capitalisation, **average** over the month (₹ crore) | Jan 2010–Oct 2026 | yes |
| `prowess_px_nifty_monthly.txt` | NIFTY 50 closing level at month end | Jan 2010–Oct 2026 | yes |
| `prowess_identity.txt` | Company name, ISIN, NSE symbol, NSE basic industry, Prowess industry group, NIC name, listing date | current | yes |
| `prowess_px_totret_monthly.txt` | Prowess "total return" series | Jan 2010–Oct 2026 | **no** (it holds the last day's return, not the month's) |

The annual files were split into two year ranges only because of a column limit in the Prowess export screen.

## 3. Annual financial fields (19 per year)

All amounts are in ₹ crore for the financial year ending on the date in `Year`. Indian companies mostly close on 31 March; FY2024 means the year ending 31 March 2024.

| Prowess field | Notebook name | Meaning | Public source |
|---|---|---|---|
| Year | `report_date` | Balance-sheet date, `dd-mm-yyyy` | Annual report / results filing |
| Months | `months` | Length of the accounting period (12 unless the year end changed) | Annual report |
| Revenue From Operations | `sales` | Net revenue from operations | Statement of profit and loss |
| Other Income | `other_income` | Non-operating income (interest, dividends, gains) | Statement of profit and loss |
| Finance Cost | `interest` | Interest and other borrowing costs | Statement of profit and loss |
| Depreciation & Amortisation | `depreciation` | Depreciation, amortisation and impairment of fixed assets | Statement of profit and loss |
| Total profit before tax | `pbt` | Profit before tax, after exceptional items | Statement of profit and loss |
| Total profit (loss) for period | `net_profit` | Profit after tax (consolidated: before minority share) | Statement of profit and loss |
| Share Capital | `equity_capital` | Paid-up equity capital | Balance sheet |
| Reserves & Surplus | `reserves` | Other equity | Balance sheet |
| Minority interest | `minority` | Non-controlling interest (consolidated only) | Balance sheet |
| Long term Borrowings | `lt_borrowings` | Non-current borrowings | Balance sheet |
| Short Term Borrowings | `st_borrowings` | Current borrowings, including current maturities of long-term debt | Balance sheet |
| Assets | `total_assets` | Total assets | Balance sheet |
| Cash & Bank Balances | `cash` | Cash, cash equivalents and other bank balances | Balance sheet |
| Net cash flow from operating activities | `cfo` | Cash from operations | Cash-flow statement |
| Dividend Paid | `dividend_paid` | Equity dividends paid in the year (sign ignored) | Cash-flow statement |
| Net worth | `net_worth` | Equity attributable to shareholders = share capital + other equity | Balance sheet |
| Operating Profit | `operating_profit` | Prowess operating profit (read but not used as an input) | n/a |

## 4. How the notebook turns the files into a modelling table

1. **Basis.** Ratios needing history (ROE on average equity, 3-year sales growth, 3-year ROE volatility) are computed separately within the consolidated and standalone series. Each company-year then uses consolidated figures where available and standalone otherwise; if the consolidated history is too short for the 3-year ratios, the standalone values are used for those two inputs.
2. **Formation date.** Each year's accounts are matched to the first 30 June at least 60 days after the year end (31 March 2024 accounts → June 2024). Periods that are not 12 months long have flow items scaled to 12 months.
3. **Target.** log(P/B) on 30 June, where P/B = June's average market cap ÷ the latest `net_worth`.
4. **Inputs.** ROE, operating margin `(pbt + interest + depreciation − other_income) / sales`, 3-year sales CAGR, debt/equity `(lt + st borrowings) / net_worth`, log(1 + interest cover) with cover capped at 100, asset turnover, payout `dividend / net_profit` (0–1.5), cash conversion `cfo / net_profit` (−1 to 3), ROE volatility, log sales, 12 sector dummies, and three market inputs measured up to 30 June: log 12-month momentum, 24-month volatility of monthly log returns (annualised) and 36-month beta to the NIFTY 50. Ratio caps are fixed in advance; the notebook (Section 5) lists them.
5. **Filters.** Formation years 2017–2025; market cap ≥ ₹5,000 crore on the formation date; 0.05 ≤ P/B ≤ 200; ROE available.
6. **Forward return** (backtest only, never an input): change in adjusted close over the next 12 months, so it excludes dividends.
7. **Sectors.** NSE basic industry labels are grouped into 12 sectors with the keyword rules in notebook Section 3; `universe.csv` shows the result.

## 5. Exact file layout the notebook reads

All files are UTF-8 text, fields separated by `|`, one company per row. The reader looks for two marker rows and ignores everything above them.

**Annual financials** (`prowess_{C|S}_2010_2018.txt`, `prowess_{C|S}_2019_2026.txt`):

```
<any title lines>
Slot Date||Mar 2019|Mar 2019|...(19 times)...|Mar 2020|...
Company Code|Company Name|Year|Months|Revenue From Operations|Other Income| ... |Net worth|Operating Profit|Year|Months|...
35548|A P L Apollo Tubes Ltd.|31-03-2019|12|7152.32|11.71| ... 
```

- The `Slot Date` row gives the fiscal-year label for each column, so every field in a year's block shares one label (`Mon yyyy`).
- The `Company Code` row repeats the 19 field names above once per year, in any order. Unknown names are ignored, and an empty cell means missing.
- `Year` must be `dd-mm-yyyy`. Both year-range files must exist for each basis, but one of them may contain only the two marker rows.

**Monthly series** (`prowess_px_adjclose_monthly.txt`, `prowess_px_mcap_monthly.txt`, `prowess_px_nifty_monthly.txt`):

```
<any title lines>
Slot Date||31 Jan 2010|28 Feb 2010|...|31 Oct 2026
Company Code|Company Name|Adjusted Closing Price|Adjusted Closing Price|...
21420|A B B India Ltd.|811.00|794.60|...
```

- Slot dates are the **calendar month end** in `dd Mon yyyy` format, even when the last trading day is earlier. The notebook looks up `30 Jun yyyy` exactly.
- In the NIFTY file, every company row carries the same index values; one row is enough.

**Identity** (`prowess_identity.txt`): line 1 is a title, line 2 is the header, and data starts on line 3. The notebook uses the columns `Company Code`, `Company Name`, `NSE symbol`, `NSE Basic Industry classification` and `Industry group` (the last is a fallback when the NSE label is blank).

`Company Code` is only a join key and must match across files. In a rebuild, the NSE symbol can serve as the code.

**Checksums.** `prowess_raw/MANIFEST.csv` needs at least the columns `file,sha256,description`. Every file it lists must exist and match its checksum. After a rebuild, write a new manifest for the files you produced and leave `prowess_px_totret_monthly.txt` out.

## 6. Rebuilding from public sources

| Input | Suggested public source | Notes |
|---|---|---|
| Company list | `universe.csv` (NSE symbols and ISINs) | Same 147 companies |
| Annual financials, FY2011–FY2026, consolidated and standalone | Company annual reports and results filed with NSE/BSE (XBRL financial results from about FY2016; PDF annual reports before that); MCA AOC-4 filings | Use audited full-year figures under Ind AS (from FY2017) or Indian GAAP (earlier) |
| Monthly adjusted close | Yahoo Finance via `yfinance`, ticker `<SYMBOL>.NS`, monthly interval, **`Close` with `auto_adjust=False`** | Yahoo's `Close` is adjusted for splits and bonuses like Prowess's series. Do not use `Adj Close`, which also adjusts for dividends |
| Monthly average market cap | Mean of daily close in the month × shares outstanding | Shares from NSE shareholding-pattern filings (quarterly); bhavcopy or Yahoo daily prices |
| NIFTY 50 | `^NSEI` on Yahoo Finance, or niftyindices.com historical data, month-end close | |
| NSE industry | NSE equity list / company page "Basic Industry" | |

**Why rebuilt results will differ from ours:**

- Prowess standardises line items across companies and years. Public statements mix Indian GAAP (to FY2016) and Ind AS (FY2017 on) and are often restated, so borrowings, other income and net worth can differ.
- Prowess's adjusted close also corrects for rights issues; Yahoo does not.
- Average market cap rebuilt from quarterly share counts will be slightly off when shares change mid-quarter.
- Small differences near the ₹5,000 crore or P/B filters change which company-years enter the sample, and with that the walk-forward splits.

What should not change: the ordering of the models, fundamentals explaining far more than market inputs, the network's stability across seeds, and the absence of a reliable link between mispricing and next-year returns.
