# Concrete compressive strength: Abrams' law vs machine learning

How much of concrete strength can a simple physical law explain, and what does machine learning add on top of it?

The notebook tests Abrams' law and the slow strength gain of slag and fly ash on the UCI concrete dataset. It then compares a 4-coefficient physics model with linear regression and random forest, validated on **mixes the model has never seen**.

## Data

UCI *Concrete Compressive Strength* dataset: 1,030 tests, 8 inputs (kg/m³ of cement, blast-furnace slag, fly ash, water, superplasticizer, coarse and fine aggregate, plus age in days), and compressive strength in MPa as the output.

> I-C. Yeh, "Modeling of strength of high-performance concrete using artificial neural networks", *Cement and Concrete Research* 28(12) (1998) 1797–1808.

The data are not stored in this repository. The notebook downloads a CSV copy on first run and checks it against the UCI original (row count and first row).

## Physics used

- **Hydration.** Cement reacts with water to form C-S-H gel. Water that does not react leaves pores.
- **Abrams' law.** $f_c = A / B^{\,w/c}$: strength falls exponentially with the water/cement ratio.
- **Age.** Strength grows roughly linearly in ln(age).
- **Pozzolanic reaction.** Slag and fly ash react slowly with the Ca(OH)₂ released by cement, so they add strength later rather than early.

## Results

| Finding | Result |
|---|---|
| Data audit | 25 exact duplicates removed; the 1,005 remaining tests come from only 427 distinct mixes |
| Abrams' law at 28 days | R² 0.59 with water/**binder** vs 0.42 with water/cement, so slag and fly ash act as binder |
| Early strength, mixes with ≥30% slag/fly ash | 56% of 28-day strength at 7 days, vs 65% without (p < 0.001) |
| Later strength gain, 28 → 90 days | +30% with ≥30% slag/fly ash, vs +19% without (p < 0.001) |

Model accuracy, validated on unseen mixes (grouped 5-fold CV):

| Model | R² | MAE (MPa) |
|---|---|---|
| Linear regression, 8 raw inputs | 0.58 | 8.3 |
| Physics model, 4 coefficients | 0.76 | 5.7 |
| Random forest, raw inputs | 0.85 | 4.5 |
| Random forest + physics features | 0.87 | 4.0 |

- A random (ungrouped) split puts the raw random forest at R² 0.90, which overstates how well it predicts new mixes.
- The forest relies mainly on age and water/binder, the same two variables the physics points to.

## How to run

```
pip install -r requirements.txt
```

Open `concrete_strength_abrams.ipynb` and run all cells. Figures are saved to `figures/`.

## Limitations

- The data contain no cement chemistry, curing temperature or aggregate type.
- The tests come from many labs, so some scatter is between labs rather than physics.
