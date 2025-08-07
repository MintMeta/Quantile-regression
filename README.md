# LIETUVIŠKA VERSIJA - Kvantilių regresija

## Kas tai? ##
Kvantilių regresija yra statistinis metodas, kuris skirtingai nei kiti, leidžia stebėti kaip nepriklausomi kintamieji veikia atsako kvantilius (o ne tik vidurkį). T.y leidžia stebėti kraštininių reikšmių tendenciją.
Šios regresijos atveju nėra prielaidos apie atsako skirstinį ir dispersiją.

- 0.25 kvantilis (25-asis): žemiausi 25 % reikšmių
- 0.50 kvantilis: mediana (vidutinė reikšmė)
- 0.75 kvantilis: reikšmė, virš kurios yra 25 % didžiausių reikšmių

## Kada naudoti? ##

- Kai turime didelį duomenų kiekį.
- Yra daug išskirčių.
- Priklausomas kintamasis nėra normaliai pasiskirstęs.
- Domina ne tik priklausomo kintamojo vidurkis, bet ir apatinis ir viršutinis rėžiai.
- Užfiksuotas heteroskedatiškumas.

# ENGLISH VERSION - Quantile regression

## What is it? ##
Quantile regression is a statistical method which, unlike others, allows us to observe how independent variables affect response quantiles (rather than just the mean). In other words, it allows us to observe the trend of extreme values.
In the case of this regression, there are no assumptions about the distribution and dispersion of the response.

- 0.25 quantile (25th): lowest 25% of values
- 0.50 quantile: median (average value)
- 0.75 quantile: value above which 25% of the highest values lie

## When to use? ##

- When we have a large amount of data.
- There are many outliers.
- The dependent variable is not normally distributed.
- We are interested not only in the mean of the dependent variable, but also in the lower and upper bounds.
- Heteroscedasticity has been detected.
