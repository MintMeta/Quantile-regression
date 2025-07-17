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

Iš koreliacijos matricos matom, jog smarkiausiai su priklausomu kintamuoju *price* koreliuoja *carat*, *X*, *Y* bei *Z* kintamieji, tačiau matmenų kovariantės taip pat turi smarkų ryšį tarpusavyje. Įtraukiant *X*, *Y* ir *Z* kintamuosius į modelį, galima susidurti su multikolinearumo problema. Tai yra dar viena priežastis, kodėl šio tipo duomenys nėra įtraukti konstruojant kvantilių regresijos modelį.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/b4c8901b-f606-4296-955f-8b8e805b5162" />

| Kintamieji  | VIF |
| ------------- | ------------- |
| carat   | 1,32  |
| depth  | 1,38  |
| table  | 1,79  |
| color  | 1,17  |
| cut  | 1,93  |
| clarity  | 1,30  |

Visos gautos VIF reikšmės yra < 4, todėl užfiksuota žema koreliacija.



