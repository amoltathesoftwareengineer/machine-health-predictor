# machine-health-predictor

Predictive maintenance for heavy machinery. Learns failure signatures from vibration, temperature and usage data to forecast remaining useful life and schedule maintenance before breakdowns.

> **Status:** early development. The data pipeline, models and benchmarks are being built in the open. See the [Roadmap](#roadmap).

---

## Why this exists

Unplanned downtime of heavy machinery (excavators, loaders, haul trucks, compressors, pumps) is expensive. A single breakdown can stop a whole site, while replacing parts too early wastes money. Maintenance today is usually one of two things:

- **Reactive:** fix it after it breaks.
- **Calendar-based:** service every N hours, whether it needs it or not.

Predictive maintenance aims for a third option: **service a machine when its own sensor data says it is wearing out.**

Open projects in this area often have the same gaps:

- Different datasets, splits and metrics, so results are **hard to compare**.
- Models are scored with plain accuracy, which hides the real cost of **late warnings and false alarms**.
- Train and test data sometimes share the same machine, which makes results look better than they are.

`machine-health-predictor` aims to give a clean, reproducible starting point:

- **Unified loaders** for public run-to-failure datasets.
- **Leakage-safe splits** (train and test never share the same machine).
- **Failure-aware metrics** that reward early, useful warnings and penalise missed failures.
- **Multiple model families** compared under the same protocol.
- **One command** to reproduce every reported result.

---

## What it predicts

| Task | Meaning |
|---|---|
| **Anomaly detection** | Flag when a machine's sensor behaviour drifts away from healthy normal. |
| **Failure prediction** | Estimate the chance of failure within the next N hours or cycles. |
| **RUL** (Remaining Useful Life) | Estimate how many hours or cycles are left before failure. |
| **Maintenance planning** | Suggest a service window that balances breakdown risk against early-replacement cost. |

---

## Data

The project is built around openly available run-to-failure and condition-monitoring datasets, such as:

- NASA C-MAPSS turbofan engine degradation dataset
- NASA / IMS bearing run-to-failure dataset
- CWRU bearing fault dataset (Case Western Reserve University)

Public datasets from real heavy-machinery fleets are rare, so these are used as standard research benchmarks for the same kinds of rotating, wearing components found in heavy machines. Datasets are **not** redistributed in this repository. Scripts download them from the original sources, and each dataset keeps its own license and citation requirements.

---

## Planned approach

### Models compared

- **Baselines:** threshold rules and simple trend fits
- **Anomaly detection:** Isolation Forest, autoencoders
- **Classical ML:** gradient boosting and random forest on engineered features (vibration RMS, kurtosis, spectral features, rolling statistics)
- **Deep learning:** LSTM / GRU / 1D-CNN sequence models for RUL

### Metrics

- **RUL error:** MAE and RMSE in cycles or hours
- **Asymmetric score:** penalises late predictions more than early ones (as used in the C-MAPSS literature)
- **Warning lead time:** how early a useful alert is raised before failure
- **False alarm rate:** alerts raised on healthy machines
- **Cost-based view:** a simple cost model comparing breakdown cost against early maintenance cost

---

## How it works

```
raw sensor data (vibration, temperature, pressure, usage hours)
                    |
                    v
        cleaning + windowing + feature extraction
                    |
                    v
      leakage-safe split (by machine, not by row)
                    |
                    v
   model (baseline / anomaly / ML / deep learning)
                    |
                    v
   health score + failure risk + RUL estimate
                    |
                    v
      metrics + plots + maintenance recommendation
```

---

## Planned repository structure

```
machine-health-predictor/
├── data/
│   ├── raw/              # downloaded datasets (git-ignored)
│   └── processed/        # cleaned features
├── machine_health/
│   ├── datasets/         # loaders for each public dataset
│   ├── features.py       # signal and rolling-window features
│   ├── models/           # baselines, anomaly, ML and deep models
│   ├── maintenance.py    # cost model and service-window logic
│   └── evaluate.py       # metrics
├── benchmarks/           # scripts and saved results
├── notebooks/            # exploration and plots
├── tests/
├── docs/
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## Quickstart (planned)

```bash
git clone https://github.com/amoltathesoftwareengineer/machine-health-predictor.git
cd machine-health-predictor
pip install -r requirements.txt

# download and prepare a dataset
python -m machine_health.datasets.download --name cmapss

# train a model and predict RUL
python -m machine_health.train --model gbm --dataset cmapss

# reproduce the benchmark table
python -m benchmarks.run_all
```

---

## Results

Results will be added once the first benchmark is complete.

| Model | Dataset | RUL MAE | Asymmetric score | False alarm rate |
|---|---|---|---|---|
| _coming soon_ | - | - | - | - |

---

## Roadmap

- [ ] Write dataset download and loader scripts
- [ ] Build cleaning, windowing and feature-extraction pipeline
- [ ] Implement leakage-safe train / test splitting
- [ ] Add baseline and anomaly-detection models
- [ ] Add ML and deep learning RUL models
- [ ] Add failure-aware metrics and the maintenance cost model
- [ ] Publish the first reproducible benchmark
- [ ] Add a simple dashboard for machine health scores
- [ ] Add more datasets and machine types

---

## Limitations

- Public datasets are mostly lab or test-rig data on engines and bearings, not full heavy-machinery fleets. Real machines have noisy sensors, mixed workloads and missing data.
- Results on one dataset may not carry over to another machine type.
- This project is for research and education. It is not a certified tool for safety-critical maintenance decisions.

---

## Contributing

Contributions are welcome, especially:

- New dataset loaders
- New models and baselines
- Better failure-aware metrics
- Bug reports and reproducibility fixes

Please open an issue to discuss before sending a large pull request.

---

## License

MIT License. See [LICENSE](LICENSE).

---

## Author

**Amol Tathe**
[LinkedIn](https://linkedin.com/in/amoltathesoftwareengineer) · [GitHub](https://github.com/amoltathesoftwareengineer)
