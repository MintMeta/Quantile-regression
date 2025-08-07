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

Iš koreliacijos matricos matom, jog smarkiausiai tarpusavyje koreliuoja *carat*, *X*, *Y* bei *Z* kintamieji. Įtraukiant *X*, *Y* ir *Z* kintamuosius į modelį, galima susidurti su multikolinearumo problema. Tai yra dar viena priežastis, kodėl šio tipo duomenys nėra įtraukti konstruojant kvantilių regresijos modelį.

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

Toliau stebėta kaip modelis tiksliai atlieka prognozes. Modeliui vertinti pasitelkta MSE (angl. mean squared errror) metrika, kuri rodo vidutinį skirtumą tarp tikrųjų ir modelio prognozuotų reikšmių. Remiantis grafiko rezultatais matoma, jog mažiausia paklaida gauta kai τ = 0.6 ir tai rodo, jog prognozuojamas 60‑asis procentilis kvantilis suteikė mažiausią prognozės dispersiją lyginant su faktinėmis kainomis testavimo duomenyse. Šiame kvantilyje prognozės geriausiai minimizuoja kvadratinę paklaidą.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/4d4d10d2-ec5c-4d07-a1cb-4a1b31966c2b" />

Taip pat svarbu pasižiūrėti ar modelio prognozuoti kvantiliai atitinka tikrąjį duomenų pasiskirstymą. 
- Juoda linija vaizduoja empirinę tikimybę, kuri yra apskaičiuota naudojantis testiniais duomenimis.
- Raudona linija rodo tobulą modelio prognozuojamų kvantilių atitikimą teoriniams kvantiliams. T. y. empirinė tikimybė visiškai sutampa su teorine (kiek teoriškai turėtų būti, tiek ir yra praktikoje).

Pirmame grafike galima matyti, jog abi linijos yra labai arti viena kitos. Tai geras signalas, rodantis, jog empirinės tikimybės beveik sutampa su teorinėmis ir pasirinkta kvantilių regresija stabiliai atlieka prognozes.

Nagrinėjant empirinių ir teorinių tikimybių skirtumų grafiką taip pat akivaizdu, kad skirtumas tarp tikimybių yra itin mažas, o ties τ = 0.8 reikšme empirinė tikimybė atitinka teorinę.
<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/ed41c1d0-03bc-4059-bac2-246cb347b591" />

Remiantis teorinėmis matematinėmis formulių išraiškomis buvo apskaičiuota vidutinė kvantilių nuostolių funkcija, kuri iš esmės parodo kiek vidutiniškai neteisingos yra kvantilio 𝜏 prognozės.

- Kuo mažesnė reikšmė – tuo modelis geriau atspindi pasirinktą kvantilį.

<img width="612" height="667" alt="Image" src="https://github.com/user-attachments/assets/684641ef-32f5-45f0-a536-bd6c345af320" />

Kvantilių nuostolių funkcijos grafike galima matyti, jog mažiausias nuostolis pastebimas kai τ = 0.1, τ = 0.2 ir τ = 0.9. Todėl galima teigti, jog modelis geriausiai geba prognozuoti pigesnius ir brangesnius deimantus, o dažniausiai klysta prognozuodamas vidutinės kainos deimantus ties τ = 0.6 kvantiliu.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/04f90c1c-405d-48f7-9aa1-855818ae0107" />

Sekančiame grafike nurodomas kvantilių regresijos palyginimas su OLS (tiesine regresija angl. *Ordinary Least Squares*)
Taškai rodo faktinių duomenų pasiskirstymą, kai *x* ašyje - deimantų karatai, o *y* ašyje - kaina. 

- $${\color{orange}OLS \space \color{orange}regresija}$$ - prognozuoja vidutinę kainą pagal *carat*.
- $${\color{red}0.1 \space \color{red}kvantilis}$$ - rodo kainos elgesį pigesnių deimantų grupėje.
- $${\color{limegreen}0.6 \space \color{limegreen}kvantilis}$$ - rodo kainos elgesį šiek tiek brangesnių nei vidutinių deimantų grupėje.
- $${\color{darkorchid}0.9 \space \color{darkorchid}kvantilis}$$ - rodo brangiausių deimantų kainą pagal *carat*.

Kadangi kvantiliai modeliuoja skirtingus kainos pasiskirstymo lygmenius, tai galima teigti, kad $${\color{red}0.1 \space \color{red}kvantilis}$$ atskiria deimantus taip, kad apytiksliai 10 % duomenų taškų (deimantų) yra žemiau šios linijos, o apie 90% – virš jos. Žemiau linijos esantys deimantai, kurių kaina yra tarp 10 % pigiausių (palyginus su kitais turinčiais panašius karatus). Virš linijos yra deimantai, kurių kaina yra didesnė nei ta 10 % pigiausių – t. y. jie patenka į brangesnių 90 % grupę.

$${\color{limegreen}0.6 \space \color{limegreen}kvantilis}$$ atskiria duomenis santykiu 60:40. Po šia linija tikėtina, kad yra 60 % deimantų, kurie yra pigesni ir turintys panašias *carat* savybes. Analizuojant grafiką galima matyti, jog ties 1 karatu, modelis tikisi, kad 60 % deimantų kainavo ne daugiau nei $5000, o 60 % 2-jų karatų deimantų kaina buvo ne didesnė nei $125,000.

> [!IMPORTANT]
> Grafike matomas taškų tankis ***NĖRA*** tiesiogiai proporcingas kvantiliui. Tai yra, nors modelis matematiškai pasako, kad: „60 % stebėjimų tikėtina bus žemiau linijos“, tai dar ne reiškia, kad vizualiai matysime 60 % taškų apačioje.

Akivaizdu, jog pigių, vidutinių ir brangių deimantų kainos priklauso nuo jų svorių, kadangi kvantilių linijos palaipsniui kyla. Galima teigti, jog didėjant deimantų svoriui, didėja ir jų kaina.

Deimantai, kurių svoris atitinka 1 ir daugiau karatų, pasižymi platesne kainų įvairove. Pastebima, jog duomenys yra labiau išsibarstę ir tai reiškia, kad svoris nebevienareikšmiškai nulemia kainą. Galbūt atsiranda didesnė įtaka kitų kokybės savybių kaip spalva, skaidrumas ar pjūvio kokybė. Taip pat tai gali lemti ir kiti iššoriniai veiksniai (galbūt rinka pasiūlo daug įvairių variantų šiame svorio intervale).


<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/84bc984b-09ba-41ec-8905-9c289f25e20d" />

Sekantis grafikas vaizduoja deimanto procentinio gylio ir kainos priklausomybę. Pastebimas neigiamas gylio poveikis žemiausiame ($${\color{red}0.1}$$) ir aukščiausiame ($${\color{darkorchid}0.9}$$) kvantiliuose, kas leidžia daryti prielaidą, jog brangiausių ir pigiausių deimantų atveju didesnis gylis gali būti laikomas trūkumu, kuris neigiamai paveikia kainą. Kadangi $${\color{darkorchid}0.9 \space \color{darkorchid}kvantilis}$$ grafike patiria smarkesnį nuolydį, todėl brangiausi deimantai yra labiausiai paveikti kainų pokyčių. Per didelis gylis laikomas neidealiu proporcijų atžvilgiu, pažeidžiama estetika, kas gali būti viena iš priežasčių, kodėl produktai įvertinami pigiau nei įprastai.

Taip pat galima matyti didelį vertikalų duomenų išsibarstymą, kuris yra susitelkęs tarp apytiksliai 58-65 procentinio gylio. Remiantis šia informacija daroma prielaida, jog to pačio gylio deimantų kaina gali būti labai įvairi ir svyruoti nuo kelių šimtų iki keliolikos tūkstančių dolerių. Taigi, procentinis deimanto gylis nėra lemiamas ir vienas pagrindinių veiksnių, kuris smarkiai įtakotų deimantų kainą. 

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/aa4438c6-0bb4-413e-86e1-11f686b921f3" />

Dar vienas grafikas, kuriame vaizduojama kintamojo *table* (deimanto viršaus plotis plačiausio taško atžvilgiu) ir kainos analizė. Kaip ir ankstesniame grafike matomas platus vertikalus išsibarstymas, kas leidžia manyti, jog panašaus pločio deimantų kainos gali ženkliai skirtis. Deimantai, kurių pločio vertės buvo žemesnė nei 50, yra ganėtinai reti atvejai ir gali būti laikomi išskirtimis. Kintamasis *table* nėra stipriai susijęs su deimanto kaina ir vienas pats nėra pakankamas požymis kainai paaiškinti. Brangesniems deimantams ($${\color{darkorchid}0.9 \space \color{darkorchid}kvantilio}$$ ir $${\color{limegreen}0.6 \space \color{limegreen}kvantilio}$$ linijos) didesnis deimanto viršaus plotis gali būti susijęs su didesne kaina, bet poveikis vis tiek nėra stiprus. 

Klasikinės tiesinės regresijos atveju $${\color{orange}OLS \space \color{orange}linija}$$ kyla aukštyn ir tai rodo, jog *table* pokyčiai turi labai silpną teigiamą įtaką vidutinei deimantų kainai.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/d937298e-fcce-4bcc-9d7f-efd011c78eb0" />

## Išvados ##

- Deimantai, sveriantys 1 ir daugiau karatų, pasižymi gerokai įvairesnėmis kainomis
- Pastebimas didelis kainų išsibarstymas rodo, kad pirkėjai vertina ne tik masę, bet ir daugybę kitų savybių, ir tai lemia didelį kainų spektrą.
-
-
