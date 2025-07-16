# Kvantilių regresinė analizė

## Įvadas ##

**Tyrimo tikslas**: Nustatyti, kaip skirtingi deimantų požymiai veikia deimantų kainą įvairiuose pasiskirstymo kvantiliuose, taip įvertinant skirtumus tarp pigesnių ir brangesnių deimantų.

Tyrimo uždaviniai:
1. Atlikti pradinė duomenų analizę ir pateikti aprašomąją statistiką.
2. Atrinkus tinkamus regresorius, sudaryti kvantilių regresijos modelį bei išnagrinėti požymius, įtakojančius deimantų kainų augimą.
3. Įvertinti sukonstruoto modelio tikslumą.
4. Pateikti rezultatus ir galutines išvadas.

## Pradinė duomenų analizė ##

Analizės metu naudotas R paketo, *ggplot2* bibliotekos duomenų rinkinys *diamonds*. Tikrintos praleistos N\A ir nulinės reikšmės bei stebėtas priklausomo kintamojo *price* pasiskirstymas. Remiantis žemiau pateiktu grafiku pastebimas asimetriškas kainos pasiskirstymas. Galima teigti, jog didžiąją dalį duomenų rinkinio sudaro pigesni deimantai. Duomenys nėra normaliai pasiskirstę, tačiau kvantilių regresinė analizė yra atspari nenormalumui, todėl duomenys puikiai tinka tolimesniam tyrimui.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/cedb97e9-f8b2-481a-9768-2651511bcd41" />




