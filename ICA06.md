# ICA 06

**Gruppe 4: Crippling Depression**

Brage Sydskogen, Martin Hovet, Tønnes Røren, Marius Kaurin, Kent Daleng

Repo for eksperimenter: https://github.com/crippling-depression/is105-ica06

Det mest relevante av kode ligger i mappen `speech/`.

## Innhold
* [Eksperiment 1](#eksperiment-1)
* [Eksperiment 2](#eksperiment-2)
* [Eksperiment 3](#eksperiment-3)
* [Eskperiment 4](#eksperiment-4)


## Eksperiment 1

![Marius](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica06-marius.png)

Marius: Maske

---

![Tønnes](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica06-tonnes.png)

Tønnes: Maske

---

![Martin](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica06-martin.png)

Martin: Maske

---

![Kent](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica06-kent.png)

Kent: Maske

---

![Brage](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica06-brage.png)

Brage: Maske

---


Vi valgte å bruke ordet “maske” i vårt eksperiment med lydfrekvenser.
Vi ser at bildene er relativt like, til tross for forskjellige stemmer og dialekter.
Både “m” og “a” holder seg lengre ned på bildet, fordi “ma” lyden er dyp.
“s” har en høy frekvens og er dermed mørk i øvre del av bildet.
“k” gir et slags mellomrom i bildet, det er den litt avsluttende k-lyden.
Selv om at “e” avsluttes lavere igjen og de mørke punktene er relativt like på alle bildene, kan vi likevel se at Martin’s uttalte “e” ikke er like åpen; de høyere frekvensene ikke er like fremtredende og blir nesten en “ø”-lyd.

Noen av forskjellene vi ser er at Tønnes sin er en del mørkere, det kommer av at han sa “maske” litt høyere enn både Marius og Martin. Martin har en del bakgrunnsstøy som vi ser, men påvirker ikke bildet mer enn at det ser litt rotete ut.

Med tanke på en teoretisk språkgjenkjennelsesalgoritme mener vi at en sammensatt analyse av transienter og formanter vil kunne legge et grunnlag for programmatisk gjenkjennelse av ord. Idéen er altså at man gjenkjenner fonemer og at sammensetningen av disse blir hele ord. Ved å oppdage pauser vil algoritmen også kunne avgrense sekvenser av fonemer til ord.

Alt dette kan gjøres ved hjelp av maskinlæring, man kan for eksempel trene en maskinlæringsalgoritme til å gjenkjenne spesifikke ord fra et større datasett med innspillinger av forskjellige folks uttalelse av ordene.

## Eksperiment 2

* [Tekst til tale](https://darn.site/text)

`espeakbox` er satt opp på Marius sin server, og blir spurt når en forespørsel
sendes til Kent's server. På denne måten blir Marius' server aldri direkte vist for klienten.

1. Klient POSTer "hallo" til server
2. Server sender forespørsel til "skytjeneste"
3. "Skytjeneste" sender tilbake lydfil
4. Server sender lydfil til Klient

## Eksperiment 3

* [Tale til tekst](https://darn.site/recognize)

Ved bruk av [Google Speech](https://cloud.google.com/speech/docs/reference/libraries) kan vi sende lydfiler til deres talegjenkjennelsestjeneste, og få tilbake en string som Google mener mest sannsynlig representerer det som ble sagt.

## Eksperiment 4

* [Tale til tekst til tale](https://darn.site/nextlevel)

En sammensetning av eksperiment 3 og 2.
1. En lydfil med tale blir sendt til Google som returnerer en tekst
2. Teksen blir så sendt til espeakbox hos Marius som returnerer en lydfil
