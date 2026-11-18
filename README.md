# ML_Mini_Project
# F1 2026 Driver Finishing-Position Prediction

A machine-learning mini-project that predicts a Formula 1 driver's finishing position using qualifying, race, practice, weather, team-form, and historical data.

The project uses time-based evaluation: earlier races train the models, while later races are used for validation and testing. This prevents future race information from leaking into predictions.

## Research Question

**Can we predict a driver's finishing position before a race?**

The project provides two prediction stages:

- **Pre-weekend:** driver form, team form, championship points, and historical ratings
- **After qualifying:** pre-weekend features plus grid position and qualifying performance

## Repository Contents

- `F1_Problem6_final.ipynb` — complete data collection and prediction pipeline
- `libraries.ipynb` — dependency setup and imports
- `README.md` — project documentation

## Methodology

### 1. Data Collection

The notebook uses `FastF1` to collect:

- 2026 race and qualifying results
- Race laps and weather data
- Historical qualifying and race results from 2022–2025
- Practice and sprint sessions
- Pit stops and race-control messages
- Track status and circuit information
- Pirelli tyre-compound and circuit data

Downloaded data is cached locally, and existing files are reused instead of downloaded again.

### 2. Feature Engineering

Each row represents one **driver in one race**.

Available features include:

- Grid position
- Gap to the fastest qualifying lap
- Sprint-race position
- Practice best-lap and long-run gaps
- Team tyre degradation
- Driver average finish
- Driver points and DNF rate
- Team points, average finish, and DNF rate
- Rain risk and pit-lane duration
- Historical 2022–2025 driver rating

Form features are calculated only from earlier rounds. A race never sees its own result.

### 3. Time-Based Splits

| Dataset | Rounds |
|---|---:|
| Training | 1–12 |
| Validation | 13–16 |
| Test | 17–20 |

The validation set selects the model, while the test set is used for the final score.

## Models

### Model 6 v1: Regression

Finishing position is predicted using:

- Grid-position baseline
- Ridge regression
- HistGradientBoosting regression

It is evaluated using:

- Mean absolute error
- Spearman rank correlation
- R² score
- Finisher-only MAE

### Model 6 v2: Plackett-Luce Ranking

The ranking model learns the likely order of finish rather than minimizing average numeric error.

Monte Carlo simulation with Gumbel noise estimates:

- Probability of winning
- Probability of a podium finish
- Probability of a points finish
- Expected finishing position

Two versions are tested:

- **Pre-weekend:** form and historical features
- **After qualifying:** pre-weekend features plus grid position and qualifying data

### Model 7: Points-Finish Classification

This model predicts whether a driver finishes in the top 10.

Compared models:

- Grid-position baseline
- Logistic regression
- HistGradientBoosting classification

Evaluation metrics include accuracy, precision, recall, F1-score, and ROC-AUC.

## Evaluation

Models are tested using walk-forward prediction across races 8–20. For every race, the model is trained using only earlier races.

Reported metrics include:

- Mean absolute error
- Spearman rank correlation
- Winner-pick accuracy
- Podium-hit count
- Points-finish ROC-AUC
- Average log probability of the actual winner

A model must beat the simple `finish = grid position` baseline to be considered useful.

## Outputs

The notebook creates or saves:

| Output | Description |
|---|---|
| `F1_2026_races.xlsx` | 2026 race laps and results |
| `F1_2026_qualifying.xlsx` | 2026 qualifying laps and results |
| `F1_2026_extra.xlsx` | Practice, sprint, pit, weather, and circuit data |
| `hist_results.parquet` | 2022–2025 qualifying and race history |
| `model_race.parquet` | Final driver-per-race feature table |
| `model6_finish.pkl` | Selected finishing-position model |
| `model7_points.pkl` | Selected points-finish model |
| `model6_pl_pre.pkl` | Pre-weekend ranking model |
| `model6_pl_post.pkl` | Post-qualifying ranking model |
| `results_model6.csv` | Model 6 v1 comparison |
| `results_model7.csv` | Model 7 comparison |
| `results_model6_v2.csv` | Walk-forward ranking comparison |
| `results_model6_v2_per_race.csv` | Predicted versus actual race results |

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/wallflowerturtles03-ai/ML_Mini_Project.git
cd ML_Mini_Project
```
### 2. Install Dependencies

The notebook installs the main packages directly:

```bash
pip install fastf1 openpyxl pyarrow scipy scikit-learn joblib
```
For Jupyter:
```bash
pip install jupyter
jupyter notebook
```
### Running the Notebook

Open F1_Problem6_final.ipynb and run the cells sequentially.

```bash
jupyter notebook F1_Problem6_final.ipynb
```
The notebook stages are:

Setup and project-folder discovery
Race, qualifying, and historical data collection
Feature-table construction
Historical-driver-rating test
Regression and points-finish models
Plackett-Luce ranking models
Predicted-versus-actual race analysis

### FastF1 Authentication
Some data collection cells require an F1 account.

Set the credentials as environment variables, or enter them when prompted:
```bash
export F1_USERNAME="your-email@example.com"
export F1_PASSWORD="your-password"
```
Windows Powershell:

```Powershell
$env:F1_USERNAME="your-email@example.com"
$env:F1_PASSWORD="your-password"
```
Credentials are not stored in the notebook. Downloaded cells skip themselves when the corresponding files already exist.

The notebook searches common project locations. The project directory can also be specified manually:

```python
os.environ["F1_BASE"] = r"C:\path\to\ML_Mini_Project"
```
### Limitations
* The 2026 rule reset reduces the usefulness of older team and car performance.
* Historical driver ratings rely on qualifying comparisons against team-mates.
* The number of available races is limited.
* DNF and unusual-race outcomes can strongly affect the metrics.
* FastF1 API limits or expired authentication can interrupt data collection.
* The notebook assumes that real qualifying/grid data is available for post-qualifying predictions.

### License
No license has been specified for this repository. All rights remain reserved unless the owner explicitly adds a license.

### Author
Yeshashwini K. & Yug Bhogi



