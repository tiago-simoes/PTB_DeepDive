# DeepDive diversity-through-time analyses

Genus-level diversity through time for **Synapsida**, **Reptilia (all)** and **Reptilia (terrestrial)** from the Early Permian to the end of the Triassic estimated with DeepDive (Cooper et al. 2024). DeepDive simulates fossil records under many diversification, spatial and sampling scenarios. It trains a recurrent neural network on these simulations and then uses the network to predict diversity in each time bin from the empirical occurrence data.

Each clade has its own folder with the same structure. All three were run with identical settings. They differ only in their occurrence data and in the values DeepDive's autotune step derived from that data.

## Contents

```
S5_DeepDive/
├── synapsida/        file prefix: syn
├── reptilia_all/     file prefix: rep_all
└── reptilia_terr/    file prefix: rep_terr
```

Inside each clade folder (`<p>` = prefix):

```
<p>.Rmd                    data preparation and DeepDive configuration (R)
<p>_dd_final.csv           occurrence data
<p>_deepdive_input.csv     DeepDive input file (100 age replicates)
<p>_config.ini             DeepDive configuration written by <p>.Rmd
<p>_config_autotuned.ini   configuration after autotuning (the one actually used)
<p>_simulations/           simulated training and test data sets
<p>_models/                trained model, test-set performance, empirical predictions, figures
```

## Data sets

|                                  | Synapsida               | Reptilia (all)              | Reptilia (terrestrial)    |
|----------------------------------|-------------------------|-----------------------------|---------------------------|
| Folder                           | `synapsida`             | `reptilia_all`              | `reptilia_terr`           |
| Occurrences                      | 3,181                   | 3,960                       | 3,135                     |
| Genera                           | 461                     | 682                         | 555                       |
| Localities                       | 91                      | 189                         | 139                       |
| Occurrences in areas 1 / 2 / 3 / 4 | 2,389 / 86 / 408 / 298 | 1,308 / 311 / 1,241 / 1,100 | 1,134 / 172 / 791 / 1,038 |

Reptilia (terrestrial) is Reptilia (all) without 127 genera (823 occurrences). These are mostly marine and other aquatic reptiles (ichthyosaurs, sauropterygians, thalattosaurs, mesosaurs, …) plus pterosaurs. Every occurrence in Reptilia (terrestrial) is also in Reptilia (all).

All taxa are genera. DeepDive's own files and code call them "species" (e.g. `n_species`, `total_sp`).

### Areas

DeepDive calls these "regions". The `Area` column holds the codes.

| Code | Area (label used in figures) | Examples from the data                                                                                                  |
|------|------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| 1    | Gondwana                     | South Africa (Western, Eastern and Northern Cape; Free State), Brazil (Rio Grande do Sul), Argentina (La Rioja), Tanzania (Ruvuma), Madagascar (Tulear), Antarctica (Fremouw) |
| 2    | East Laurasia (EA)           | China (Xinjiang, Shanxi, Gansu, Guizhou, Yunnan, …), Laos                                                               |
| 3    | Central Laurasia (Eu, WA)    | European Russia (Arkhangelsk, Orenburg, Kirov, Perm), Germany, France, England, Switzerland, Italy                       |
| 4    | West Laurasia (NA)           | USA (Texas, Oklahoma, New Mexico, Arizona, …), Canada (Nova Scotia, British Columbia), East Greenland                    |

## Time bins and the age shift

Before the input was built, all ages were shifted by −199 Myr (`MaxAge − 199`, `MinAge − 199`), so that the youngest bin ends at time 0. The Rmd notes that the simulations freeze otherwise. Every time value in the input, configuration and output files is therefore on this shifted scale:

> **Age (Ma) = DeepDive time + 199**

There are 13 bins, roughly at stage level:

| Bin | DeepDive time | Age (Ma)    | Interval                     |
|-----|---------------|-------------|------------------------------|
| 1   | 99.9–91.1     | 298.9–290.1 | Asselian–Sakmarian           |
| 2   | 91.1–84.5     | 290.1–283.5 | Artinskian                   |
| 3   | 84.5–75.4     | 283.5–274.4 | Kungurian                    |
| 4   | 75.4–65.3     | 274.4–264.3 | Roadian–Wordian              |
| 5   | 65.3–60.5     | 264.3–259.5 | Capitanian                   |
| 6   | 60.5–53       | 259.5–252.0 | Wuchiapingian–Changhsingian  |
| 7   | 53–47.7       | 252.0–246.7 | Induan–Olenekian             |
| 8   | 47.7–43       | 246.7–242.0 | Anisian                      |
| 9   | 43–38         | 242.0–237.0 | Ladinian                     |
| 10  | 38–28         | 237.0–227.0 | Carnian                      |
| 11  | 28–18         | 227.0–217.0 | Norian (early)               |
| 12  | 18–9          | 217.0–208.0 | Norian (middle–late)         |
| 13  | 9–0           | 208.0–199.0 | latest Norian–Rhaetian       |

The oldest boundary is the oldest `MaxAge` in each data set, which is 298.9 Ma for all three. The Rmd sets the youngest boundary to 0 (199 Ma) instead of 199.3 Ma.

**Bin order differs between files.** In `<p>_deepdive_input.csv` (columns `t1`–`t13`) bins run from oldest to youngest. In `Empirical_features_.csv`, `Empirical_predictions_.csv` and the simulation `.npy` arrays they run from youngest to oldest.

## Workflow

1. **Prepare the input (R, `<p>.Rmd`, DeepDiveR package).** The script:
   - reads `<p>_dd_final.csv` and keeps `Locality`, `Taxon`, `Area`, `MinAge` and `MaxAge`;
   - shifts the ages by −199 Myr;
   - renames `Area` to `Region`, because `prep_dd_input()` expects that name;
   - runs `prep_dd_input(r = 100)` to write `<p>_deepdive_input.csv`, with 100 replicates in which occurrence ages are resampled within their age ranges;
   - runs `create_config()` and `edit_config()` to write `<p>_config.ini`.
2. **Run DeepDive (Python)** with `python run_dd_config.py <p>_config.ini -wd <clade folder>`, using `run_dd_config.py` from the DeepDive Python repository. Because `autotune = TRUE`, the run does four things:
   - fits the simulation settings to the empirical data and writes `<p>_config_autotuned.ini`;
   - simulates 10,000 training and 1,000 test data sets into `<p>_simulations/`;
   - trains the network and saves it in `<p>_models/`;
   - predicts diversity for each of the 100 empirical replicates (`Empirical_predictions_.csv`).
3. **Draw the final figures.** Each clade has one in `<p>_models/<p>_deepdive_final.pdf`, drawn in R 4.4.1 and finished in Illustrator. The plotting script is not in this folder.

## Settings

These settings are the same for all three clades:

- **Data:** 4 regions, 13 time bins and 100 age replicates.
- **Simulation counts and seeds:** 10,000 training simulations (seed 123) and 1,000 test simulations (seed 432).
- **Extinct clades:** `extant_sp = 0`, `pr_extant_clade = 0`, `present_diversity = NA`.
- **Rates and timing:**
  - mass-extinction probability `p_mass_extinction = 0.02` per Myr, with magnitude 0.5–1;
  - speciation and extinction rate ranges (`rangeL`, `rangeM`) of 0.02–0.5;
  - root age 79.92–99.9 in DeepDive time (278.9–298.9 Ma).
- **Model and training:**
  - LSTM layers of 64 and 32 units, dense layers of 64 and 32 units, and no dropout;
  - up to 1,000 epochs with early stopping (patience 10), batch size 100 and a 20% validation split.
- **Predictions:** one prediction per replicate, 100 replicates.

The inline comments in the Rmd mention 100 training and 10 test simulations. These are leftovers: the values actually set and used are 10,000 and 1,000.

Selected values from the autotuning step (`<p>_config_autotuned.ini`):

| Setting                      | Synapsida  | Reptilia (all) | Reptilia (terrestrial) |
|------------------------------|------------|----------------|------------------------|
| `total_sp` (range)           | 290–11,220 | 260–17,480     | 226–13,940             |
| `dd_k` (range)               | 21–725     | 35–680         | 28–610                 |
| `target_n_occs`              | 3,181      | 3,960          | 3,135                  |
| `min_n_occurrences`          | 636        | 792            | 627                    |
| `maximum_localities_per_bin` | 22         | 44             | 26                     |

## File formats

### `<p>_dd_final.csv` (occurrences)

| Column           | Content                                                                                                                         |
|------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `id`             | record ID                                                                                                                       |
| `occurrence_no`  | occurrence ID. Numeric IDs are Paleobiology Database occurrence numbers. IDs starting with "N" (315 in Synapsida, 211 in each reptile set) are not from the PBDB. |
| `Taxon`          | genus                                                                                                                           |
| `MaxAge`, `MinAge` | age range (Ma)                                                                                                                |
| `Locality`       | locality unit, mostly a state or province; a few are named after formations (e.g. Fremouw)                                      |
| `Area`           | area code 1–4 (see Areas)                                                                                                        |

### `<p>_deepdive_input.csv`

A long table with the columns `Replicate`, `Type`, `Region` and `t1`–`t13` (`t1` is the oldest bin). Each of the 100 replicates has these rows:

- `bin_start`, `bin_mid` and `bin_dur`: bin start, midpoint and duration in DeepDive time;
- `locs`, one row per region: the number of localities in each bin;
- `occs`, one row per genus and region: the number of occurrences in each bin. There are 1,844 such rows per replicate for Synapsida, 2,728 for Reptilia (all) and 2,220 for Reptilia (terrestrial). Genus names are not stored.

### `<p>_simulations/`

The `20260218` in the file names is the name of the simulation run.

| File                                          | Shape          | Content                                                              |
|-----------------------------------------------|----------------|----------------------------------------------------------------------|
| `simulations_20260218_training_features.npy`  | 10,000 × 13 × 21 | features of each simulated fossil record: simulation × bin × feature. These are the same 21 features as in `Empirical_features_.csv`. |
| `simulations_20260218_training_labels.npy`    | 10,000 × 13    | simulated diversity per bin                                         |
| `simulations_20260218_training_totdiv.npy`    | 10,000         | total number of taxa in each simulation                              |
| `simulations_20260218_test_*.npy`             | 1,000 × …      | the same three arrays for the test set                               |

### `<p>_models/`

The model files share the run name `simulations_20260218_lstm64_32_d64_32`, shortened to `…` below.

| File | Content |
|------|---------|
| `rnn_model….keras` | trained network (Keras) |
| `rnn_rescaler….pkl` | feature rescaler used with the model; loading it requires the `deepdive` Python package |
| `rnn_history….pkl` | Python dict with `loss`, `mae`, `val_loss` and `val_mae` per epoch. Training ran for 47 epochs (Synapsida), 49 (Reptilia all) and 37 (Reptilia terrestrial). |
| `history_plot….pdf` | training and validation loss, with the early-stopping point |
| `testset_pred_.npy` | predictions for the 1,000 test simulations (1,000 × 13), on the log(1 + diversity) scale |
| `testset_pred_MSA.txt` | test-set mean squared error on the same scale: 0.199 (Synapsida), 0.158 (Reptilia all), 0.170 (Reptilia terrestrial) |
| `Empirical_features_.csv` | the 21 features computed from the empirical data, averaged over the 100 replicates. There are 13 rows, youngest bin first. |
| `Empirical_predictions_.csv` | predicted genus diversity, as 100 rows (one per replicate) × 14 columns. Each column name is the older boundary of a bin in DeepDive time (add 199 for Ma), and the column holds that bin's prediction; for example, `60.5` is the 259.5–252.0 Ma bin. Column `0.0` repeats the youngest bin (`9.0`) so the values can be drawn as steps. |
| `Empirical_predictions_.pdf` | DeepDive's plot of the predictions, with a time axis in DeepDive time |
| `config_settings.txt` (`config_settings_autotuned.txt` in `synapsida`) | copy of the configuration used for the run, with the same values as `<p>_config_autotuned.ini` |
| `<p>_deepdive_final.pdf` | final figure for the clade (see below) |

Panels of `<p>_deepdive_final.pdf`:

- **A:** extinction and speciation events.
- **B:** raw fossil occurrences by region.
- **C:** net diversification events.
- **D:** diversity through time, from the DeepDive estimates in `Empirical_predictions_.csv`.
- **E:** diversity through time by region, i.e. the genera sampled in each region (`n_species_1.0`–`n_species_4.0` in `Empirical_features_.csv`).

The 21 features are:

- `n_species`, `n_occs`, `n_singletons`, `n_endemics`, `time_bin_duration`, `range_through_div`, `n_localities`, `origination_events`, `extinction_events`;
- for each region *r* = 1.0–4.0: `n_species_r`, `n_occs_r` and `n_locs_r` (e.g. `n_species_1.0`).

## Re-running

- `<p>.Rmd` and `<p>_config.ini` use absolute paths from the lab workstation (`C:/Users/SimoesLabAdmin/Documents/DeepDiveR/updated_occurrences/<clade>`). The autotuned configurations use `.\updated_occurrences\<clade>\`. Update `wd` and the paths in the Rmd before re-running.
- **Software:**
  - DeepDiveR, an R package installed from GitHub (`DeepDive-project/DeepDiveR`);
  - DeepDive for Python (`run_dd_config.py`), which uses TensorFlow/Keras.

  The package versions were not recorded in this folder. DeepDive's PDFs were made with Matplotlib 3.9.2 and the final figures with R 4.4.1.

## Reference

Cooper, R. B., Flannery-Sutherland, J. T. & Silvestro, D. (2024). DeepDive: estimating global biodiversity patterns through time using deep learning. *Nature Communications* 15, 4199. https://doi.org/10.1038/s41467-024-48434-7
