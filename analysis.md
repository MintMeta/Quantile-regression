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

Žemiau pateiktame sklaidų diagramų paveiksle pastebimos kintamųjų X, Y ir Z nulinės reikšmės. Tikėtina, kad jos yra klaidingos ir darys didelę įtaką mūsų modeliui, todėl šių kintamųjų neįtrauksime į kvantilių regresijos modelį. Taip pat galima matyti, jog didėjant karatų svoriui, didėja ir kainos išsibarstymas. Tai rodo, jog pateiktose sklaidų diagramose vyrauja netiesiniai ryšiai.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/9a46c52a-6abc-4678-b25c-355e124a881d" />

Kategorinių kintamųjų reikšmių pasiskirstymas stebėtas stačiakampėmis diagramomis. Iš diagramų matome, jog duomenyse yra daug išskirčių, todėl nuspręsta taikyti kvantilių regresiją, kadangi ji nėra jautri išskirtims.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/921cb032-85f8-4062-880f-3eaa238380b6" />

## Prielaidų patikra ##

Atlikus pradinę analizę sekantis žingsnis ką privalome padaryti yra patikrinti prielaidas. Kvantilių regresinėje analizėje, skirtingai nei kito tipo analizėse, nereikia tikrinti normalumo ar heteroskedatiškumo, nes ji niekur nenaudoja prielaidų apie klaidų skirstinį ar jų vienodą sklaidą. Todėl net jei likučiai:

- nėra normalūs,
- jų sklaida kinta (heteroskedastiškumas),

kvantilių regresija vis tiek veikia. Tačiau reikia patikrinti ar kintamieji nėra tarpusavyje susiję. Multikolinearumui identifikuoti pasitelkta koreliacijų matrica, bei VIF reikšmėmis.

### Koreliacijos matrica ###

Iš koreliacijos matricos matom, jog smarkiausiai su priklausomu kintamuoju *price* koreliuoja *carat*, *X*, *Y* bei *Z* kintamieji, tačiau matmenų kovariantės taip pat turi smarkų ryšį tarpusavyje. Įtraukiant *X*, *Y* ir *Z* kintamuosius į modelį, galima susidurti su multikolinearumo problema. Tai yra dar viena priežastis, kodėl šio tipo duomenys nėra įtraukti konstruojant kvantilių regresijos modelį. Stipriausia koreliacija su atsako kintamuoju *price* matoma *carat* dalyje, kuri siekė net 0,92. Galima galvoti, jog šis kintamasis bus vienas reikšmingiausių faktorių nulemiančių kainas.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/2909c3b9-83ca-4046-b981-c43fe36ca261" />

### VIF reikšmės ###

Dar viena multikolinearumo patikra atlikta naudojant VIF reikšmes. VIF rodo, kiek daug padidėja sukurto modelio koeficiento standartinė paklaida dėl to, kad kintamieji tarpusavyje koreliuoja. Teorijoje yra sakoma, jog jeigu VIF > 4, tuomet ryšys tarp kintamųjų egzistuoja ir tikėtina, jog multikolinearumas gali sukelti problemų. Tokiu atveju derėtų spręsti šią problemą įtraukiant naujus arba panaikinant netinkamus kintamuosius.

Žemiau pateiktoje lentelėje nurodomi VIF reikšmių gauti rezultatai.

| Kintamieji  | VIF |
| ------------- | ------------- |
| carat   | 1,32  |
| depth  | 1,38  |
| table  | 1,79  |
| color  | 1,17  |
| cut  | 1,93  |
| clarity  | 1,30  |

Visos gautos VIF reikšmės yra < 4, todėl užfiksuota žema koreliacija. Kintamieji yra galimi naudoti modelyje.

## Kvantilių regresija ##
> [!NOTE]
> Duomenų rinkinys buvo padalintas į mokymo ir testavimo aibes santykiu 90/10, kur 90 % duomenų buvo naudojami modelio apmokymui, o likusi 10 % dalis - testavimui.
> Reikšmė τ (tau) yra kvantilio lygis, kuris yra tarp 0 ir 1. 

Analizės metu naudotos τ reikšmės nuo 0.1, 0.2,..., 0.9. Sukurtas tuščias vektorius, kuriame talpintos MSE metrikų reikšmės. Šio vektoriaus ilgis atitinka τ reikšmių kiekiui (kadangi į šį vektorių buvo įrašomi vidutinės kvadratinės paklaidos (MSE) rezultatai kiekvienam τ).

```
taus <- seq(0.1, 0.9, by = 0.1)
mse_values <- numeric(length(taus))
```
Toliau sukurtas ciklas, kuris eina per kiekvieną τ elementą. Taip pat ciklo metu yra sukuriami kvantilių regresijos modeliai, kurie yra apmokomi su mokymo aibės duomenimis. Po modelio sukūrimo buvo atliktas atsako reikšmės prognozavimas (su testavimo duomenimis) ir apskaičiuojama vidutinė kvadratinė paklaida, kurios reikšmė įrašyta į anksčiau sukurtą vektorių.

```
for (i in seq_along(taus)) {
  tau <- taus[i]
  model <- rq(price ~ carat + depth + table + color + cut + clarity, tau = tau, data = train_data)
  preds <- predict(model, newdata = test_data)
  mse_values[i] <- mean((test_data$price - preds)^2)
}
```

Matematinis atsako ir kovariančių užrašymas bei kvantilių regresijos modelio struktūra, kuri buvo sudaryta remiantis teorinio aprašymo formulėmis yra pateikiami žemiau esančiame paveiksle. Čia *y* yra atsakas, *x* - kovariančių ir intercepto (laisvojo nario) vektoriaus komponentės vektorius.
> [!NOTE]
> Interceptas užtikrina, kad net jei visi kiti kintamieji būtų lygūs nuliui, modelis vis tiek turi pastovią pradžios reikšmę ir pastovų poslinkį.

<img width="614" height="256" alt="Image" src="https://github.com/user-attachments/assets/fa3ddca5-8e2d-4c6d-b67b-31d10dfbc6f1" />


<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/4d4d10d2-ec5c-4d07-a1cb-4a1b31966c2b" />

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/ed41c1d0-03bc-4059-bac2-246cb347b591" />
