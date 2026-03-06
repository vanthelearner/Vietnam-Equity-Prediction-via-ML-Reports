
## 1) System Overview

> [!info]
> This is the top-level view.

```mermaid
flowchart LR
    A["Pipeline 1<br/>Pull Raw Data<br/>Bloomberg -> raw CSVs"]
    B["Pipeline 2<br/>Process Raw Data<br/>clean, filter, engineer, split"]
    C["Pipeline 3<br/>Model Pipeline<br/>monthly translation, rolling train/test, outputs"]

    A --> B --> C

    A1["raw_stock_data.csv<br/>raw_macro_data.csv"] --> B
    B1["model_data_v2_balanced.csv<br/>train/val/test splits<br/>FF diagnostics"] --> C
    C1["predictions<br/>R2<br/>importance<br/>portfolio<br/>benchmark compare"]

    C --> C1
```

## 2) File Lineage

> [!info]
> This view focuses on the main file handoffs.

```mermaid
flowchart TB
    subgraph P1["Pipeline 1 Files"]
        P1S["raw_data/raw_stock_data.csv<br/>2,437,880 rows<br/>699 tickers"]
        P1M["raw_data/raw_macro_data.csv<br/>6,872 rows<br/>25 columns"]
    end

    subgraph P2["Pipeline 2 Files"]
        P2T["outputs/_intermediate/vn_transform.csv"]
        P2C["outputs/01_data_processing/stock_data_clean_v2_balanced.csv<br/>902,959 rows<br/>590 tickers"]
        P2MC["outputs/02_macro/macro_data_v2_balanced.csv"]
        P2MD["outputs/03_model_data/model_data_v2_balanced.csv<br/>787,953 rows<br/>441 tickers"]
        P2S["outputs/04_model_split/train val test CSVs"]
        P2B["outputs/07_market_portfolios/market_portfolio_daily_returns.csv<br/>8,850 rows"]
    end

    subgraph P3["Pipeline 3 Files"]
        P3RF["data/risk-free.csv<br/>FRED DGS3MO"]
        P3P["data/panel_input.csv<br/>39,427 rows<br/>437 assets<br/>270 months"]
        P3BM["data/benchmark_monthly.csv<br/>71 months"]
        P3RUN["outputs/run_*/...<br/>model results"]
    end

    P1S --> P2T --> P2C --> P2MD
    P1M --> P2MC --> P2MD
    P2MD --> P2S
    P2MD --> P3P
    P2B --> P3BM
    P3RF --> P3P
    P3P --> P3RUN
    P3BM --> P3RUN
```

## 4) Filtering Funnel

> [!info]
> This is the best view for understanding where the stock sample gets cut down.

```mermaid
flowchart TB
    A["Raw stock panel<br/>2,437,880 rows<br/>699 tickers<br/>100.0% of raw"]
    B["After transform<br/>2,437,880 rows<br/>699 tickers<br/>100.0% of raw"]
    C["After process_stock<br/>902,959 rows<br/>590 tickers<br/>37.0% of raw"]
    D["After build_model_data<br/>787,953 rows<br/>441 tickers<br/>32.3% of raw"]
    E["Train split<br/>404,311 rows<br/>441 tickers"]
    F["Validation split<br/>98,762 rows<br/>293 tickers"]
    G["Test split<br/>284,880 rows<br/>313 tickers"]

    A --> B --> C --> D
    D --> E
    D --> F
    D --> G
```


```mermaid
pie showData
    title Pipeline 2 Split of Final Model Data Rows
    "Train" : 404311
    "Validation" : 98762
    "Test" : 284880
```


## 5) Pipeline 3 Preparation and Rolling Training Logic

> [!info]
> Pipeline 3 does not train directly on Pipeline 2 daily rows. It first converts them into a monthly panel, then applies another preprocessing layer, then rolls windows through time.

```mermaid
flowchart LR
    A["Pipeline 2 daily model panel<br/>787,953 rows<br/>441 tickers"]
    B["Prepare-input notebook<br/>group by ticker-month<br/>compound y_next_1d<br/>take month-end snapshot"]
    C["panel_input.csv<br/>39,427 rows<br/>437 assets<br/>270 months"]
    D["preprocess.py<br/>price/size filters<br/>coverage filter<br/>monthly median fill<br/>winsorize<br/>rank-scale"]
    E["Full sample<br/>38,990 rows<br/>432 assets"]
    F["Rolling windows<br/>60m train<br/>24m val<br/>12m test<br/>step 12m<br/>15 windows"]
    G["Per-window model loop<br/>fit train<br/>choose on val<br/>refit train+val<br/>predict test"]

    A --> B --> C --> D --> E --> F --> G
```

```mermaid
flowchart LR
    W1["Window k<br/>Train 60 months"] --> W2["Validate 24 months"]
    W2 --> W3["Test 12 months"]
    W3 --> W4["Shift forward 12 months"]
    W4 --> W5["Build Window k+1"]
    W5 --> W1
```


Rolling Window Logic (Pipeline 3)

| Window | Train start | Train end  | Val start  | Val end    | Test start | Test end   |
| -----: | ----------- | ---------- | ---------- | ---------- | ---------- | ---------- |
|      1 | 2003-08-31  | 2008-07-31 | 2008-08-31 | 2010-07-31 | 2010-08-31 | 2011-07-31 |
|      2 | 2004-08-31  | 2009-07-31 | 2009-08-31 | 2011-07-31 | 2011-08-31 | 2012-07-31 |
|      3 | 2005-08-31  | 2010-07-31 | 2010-08-31 | 2012-07-31 | 2012-08-31 | 2013-07-31 |
|      4 | 2006-08-31  | 2011-07-31 | 2011-08-31 | 2013-07-31 | 2013-08-31 | 2014-07-31 |
|      5 | 2007-08-31  | 2012-07-31 | 2012-08-31 | 2014-07-31 | 2014-08-31 | 2015-07-31 |
|      6 | 2008-08-31  | 2013-07-31 | 2013-08-31 | 2015-07-31 | 2015-08-31 | 2016-07-31 |
|      7 | 2009-08-31  | 2014-07-31 | 2014-08-31 | 2016-07-31 | 2016-08-31 | 2017-07-31 |
|      8 | 2010-08-31  | 2015-07-31 | 2015-08-31 | 2017-07-31 | 2017-08-31 | 2018-07-31 |
|      9 | 2011-08-31  | 2016-07-31 | 2016-08-31 | 2018-07-31 | 2018-08-31 | 2019-07-31 |
|     10 | 2012-08-31  | 2017-07-31 | 2017-08-31 | 2019-07-31 | 2019-08-31 | 2020-07-31 |
|     11 | 2013-08-31  | 2018-07-31 | 2018-08-31 | 2020-07-31 | 2020-08-31 | 2021-07-31 |
|     12 | 2014-08-31  | 2019-07-31 | 2019-08-31 | 2021-07-31 | 2021-08-31 | 2022-07-31 |
|     13 | 2015-08-31  | 2020-07-31 | 2020-08-31 | 2022-07-31 | 2022-08-31 | 2023-07-31 |
|     14 | 2016-08-31  | 2021-07-31 | 2021-08-31 | 2023-07-31 | 2023-08-31 | 2024-07-31 |
|     15 | 2017-08-31  | 2022-07-31 | 2022-08-31 | 2024-07-31 | 2024-08-31 | 2025-07-31 |
```mermaid
gantt
    title Rolling Window Logic (Pipeline 3)
    dateFormat  YYYY-MM-DD
    axisFormat  %Y-%m

    section Window 1
    Train W1 :done, w1tr, 2003-08-31, 2008-07-31
    Val W1   :done, w1va, 2008-08-31, 2010-07-31
    Test W1  :active, w1te, 2010-08-31, 2011-07-31

    section Window 2
    Train W2 :done, w2tr, 2004-08-31, 2009-07-31
    Val W2   :done, w2va, 2009-08-31, 2011-07-31
    Test W2  :active, w2te, 2011-08-31, 2012-07-31

    section Window 3
    Train W3 :done, w3tr, 2005-08-31, 2010-07-31
    Val W3   :done, w3va, 2010-08-31, 2012-07-31
    Test W3  :active, w3te, 2012-08-31, 2013-07-31

    section Window 4
    Train W4 :done, w4tr, 2006-08-31, 2011-07-31
    Val W4   :done, w4va, 2011-08-31, 2013-07-31
    Test W4  :active, w4te, 2013-08-31, 2014-07-31

    section Window 5
    Train W5 :done, w5tr, 2007-08-31, 2012-07-31
    Val W5   :done, w5va, 2012-08-31, 2014-07-31
    Test W5  :active, w5te, 2014-08-31, 2015-07-31

    section Window 6
    Train W6 :done, w6tr, 2008-08-31, 2013-07-31
    Val W6   :done, w6va, 2013-08-31, 2015-07-31
    Test W6  :active, w6te, 2015-08-31, 2016-07-31

    section Window 7
    Train W7 :done, w7tr, 2009-08-31, 2014-07-31
    Val W7   :done, w7va, 2014-08-31, 2016-07-31
    Test W7  :active, w7te, 2016-08-31, 2017-07-31

    section Window 8
    Train W8 :done, w8tr, 2010-08-31, 2015-07-31
    Val W8   :done, w8va, 2015-08-31, 2017-07-31
    Test W8  :active, w8te, 2017-08-31, 2018-07-31

    section Window 9
    Train W9 :done, w9tr, 2011-08-31, 2016-07-31
    Val W9   :done, w9va, 2016-08-31, 2018-07-31
    Test W9  :active, w9te, 2018-08-31, 2019-07-31

    section Window 10
    Train W10 :done, w10tr, 2012-08-31, 2017-07-31
    Val W10   :done, w10va, 2017-08-31, 2019-07-31
    Test W10  :active, w10te, 2019-08-31, 2020-07-31

    section Window 11
    Train W11 :done, w11tr, 2013-08-31, 2018-07-31
    Val W11   :done, w11va, 2018-08-31, 2020-07-31
    Test W11  :active, w11te, 2020-08-31, 2021-07-31

    section Window 12
    Train W12 :done, w12tr, 2014-08-31, 2019-07-31
    Val W12   :done, w12va, 2019-08-31, 2021-07-31
    Test W12  :active, w12te, 2021-08-31, 2022-07-31

    section Window 13
    Train W13 :done, w13tr, 2015-08-31, 2020-07-31
    Val W13   :done, w13va, 2020-08-31, 2022-07-31
    Test W13  :active, w13te, 2022-08-31, 2023-07-31

    section Window 14
    Train W14 :done, w14tr, 2016-08-31, 2021-07-31
    Val W14   :done, w14va, 2021-08-31, 2023-07-31
    Test W14  :active, w14te, 2023-08-31, 2024-07-31

    section Window 15
    Train W15 :done, w15tr, 2017-08-31, 2022-07-31
    Val W15   :done, w15va, 2022-08-31, 2024-07-31
    Test W15  :active, w15te, 2024-08-31, 2025-07-31

```


## 6) Output Taxonomy

> [!info]
> This view is for understanding where the model results are saved and what each output family means.

```mermaid
flowchart TD
    A["Pipeline 3 run folder<br/>outputs/run_YYYYMMDD..."]

    A --> B["predictions/<br/>yhat, y_true, per-window prediction files"]
    A --> C["r2/<br/>OOS R2 summaries by model and sample"]
    A --> D["importance/<br/>variable importance and complexity"]
    A --> E["portfolio/<br/>deciles, long-only, long-short, turnover, net return"]
    A --> F["benchmark/<br/>strategy vs benchmark comparisons"]
    A --> G["compare/<br/>cross-model tables, DM results, merged summaries"]
    A --> H["windows/<br/>chosen parameters and per-window metrics"]
```


