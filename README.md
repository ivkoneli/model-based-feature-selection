# Izbor atributa na osnovu modela mašinskog učenja

Seminarski rad iz predmeta *Prikupljanje i predobrada strukturiranih podataka za mašinsko učenje*.

**Tema:** model-based feature selection, odnosno izbor atributa pomoću važnosti koju daje model stabla (Random Forest, gradient boosting).

## Istraživačko pitanje

> Koliko atributa može da se izostavi na osnovu važnosti iz modela stabla, a da rezultat na test skupu ostane sličan rezultatu modela sa svim atributima?

## Plan eksperimenta

1. **Baseline:** isti model obučen na svim atributima.
2. **Važnost atributa** se računa samo na skupu za obučavanje (ili na train delu svakog folda pri cross-validation):
   - MDI (impurity-based `feature_importances_`),
   - permutaciona važnost,
   - poređenje sa `SelectFromModel` (prag mean/median) i RFE/RFECV.
3. **Kriva broj atributa → test metrika:** zadržava se top-k atributa za više vrednosti k.
4. **Kriterijum za „sličan rezultat“:** najmanji k čiji je rezultat u okviru zadate tolerancije od baseline-a (npr. ≤1 p.p.), uz CV standardnu devijaciju.
5. **Dodatno:** stabilnost izbora između foldova, uticaj korelisanih atributa na MDI, ušteda u vremenu obučavanja i predikcije.

Sve faze predobrade koje uče iz podataka nalaze se u `Pipeline`-u, da ne bi bilo curenja informacija iz test skupa.

## Kandidati za skupove podataka

Brzi probe: Random Forest (200 stabala), split 75/25, top-k atributa po MDI važnosti sa train skupa.

| Skup | Izvor | n | atributa | Svi atributi | Top-k | Zašto ima smisla |
|---|---|---|---|---|---|---|
| **Human Activity Recognition (HAR)** | [UCI](https://archive.ics.uci.edu/dataset/240) / OpenML `har` | 10.299 | 561 | acc 0.978 | 20 → 0.972, 50 → 0.976 | realni senzorski podaci, mnogo korelisanih i redundantnih atributa |
| **Madelon** | [UCI](https://archive.ics.uci.edu/dataset/171) / OpenML `madelon` | 2.600 | 500 | AUC 0.806 | 20 → **0.957** | veštački skup: 20 relevantnih i 480 šumnih atributa, pa je poznato šta je „tačan“ izbor |
| **Superconductivity** | [UCI](https://archive.ics.uci.edu/dataset/464) / OpenML `superconduct` | 21.263 | 81 | R² 0.916 | 10 → 0.907, 20 → 0.913 | regresija, mnogo agregiranih i korelisanih atributa |
| Spambase | [UCI](https://archive.ics.uci.edu/dataset/94) / OpenML `spambase` | 4.601 | 57 | AUC 0.989 | 20 → 0.987 | mali skup, posluži za brzu proveru |
| SECOM | [UCI](https://archive.ics.uci.edu/dataset/179) | 1.567 | 590 | – | – | proizvodnja poluprovodnika, nedostajuće vrednosti, jaka neizbalansiranost; težak skup |

**Izabran skup: HAR.** Klasifikacija 6 aktivnosti (hodanje, penjanje i silaženje niz stepenice, sedenje, stajanje, ležanje) na osnovu 561 atributa iz signala akcelerometra i žiroskopa. Atributi su jako korelisani, pa se očekuje da je dovoljan mali deo njih. Koristi se zvanična podela po osobama (21 osoba u train, 9 u test skupu).

Etape eksperimenta su opisane u [`notebooks/feature_selection_har.ipynb`](notebooks/feature_selection_har.ipynb).

Podaci se skidaju direktno u notebooku sa UCI repozitorijuma i ne čuvaju se u repozitorijumu.

## Struktura

```
data/        # lokalni keš skupova (nije u repou)
notebooks/   # praktični deo (Jupyter)
docs/        # teorijski deo (izvorni format + PDF)
```

## Literatura (polazna)

- Breiman, L. (2001). *Random Forests*. Machine Learning 45.
- Guyon, I., Elisseeff, A. (2003). *An Introduction to Variable and Feature Selection*. JMLR 3.
- Strobl, C. et al. (2007). *Bias in random forest variable importance measures*. BMC Bioinformatics.
- Kursa, M., Rudnicki, W. (2010). *Feature Selection with the Boruta Package*. JSS 36.
- Cherepanova, V. et al. (2023). *A Performance-Driven Benchmark for Feature Selection in Tabular Deep Learning*. NeurIPS. [arXiv:2311.05877](https://arxiv.org/abs/2311.05877)
