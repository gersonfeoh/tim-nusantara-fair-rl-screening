# When to Ask and When to Refer: code for a fairness-audited Q-learning agent for depression screening

Code and result files for the manuscript "When to Ask and When to Refer: Fairness-Audited Reinforcement Learning for Depression Screening" (Tim Nusantara, Doctoral Program in Information Systems, Universitas Komputer Indonesia). The manuscript is being prepared for Jurnal Sinkron. Link to the published paper: [to be added after publication].

Archived release (v1.0.1): [https://doi.org/10.5281/zenodo.24536284](https://doi.org/10.5281/zenodo.24536284). Previous release (v1.0.0): [https://doi.org/10.5281/zenodo.24500109](https://doi.org/10.5281/zenodo.24500109). All versions: [https://doi.org/10.5281/zenodo.24500108](https://doi.org/10.5281/zenodo.24500108).

## What this repository does

A short depression questionnaire saves time but misses some respondents who would score as severe on the full form. This code simulates a screening service in which a tabular Q-learning agent chooses, one step at a time, among three actions:

| Action | Meaning |
|---|---|
| STOP | Use the result from the answers seen so far. |
| ASK | Request one more item from the hidden items (at most three). |
| REFER | Send the respondent to a counselor, whose capacity is limited. |

The agent starts from the seven depression items of the 21-item Depression Anxiety Stress Scales (DASS-21). Hidden items come from the 42-item version (DASS-42). The reference label is the severity band computed from 14 DASS-42 depression items. Policies are trained with several referral prices (lambda), and one is selected on validation data under two rules fixed in advance: the counselor reviews at most 10% of respondents, and the age gap in missed severe cases stays within one percentage point. The locked policy is then evaluated on a held-out test set against comparison policies that receive the same number of counselor reviews.

## Repository layout

```
notebooks/   Colab notebooks (see below)
data/        empty; the raw data are NOT redistributed (see Data)
results/     result files written by the notebooks
requirements.txt
CITATION.cff
.zenodo.json
LICENSE
```

| Notebook | Purpose |
|---|---|
| `UjiCoba_v4.2_Agen3Aksi_TimNusantara.ipynb` | Main notebook: data cleaning, reference bands, prediction models for 7 to 10 items, Q-learning agent, policy selection, test evaluation, and the ask-cost demo. Runs the single split with seed 42. |
| `Audit_v4.2_TimNusantara.ipynb` | Audit notebook: repeats the pipeline over seven random splits (seeds 1, 2, 3, 4, 5, 6, 42), runs the fairness and sensitivity checks, and writes `cek_tambahan_v42.json`, which stores the results and the software versions used. |
| `Audit_v4.2_GPU_T4_pembanding.ipynb` | Comparison only, not a reference. The same audit code executed on the Colab GPU T4 runtime (Python 3.13.16) on 25 November 2026, kept with its outputs to document runtime sensitivity (see Numerical sensitivity). Do not use its numbers for reporting, and do not rerun it. |

## Data

The data are the open DASS responses of the Open-Source Psychometrics Project: https://openpsychometrics.org/_rawdata/ (file `DASS_data_21.02.19.zip`). The notebooks download this file themselves. The source page states no license, citation, or redistribution terms, so the file is not copied into this repository; please cite the source when reusing the data.

Cleaning follows Step 0 and Step 0b of the main notebook: 39,775 raw rows are reduced to 28,474 after a validity check on fictitious words and an age check. The notebook prints the number of rows removed at each stage.

The data are international, voluntary, online respondents and not Indonesian students. The agent is a simulation study of a decision method, not a validated clinical tool.

## How to run

Google Colab is the reference platform, because the reported numbers come from Colab runs.

1. Open a notebook from the `notebooks` folder in Google Colab.
2. Choose Runtime, Change runtime type, CPU, then Run all. Use the CPU runtime for both notebooks; the GPU runtime runs on a different machine and gives slightly different numbers.
3. The audit notebook runs seven splits; in the stored run it took about 3 minutes (193 seconds). The main notebook runs one split.

Results are written to the folder named in the first cell of each notebook.

For a local run, create an environment with Python 3.13 and install the pinned packages:

```
python -m venv .venv
pip install -r requirements.txt
```

## Expected output for seed 42 (main notebook)

On Colab, the main notebook prints:

| Item | Expected value |
|---|---|
| Rows after cleaning | 28,474 (174 columns; all source columns are retained) |
| Selected policy | lambda = 0.90, ask cost = 0.10 |
| Counselor reviews on the test set | 480 of 5,695 participants (8.4%) |

The manuscript reports the seven-split averages from the audit notebook (abstract): the agent referred 7.2% of respondents and missed 13.1 severe cases per test set, against 37.6 for referral without asking and 22.0 for a fixed ask-then-refer rule, with 0.54 extra items asked per respondent. A rule that asks everyone three items missed 9.9. The age gap in missed severe cases did not exceed 0.79 percentage points.

## Numerical sensitivity

With identical package versions, the number of counselor reviews for seed 42 differed between platforms in our tests (480 on the Colab CPU runtime, 477 on the Colab GPU T4 runtime and on another x86_64 machine). The cause is floating-point behavior of the underlying linear algebra libraries on different CPUs, which can change a few borderline decisions. Small differences of a few respondents on a single split are therefore expected outside the Colab CPU runtime. The conclusions of the manuscript rest on averages over seven splits, not on one count.

The GPU T4 comparison run (`notebooks/Audit_v4.2_GPU_T4_pembanding.ipynb`) used the same code and package versions; only the Python patch release and the machine differed. Our code does not use the GPU, so the difference comes from the machine, not from the GPU itself. Its seven-split averages were 13.7 missed severe cases for the agent, 22.0 for the fixed rule, 49.3 for referral without asking, and 11.0 for asking everyone three items, against 13.1, 22.0, 37.6, and 9.9 on the CPU reference run. The ranking of the policies was the same, and the agent missed fewer cases than the fixed rule in all seven splits on both runtimes. Referral without asking changed most, because two splits selected a different lambda on the GPU runtime, which reduced the shared counselor budget.

Package versions recorded by the audit notebook in its Colab CPU run: Python 3.13.18, numpy 2.4.6, pandas 2.2.3, scipy 1.16.3, scikit-learn 1.6.1. In that run, seed 42 gave 480 counselor reviews and all seven splits matched the numbers reported in the manuscript.

## Limitations

1. The reference label is a score band from 14 DASS-42 items, not a clinical diagnosis, and it shares items with the inputs the agent can request.
2. The counselor is assumed to be always right in the simulation.
3. All data come from one open archive of voluntary online respondents; no external validation was done.
4. The age limit did not change which policy was selected, so fairness was audited and not enforced.

## Ringkasan dalam bahasa Indonesia

Repositori ini memuat kode dan berkas hasil untuk naskah tentang agen Q-learning tiga aksi (BERHENTI, TANYA, ke konselor) pada skrining depresi dengan data terbuka DASS. Data mentah tidak disertakan dan diunduh langsung dari Open-Source Psychometrics Project. Colab dengan runtime CPU adalah platform acuan; berkas `Audit_v4.2_GPU_T4_pembanding.ipynb` hanya pembanding dari runtime GPU T4. Hasil satu split (seed 42) dapat berbeda beberapa responden pada mesin lain, sehingga kesimpulan naskah bertumpu pada rata-rata tujuh split.

## How to cite

Please cite the archived release through the DOI above, and the manuscript once published. See `CITATION.cff`.

## License

Code is released under the MIT License (see `LICENSE`). The DASS data belong to the Open-Source Psychometrics Project and are not covered by this license.
