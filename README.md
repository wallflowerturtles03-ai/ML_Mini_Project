# Formula 1 2026 Machine-Learning Mini Project

A Formula 1 machine-learning project that uses 2026 race data and historical 2022–2025 results to model lap pace, tyre performance, pit stops, qualifying order, race finishing order, and points finishes.

The main analysis is in [`Group_E_Notebook.ipynb`](Group_E_Notebook.ipynb). [`Figures_for_f1.ipynb`](Figures_for_f1.ipynb) builds the report figures from the data and model outputs saved by the main notebook. [`libraries.ipynb`](libraries.ipynb) contains a small FastF1 setup example.

## Research objectives

The project evaluates several Formula 1 prediction problems:

- **Lap pace:** estimate lap-time performance from race-lap features.
- **Tyre degradation:** measure whether tyre age predicts increasing lap time.
- **Pit-stop detection:** predict whether a lap is followed by a pit stop.
- **Qualifying:** predict the Q1 gap and the eventual qualifying order.
- **Race finishing position:** rank drivers pre-weekend and after qualifying.
- **Points finish:** classify whether a driver finishes in the top 10.
- **Lap-time consistency:** compare tyre stints by compound.
- **Live-race replay:** replay a race lap by lap through selected models.

## Repository files

| File | Purpose |
|---|---|
| [`Group_E_Notebook.ipynb`](Group_E_Notebook.ipynb) | Complete data collection, feature construction, model training, evaluation, forecasting, and replay pipeline |
| [`Figures_for_f1.ipynb`](Figures_for_f1.ipynb) | Recreates six report figures from saved model outputs |
| [`libraries.ipynb`](libraries.ipynb) | Minimal package installation and FastF1/plotting imports |
| `README.md` | Project documentation |

## Data and features

The notebook collects or reads FastF1 data for 20 rounds of the 2026 season, including:

- Race laps and race results
- Qualifying laps and results
- Practice and sprint sessions
- Pit stops and race-control messages
- Weather and track status
- Tyre compounds and tyre life
- Circuit and extra race information
- Historical qualifying and race results from 2022–2025

The saved workbooks are designed to separate data collection from analysis:

- `F1_2026_races.xlsx`
- `F1_2026_qualifying.xlsx`
- `F1_2026_extra.xlsx`

Most modelling work reads saved parquet and CSV outputs rather than downloading data again. This lets the analysis run without repeating the full FastF1 collection process.

## Notebook pipeline

The main notebook contains 23 numbered cells grouped into eight sections.

| Stage | Notebook section | What it does |
|---|---|---|
| Setup | Section 1 | Sets the project folder, defines a file lookup helper, and fixes the train/validation/test split |
| Data collection | Section 2 | Downloads 2026 race, qualifying, practice, sprint, pit, weather, and historical data |
| Modelling tables | Section 3 | Builds lap-level pace and pit-stop tables and applies modelling cutoffs |
| Historical analysis | Section 4 | Tests 2022–2025 driver-at-circuit affinity, circuit behaviour, and team-mate qualifying ratings |
| In-race models | Section 5 | Models lap pace, tyre degradation, pit-stop likelihood, and stint consistency |
| Forecast models | Section 6 | Builds qualifying and race forecasting models in v1 and walk-forward v2 variants |
| Remaining-race forecast | Section 7 | Forecasts rounds 21–23 before the weekend, after practice, and after qualifying |
| Live-race replay | Section 8 | Replays a test race lap by lap through selected models |

## Evaluation strategy

All models use a time-based evaluation split:

| Split | Rounds |
|---|---:|
| Training | 1–12 |
| Validation | 13–16 |
| Test | 17–20 |

The validation set is used to select a model or threshold. The test set is used once for the final comparison.

Forward forecasting also uses walk-forward evaluation: race *r* is predicted using only races earlier than *r*. For qualifying, the project compares a recent-form baseline, form-only features, and form plus practice data.

## Models

### Lap pace and tyre degradation

- Baseline: average or simple tyre-related reference
- Linear regression
- Ridge regression
- Gradient boosting

### Pit-stop prediction

Because pit stops occur on only a small proportion of laps, PR-AUC is used instead of accuracy alone.

Models compared include:

- Never-pit baseline
- Logistic regression
- Gradient boosting

### Qualifying

The notebook evaluates:

- A recent-form baseline
- A form-only qualifying model
- A model using form and practice/FP3 features

### Race outcome

Race finishing position and race order are modelled with:

- Grid-position baseline
- Ridge or linear regression
- HistGradientBoosting regression
- Plackett-Luce ranking
- Points-finish logistic and gradient-boosting classifiers

The Plackett-Luce implementation uses Monte Carlo simulation to estimate ranking distributions and race outcomes.

## Key outputs

The notebook creates or reads the following types of outputs:

| Output | Description |
|---|---|
| `F1_2026_races.xlsx` | Race laps and results |
| `F1_2026_qualifying.xlsx` | Qualifying laps and results |
| `F1_2026_extra.xlsx` | Practice, sprint, pit-stop, weather, and circuit data |
| `hist_results.parquet` | Historical 2022–2025 qualifying and race data |
| `model_pace.parquet` | Lap-level pace modelling table |
| `model_pit.parquet` | Lap-level pit-stop modelling table |
| `model_quali.parquet` | Qualifying feature table |
| `model_race.parquet` | Driver-per-race feature table |
| `results_model*.csv` | Model comparisons |
| `saved_models/*` | Fitted model artefacts |
| `figures/` | Saved report visualisations |
| `site_export/replay_*` | Live-race replay exports |

## Report figures

[`Figures_for_f1.ipynb`](Figures_for_f1.ipynb) recreates six figures:

1. Race-lap coverage and train/validation/test split
2. Random-split versus time-split leakage check
3. Pit-stop detection PR-AUC
4. Qualifying walk-forward comparison
5. Race-winner and ranking walk-forward comparison
6. Lap-time consistency across tyre compounds

Run the main notebook first. The figure notebook reads its saved CSV, parquet, Excel, JSON, and model files and does not refit the models.

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/wallflowerturtles03-ai/ML_Mini_Project.git
cd ML_Mini_Project
```

### 2. Create a Python environment

Python is used through Jupyter. A modern Python environment with Jupyter and the packages below is sufficient:

```bash
python -m pip install --upgrade pip
python -m pip install jupyter fastf1 openpyxl pyarrow pandas numpy matplotlib scipy scikit-learn joblib
```

### 3. Start Jupyter

```bash
jupyter notebook
```

Open [`Group_E_Notebook.ipynb`](Group_E_Notebook.ipynb).

## Running the project

Run the cells in order. The main notebook has a setup cell at the beginning that expects a local project folder.

For analysis and reproduction of the reported results:

1. Run Cell 1.
2. Skip the data-collection section if the three Excel workbooks are already available.
3. Continue through the modelling sections in order.
4. Run [`Figures_for_f1.ipynb`](Figures_for_f1.ipynb) after the model outputs have been created.

The notebook searches for saved files from the project directory and a `Data` subdirectory. It also searches upwards for some figure inputs.

A Windows-style project path is used in the notebook:

```python
BASE = r"C:\path\to\ML_Mini_Project"
```

Change `BASE` to the local directory containing this repository and, if applicable, its `Data` folder.

## FastF1 authentication

The data-collection cells require FastF1 authentication for some 2026 sessions. Credentials can be supplied through environment variables:

```bash
export F1_USERNAME="your-email@example.com"
export F1_PASSWORD="your-password"
```

PowerShell:

```powershell
$env:F1_USERNAME="your-email@example.com"
$env:F1_PASSWORD="your-password"
```

You can instead enter the credentials when prompted. Credentials are not written to the notebook.

FastF1 may limit requests and caches downloaded sessions locally. Cells that write existing output files skip themselves, making data collection resumable.

## Limitations

- The 2026 rule reset makes older car and team performance less representative.
- Historical team-mate comparisons may not fully capture new driver or car combinations.
- Only a limited number of races are available for walk-forward evaluation.
- DNFs and unusual race events can strongly affect ranking and points-finish metrics.
- Pit-stop detection is affected by heavy class imbalance.
- Post-qualifying forecasts require real qualifying/grid data.
- Some outputs depend on files generated locally, such as parquet and Excel datasets.
- The project contains machine-learning baselines rather than a production prediction service.

## Licence

No licence has been specified in this repository. Copyright remains with the repository owner unless an explicit licence is added.

## Authors

- Yeshashwini K.
- Yug Bhogi
