# Corn yield and weather: source-data mirror

This repository hosts source files used in a FIT5145 Assignment 3 corn-yield analysis. GitHub is a download mirror, not the original data provider. The original providers are USDA NASS and NOAA NCEI.

The analysis studies **12 US states, 2000–2024**, using May–August weather. The files here are inputs **before unit conversion and the yield–weather merge**. They are not the final modelling table. No report, predictions, model cache, or credentials are included.

## Files

| File | Contents |
|---|---|
| `usda_corn_yield_state_raw_1981_2024.tsv` | 528 USDA state-year records: 12 states × 44 years (1981–2024). The official header and selected row bytes are preserved. R filters these records to 2000–2024, producing 300 records. |
| `climdiv-pcpnst-v1.0.0-20260904` | Unmodified NOAA statewide monthly precipitation file. Original units: inches. |
| `climdiv-tmpcst-v1.0.0-20260904` | Unmodified NOAA statewide monthly mean-temperature file. Original units: degrees Fahrenheit. |
| `climdiv-tmaxst-v1.0.0-20260904` | Unmodified NOAA statewide monthly mean of daily maximum temperatures. Original units: degrees Fahrenheit. This is not the single highest temperature in each month. |
| `usda_readme.txt` | USDA's source-file field documentation, retained from the original collection. |
| `noaa_state_readme.txt` | NOAA's original record-layout, unit, missing-value and state-code documentation. |
| `SHA256SUMS.txt` | SHA256 checksums for the six source/documentation files above. |

The NOAA files have no filename extension because their original official filenames are preserved. They are fixed-width plain-text files.

## Original sources and versions

Original collection date recorded for these files: **22 September 2026**. Collection/release dates identify the data version; they do not extend the analysis beyond 2024.

### USDA NASS

- Database: [Quick Stats](https://quickstats.nass.usda.gov/).
- Official bulk-download catalogue: [USDA datasets](https://www.nass.usda.gov/datasets/).
- Original release: `qs.crops_20260919.txt.gz`.
- Original release URL: <https://www.nass.usda.gov/datasets/qs.crops_20260919.txt.gz>.
- The original dated URL returned HTTP 404 when checked on 3 October 2026. It is retained here as provenance, not as a guaranteed working download. The official catalogue rotates releases.
- Original compressed file size: 1,129,988,139 bytes. The full file is not included in this repository.
- Original compressed-file SHA256: `f9244c8ee8e0fb4267338e807959d22b2322e757b75aa9d07a40a9579b1857bb`.

The TSV is a **filtered subset of official raw records**, not an untouched copy of the full USDA database. The subset extraction selected the following fields and retained the matched lines without changing the source values:

| Field | Filter |
|---|---|
| `SOURCE_DESC` | `SURVEY` |
| `SHORT_DESC` | `CORN, GRAIN - YIELD, MEASURED IN BU / ACRE` |
| `AGG_LEVEL_DESC` | `STATE` |
| `DOMAIN_DESC` | `TOTAL` |
| `DOMAINCAT_DESC` | `NOT SPECIFIED` |
| `FREQ_DESC` | `ANNUAL` |
| `REFERENCE_PERIOD_DESC` | `YEAR` |
| `UNIT_DESC` | `BU / ACRE` |
| `YEAR` | 1981–2024 in the archived subset; 2000–2024 in the current R analysis |
| `STATE_ALPHA` | `IL`, `IN`, `IA`, `KS`, `MI`, `MN`, `MO`, `NE`, `ND`, `OH`, `SD`, `WI` |

### NOAA NCEI

Official catalogue: [nClimDiv monthly statewide data](https://www.ncei.noaa.gov/monitoring-content/data/us/climdiv/monthly/current/).

Version: **4 September 2026**.

- [Precipitation source](https://www.ncei.noaa.gov/monitoring-content/data/us/climdiv/monthly/current/climdiv-pcpnst-v1.0.0-20260904)
- [Mean-temperature source](https://www.ncei.noaa.gov/monitoring-content/data/us/climdiv/monthly/current/climdiv-tmpcst-v1.0.0-20260904)
- [Mean daily maximum-temperature source](https://www.ncei.noaa.gov/monitoring-content/data/us/climdiv/monthly/current/climdiv-tmaxst-v1.0.0-20260904)
- [Official record documentation](https://www.ncei.noaa.gov/monitoring-content/data/us/climdiv/monthly/current/state-readme.txt)

These are statewide area-weighted summaries, not corn-growing-area-weighted weather. NOAA files include years and geographical records outside the study; R filters them before merging.

## Processing performed in R Markdown

1. Download the source files using public GitHub raw URLs and verify their checksums.
2. Filter USDA records to the specified annual corn-grain-yield measure, the 12 states and 2000–2024.
3. Parse NOAA's fixed-width records and select the corresponding states, years and May–August months.
4. Convert precipitation using `millimetres = inches * 25.4`, and temperature using `Celsius = (Fahrenheit - 32) * 5 / 9`. Store converted weather values to six decimal places.
5. Match states using an explicit code mapping; USDA and NOAA numerical state codes are different.
6. Merge by state and year and check missing values, duplicate keys, numeric values and complete 12 × 25 coverage.
7. Retain a 12-weather-feature source table for feature-selection checks. Use only the four monthly precipitation and four mean-temperature variables in the modelling table; maximum temperature is not a model input.
8. Split training years (2000–2019) and test years (2020–2024) before exploratory fitting and model validation. Estimate trends and standardisation parameters within each training window.

| State | Abbreviation | USDA ANSI | NOAA statewide code |
|---|---|---|---|
| Illinois | IL | 17 | 011 |
| Indiana | IN | 18 | 012 |
| Iowa | IA | 19 | 013 |
| Kansas | KS | 20 | 014 |
| Michigan | MI | 26 | 020 |
| Minnesota | MN | 27 | 021 |
| Missouri | MO | 29 | 023 |
| Nebraska | NE | 31 | 025 |
| North Dakota | ND | 38 | 032 |
| Ohio | OH | 39 | 033 |
| South Dakota | SD | 46 | 039 |
| Wisconsin | WI | 55 | 047 |

## Reproducibility and attribution

Use a commit-pinned raw URL in the R Markdown so later repository edits do not silently change the inputs. Verify `SHA256SUMS.txt` against downloaded bytes. A mirror does not guarantee network availability, and these snapshots do not reconstruct the data versions available at each historical forecast date.

Cite **USDA NASS Quick Stats** and **NOAA NCEI nClimDiv**, including their original links and snapshot versions, in both the report and R Markdown. Identify this repository separately as the source-file mirror. Do not attribute the official data to the repository owner or present the USDA subset as the complete original database.
