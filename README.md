# Renewable capacity expansion: a system dynamics model in Vensim

A small system dynamics model of wind and solar capacity growth, built in Vensim to learn the methodology, with a gas price shock scenario.

## What is in this repository
- `model/`: Vensim model files (v1 base model, v2 with a gas price shock).
- `results/`: exported time series for each run.
- `figures/`: model sketch and result graphs.
- `docs/model_note.pdf`: a two-page note on the model, the scenarios and what I learned.

## Model
Two stocks (installed capacity, capacity under construction), a three-year construction delay and a one-year perception delay. Two feedback loops: learning (reinforcing) and market value (balancing). All parameters are illustrative and not calibrated to data.

## Scenarios
| Run | Gas shock | Supply line weight |
| --- | --- | --- |
| Base | none | 0 (naive) |
| Gas shock, naive | 50 EUR/MWh, 2030, 3 years | 0 |
| Gas shock, aware | 50 EUR/MWh, 2030, 3 years | 1 |

Supply line weight 0 means investors ignore plants under construction. Weight 1 means they count them.

## Software
Vensim PLE, time step 0.019231 years (weekly), 2025 to 2060.
