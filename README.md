# Renewable capacity expansion: a system dynamics model in Vensim

A small system dynamics model of wind and solar capacity growth, built in Vensim to learn the methodology, with a gas price shock scenario. Parameters are illustrative and not calibrated to data.

## What is in this repository

| File | Content |
| --- | --- |
| `renewable-capacity-gas-shock-vensim.mdl` | The Vensim model, including the gas price shock settings |
| `output-installed capacity.txt` | Exported time series of installed capacity for each run |
| `README.md` | This file: model description, all variables, constants and equations, scenarios and results |

## The model in short

Investors compare what a new wind or solar plant earns with what it costs, and start projects when the ratio (profitability) is above 1.

- **Stocks:** installed renewable capacity (GW) and capacity under construction (GW).
- **Flows:** investment starts, completions (after the construction time) and retirements (after the plant lifetime).
- **Delays:** a three-year construction delay and a one-year perception delay on market value.
- **R1, learning (reinforcing):** more installed capacity lowers costs, which raises profitability and drives more investment.
- **B1, market value (balancing):** more installed capacity raises the renewable share, which lowers the market value of renewable electricity, so profitability and investment fall.
- **B2, retirement (balancing):** more installed capacity means more plants retire.

Market value is the average price per MWh that wind and solar receive. In the model, gas plants set the electricity price, and the market value falls as the renewable share rises.

## Model settings

Vensim PLE. Time from 2025 to 2060 (years). Time step 0.019231 years (weekly), Euler integration.

## Input data at a glance

All values are illustrative and not calibrated to data. The full list with meanings is under Constants below.

| Group | Input | Value | Unit |
| --- | --- | --- | --- |
| Power system | Initial installed capacity | 50 | GW |
| | Electricity demand | 500 (fixed) | TWh/yr |
| | Capacity factor | 0.30 | dimensionless |
| | Plant lifetime | 25 | years |
| | Construction time | 3 | years |
| Costs | Initial LCOE | 55 | EUR/MWh |
| | Learning rate | 0.15 (per doubling) | dimensionless |
| Fuel and carbon | Base gas fuel cost | 60 | EUR/MWh |
| | Emission factor | 0.37 | t CO2/MWh |
| | Base CO2 price | 80 | EUR/t |
| | CO2 step height / time | 0 / 2030 (not used) | EUR/t / year |
| Market | Cannibalisation factor | 0.8 | dimensionless |
| Investor behaviour | Investment response | 1.0 | 1/yr |
| | Max growth rate | 0.25 | 1/yr |
| | Perception time | 1 | years |
| Simulation | Time span | 2025 to 2060 | years |
| | Time step | 0.019231 (weekly) | years |

Only the gas shock settings (height, start, duration) and the supply line weight change between the runs. See Scenarios below.

## Stocks

| Stock | Unit | Initial value | Equation |
| --- | --- | --- | --- |
| Installed renewable capacity, K | GW | Initial installed capacity (50) | dK/dt = Completions − Retirements |
| Capacity under construction, U | GW | 12.5 × Construction time (37.5) | dU/dt = Investment starts − Completions |

The initial value of U is set so that the pipeline starts in balance with an investment rate of 12.5 GW per year.

## Flows

All flows are in GW per year.

| Flow | Equation |
| --- | --- |
| Investment starts | `K * MAX(0, MIN(Max growth rate, Investment response * (Profitability - 1)))` |
| Completions | `U / Construction time` |
| Retirements | `K / Plant lifetime` |

Investment starts is proportional to installed capacity K (growth is relative to what exists), is zero when profitability is below 1, and cannot exceed the maximum growth rate.

## Auxiliary variables

| Variable | Unit | Equation |
| --- | --- | --- |
| Generation per GW | TWh/yr per GW | `8.76 * Capacity factor` |
| Renewable generation | TWh/yr | `K * Generation per GW` |
| Actual renewable share | dimensionless | `MIN(1, Renewable generation / Electricity demand)` |
| Share seen by investors | dimensionless | `MIN(1, (K + Supply line weight * U) * Generation per GW / Electricity demand)` |
| CO2 price | EUR/t | `Base CO2 price + STEP(CO2 step height, CO2 step time)` |
| Gas fuel cost | EUR/MWh | `Base gas fuel cost + Gas price trend * (Time - 2025) + Gas shock height * PULSE(Gas shock start, Gas shock duration)` |
| Gas set price | EUR/MWh | `Gas fuel cost + CO2 price * Emission factor` |
| Expected market value | EUR/MWh | `Gas set price * MAX(0.1, 1 - Cannibalisation factor * Share seen by investors)` |
| Perceived market value | EUR/MWh | `SMOOTH(Expected market value, Perception time)` |
| Learning exponent | dimensionless | `-LN(1 - Learning rate) / LN(2)` |
| Renewable LCOE | EUR/MWh | `Initial LCOE * (K / Initial installed capacity) ^ (-Learning exponent)` |
| Profitability | dimensionless | `Perceived market value / Renewable LCOE` |

Notes on the equations:

- **8.76** converts GW at a given capacity factor into TWh per year (8760 hours divided by 1000).
- **Share seen by investors** uses the supply line weight. With weight 0 investors count only installed plants (naive). With weight 1 they also count plants under construction (aware).
- **Cannibalisation:** the market value is the gas set price reduced by a share of the renewable share, with a floor of 10% of the gas set price.
- **Learning curve:** each doubling of installed capacity cuts LCOE by the learning rate. With a learning rate of 0.15 the exponent is 0.234.
- **Gas shock:** PULSE adds the shock height to the gas fuel cost from the start year for the given duration, then returns it to zero.

## Constants

| Constant | Value | Unit | Meaning |
| --- | --- | --- | --- |
| Initial installed capacity | 50 | GW | Starting stock K, and the reference capacity for learning |
| Electricity demand | 500 | TWh/yr | Fixed demand |
| Capacity factor | 0.30 | dimensionless | Average output of wind and solar relative to rated capacity |
| Base gas fuel cost | 60 | EUR/MWh | Gas fuel cost before any trend or shock |
| Emission factor | 0.37 | t CO2/MWh | CO2 emitted per MWh of gas generation |
| Base CO2 price | 80 | EUR/t | CO2 price before any step |
| CO2 step height | 0 | EUR/t | Not used in the three runs |
| CO2 step time | 2030 | year | Not used in the three runs |
| Initial LCOE | 55 | EUR/MWh | Cost of renewable electricity at the initial capacity |
| Learning rate | 0.15 | dimensionless | Cost reduction per doubling of capacity |
| Cannibalisation factor | 0.8 | dimensionless | How strongly the renewable share lowers market value |
| Construction time | 3 | years | Delay from investment start to completion |
| Plant lifetime | 25 | years | Time until a plant retires |
| Max growth rate | 0.25 | 1/yr | Upper limit on investment starts relative to K |
| Investment response | 1.0 | 1/yr | How strongly investors react to profitability above 1 |
| Perception time | 1 | years | Delay in how fast investors update the market value they see |
| Supply line weight | 0 or 1 | dimensionless | 0 = investors ignore plants under construction, 1 = they count them |
| Gas shock height | 0 or 50 | EUR/MWh | Size of the temporary gas fuel cost increase |
| Gas shock start | 2030 | year | When the shock begins |
| Gas shock duration | 3 | years | How long the shock lasts |
| Gas price trend | 0 | EUR/MWh per year | Steady change in gas fuel cost |

## Feedback loops

| Loop | Type | Path |
| --- | --- | --- |
| R1, learning | Reinforcing | K → Renewable LCOE → Profitability → Investment starts → U → Completions → K |
| B1, market value | Balancing | K → Share seen by investors → Expected market value → Perceived market value → Profitability → Investment starts → U → Completions → K |
| B2, retirement | Balancing | K → Retirements → K |
| R2, base growth | Reinforcing | K → Investment starts → U → Completions → K (investment is proportional to K) |

## Scenarios

| Run | Gas shock height | Shock start and duration | Gas price trend | Supply line weight |
| --- | --- | --- | --- | --- |
| Base | 0 | not used | 0 | 0 (naive) |
| Gas shock, naive | 50 EUR/MWh | 2030, 3 years | 0 | 0 |
| Gas shock, aware | 50 EUR/MWh | 2030, 3 years | 0 | 1 |

Everything else is identical across the runs.

## Main result

A temporary gas price shock raised peak installed capacity from 142.4 GW to 204.6 GW and was followed by a long period with almost no new investment. Investors who counted plants under construction built much less (peak 127.5 GW). The overbuild comes from the construction delay combined with investors who cannot see the pipeline, not from the shock alone.

| Run | Peak capacity (GW) | Year of peak | Lowest capacity after peak (GW) | Capacity in 2060 (GW) |
| --- | --- | --- | --- | --- |
| Base (naive) | 142.4 | 2034.6 | 103.4 | 109.6 |
| Gas shock (naive) | 204.6 | 2037.2 | 102.7 | 103.3 |
| Gas shock (aware) | 127.5 | 2037.0 | 96.2 | 96.2 |

In the naive shock run the renewable share reaches the model's 100% cap, so that peak is an upper bound that shows the direction of the effect, not a forecast.

## Limits

The model has fixed electricity demand, one aggregated wind and solar technology, gas as the only price setter, and no curtailment, storage or grid limits. Investors treat a temporary shock like lasting news. There is no real data.

## Next steps

- Add demand growth from electrification.
- Split wind and solar, with their own costs and learning.
- Add curtailment and storage.
- Calibrate to historical capacity growth and prices, then compare scenarios with published outlooks.
