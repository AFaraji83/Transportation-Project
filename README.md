# Uber & Lyft in Boston: pricing, trip distribution and a small revenue model

Our course project for **Transportation Planning** (Industrial Engineering, Sharif University of Technology, fall 2025).
Everything lives in one notebook, [`transportation_project.ipynb`](transportation_project.ipynb), which runs from the raw csv to the last figure without any manual steps.

The data is Kaggle's [Uber and Lyft Dataset Boston, MA](https://www.kaggle.com/datasets/brllrb/uber-and-lyft-dataset-boston-ma): about 693k price quotes for 12 neighbourhoods in central Boston between 26 Nov and 18 Dec 2018, each with the weather at that hour.

## What's in the notebook

**Part A (phase 1)**
- A first look at the data: missing values, the routes that appear, which services each company offers, encoding
- Statistical tests on price: ANOVA / Welch / Kruskal-Wallis, Tukey and Games-Howell, normality tests, outlier rules
- Weather: PCA and k-means on the weather records, and predicting rain from the other weather variables
- Price models: feature selection (mutual information, RFECV, Lasso), linear regression, random forest, XGBoost, Taguchi L9 vs full grid search, stacking and other hybrids

**Part B (phase 2)**
- OD matrices, overall and per weekday, and an inflow/outflow balance check
- Trip distribution with Furness, Detroit, Fratar and a doubly constrained gravity model (exponential and power deterrence)
- An integer program that plans trips for a new competitor with a 20 km/day fleet limit

## A few things we found

- **The file isn't a trip log.** It's prices that someone scraped from the apps on a fixed schedule. Once you divide by the number of dates in the file, the weekday × hour "demand" heatmap is flat: 156 records per zone per hour. Any count of "trips" in this dataset is really a count of quotes.
- **Price is almost entirely service type + distance + surge.** Service type alone explains about 77% of the variance. Weather, hour and date have no measurable effect.
- **Best price model: XGBoost**, test R² 0.976 and MAE about $0.95. Taguchi found a near-optimal setting in 9 runs where the grid needed 81. Stacking matched XGBoost but didn't beat it.
- **Rain can be predicted** from visibility, cloud cover, humidity and dew point: random forest with F1 ≈ 0.72 and ROC-AUC ≈ 0.95 on days it never saw. This only holds once `precipProbability` and the text summaries are removed, because they give the answer away.
- **The future trip targets can't be met exactly on this network.** Trips only run between two groups of six zones, and the targets break that balance by 0.34%. That's why every distribution method "converges" in one or two iterations at 1% tolerance. With network-consistent targets, Furness is fastest and Detroit stalls at about 0.04% error.
- **Revenue plan:** with only 20 km a day, the best plan runs Financial District → South Station over and over (about $1,830 a week). If the car has to come back to the zones it drives into, that drops to about $1,520.

| Price by service type | Records per weekday-hour, normalised |
|---|---|
| ![](figures/A2_3_price_by_name.png) | ![](figures/A2_8_heatmap.png) |

| All price models on the same test set | Convergence of the distribution methods |
|---|---|
| ![](figures/A4_6_all_models.png) | ![](figures/B2_convergence.png) |

## Running it

1. Download `rideshare_kaggle.csv` from the Kaggle link above and put it in `data/`. It's about 200 MB, so it isn't in the repo.
2. Install the packages:
   ```bash
   pip install -r requirements.txt
   ```
3. Open `transportation_project.ipynb` and run all cells. It takes around 10 minutes on a laptop, mostly the grid search in A-4.5.

Figures go to `figures/`, and the OD matrices and the trip plan go to `outputs/` as Excel files.

Our course gave us a trimmed copy of the Kaggle file with 39 of the 57 columns. The notebook only uses columns that are in both versions, so the full Kaggle file works too, though a few counts (duplicate rows, for example) may come out slightly different.

## Layout

```
├── transportation_project.ipynb   the whole project
├── requirements.txt
├── data/                          put rideshare_kaggle.csv here (git-ignored)
├── figures/                       every plot the notebook makes
└── outputs/                       OD matrices, distribution results, trip plan (xlsx)
```

## Team

Amirabbas Donyadideh, Arshia Sattari Khavas, Alireza Faraji. The course was taught by Dr. Hassannayebi.
