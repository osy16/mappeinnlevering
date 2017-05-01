# ICA 02

**Gruppe 4: Crippling Depression**

Brage Sydskogen, Martin Hovet, Tønnes Røren, Marius Kaurin, Kent Daleng, Stian Simonsen

Repository: https://github.com/chinatsu/is105-ica02

## Innhold

* [Oppgave 1](#oppgave-1)
* [Oppgave 2](#oppgave-2)
* [Oppgave 3](#oppgave-3)
* [Oppgave 4](#oppgave-4)
* [Oppgave 5](#oppgave-5)

---

## Oppgave 1:

Til en gitt tid kan man gi et nøyaktig antall prosesser som kjører på en maskin, i alle fall i userspace, da man ikke har spesiell innsikt i hvilke prosesser som kjører i kjernen til operativsystemet når systemkall blir utført.

På Linux og Mac (og kanskje Windows også?), har prosesser andre tilstander enn å “kjøre”. Prosessene kan være i “uninterruptible sleep”, som oftest er dette mens I/O-kall blir utført (f.eks. ved kopiering av en fil, vil prosessen utføre read og write på harddisk); “interruptible sleep”, der prosessen venter på en “event” som skal fullføres; “zombie”, som betyr at forelderprosessen har blitt drept, men barna er fremdeles “i live”, de har ingen funksjon etter det.

|                              | Brage | Martin | Tønnes | Stian | Marius | Kent |
|-----------------------------:|-------|--------|--------|-------|--------|------|
| Prosesser (lokal maskin)     | 234 (2 running) | 75 | 80 | 79(?) | 153 | 88 (1 running) |
| Prosesser (server)           | 116 (1 running) | 118 (1 running) | 119 (1 running) | 118(?) |122 (1 running) | 127 (1 running) |
| Prosessortype                | Intel Core i5-2415M | Intel Core i7-6600U |Intel Core i3-4010U | Intel Core i5-3230M | AMD A8-7410 APU | Intel Celeron 2955U |
| Arkitektur                   | x86_64 | x86_64 | x86_64 | x84_64 | x86_64 | x86_64 |
| Klokkefrekvens               | 2300Mhz | 2800MHz | 1700MHz | 2600MHz | 2200Mhz | 1400MHz |
| Primært minne                | 4096Mb | 8060Mb | 4096Mb | 4096Mb | 8192Mb | 4096Mb |
|Cache                         | L1 Data: 2x32b, L1 Instructions: 2x32b, L2: 2x256Kb, L3: 3072Kb | L1 Data: 2x32, L1 Instructions: 2x32 L2: 2x256Kb, L3: 4096Kb | L1 Data: 2x32b, L1 Instructions: 2x32b, L2: 2x512Kb, L3: 3072Kb | L1 Data: 2x32kb, L1 Instructions: 2x32kb, L2: 2x256kb L3: 3072Kb | L1 Data: 4x32b, L1 Instructions: 4x32b, L2: 2048Kb | L1 Data: 2x32b, L1 Instructions: 2x32b, L2: 2x256Kb, L3: 2048Mb |
| CPU-kjerner (lokalmaskin)    | 2 | 2 | 2 | 2 | 4 | 2 |
| CPU-kjerner (server)         | 1 | 1 | 1 | 1 | 1 | 1 |
| Mest ressurskrevende prosess | Top (task manager) | Taskmgr.exe (oppgavebehandling) | Synaptics Touchpad 64-bit enhancements (touchpad-driver) | Football manager 2017 (steam spill) | WMI Provider Host | Firefox (nettleser) |

Vi kan si at alle systemene er like, da de bruker samme type prosessorarkitektur, og har ganske like cache-størrelser. Alle prosessortypene er altså ulike, men maskinkoden som kjører på hver av de er like, og vil oppføre seg likt med varierende nivåer av effektivitet (mtp. klokkefrekvens). Marius er den eneste av oss som har AMD-prosessor, en APU som kombinerer grafikkprosessor og prosessor på samme chip. Med tanke på den mest ressurskrevende prosessen, er det i alle tilfeller nettlesere som vinner minnebruk, mens oppgavebehandlinger popper, ikke overraskende, opp ganske høyt på listene i prosessorbruk -- mest fordi det brukes aktivt når man sjekker prosessene.

Vi tar utgangspunkt i oppstart av Linux-baserte distribusjoner, da oppstartsprosessen er forholdsvis kjent, og enkel å forstå. Etter en power-on self test, vil en moderne datamaskin mest sannsynlig ha EFI, og dermed forvente en FAT32-formatert partisjon på den førsteprioriterte harddisken, spesifikt for å kjøre en EFI bootloader fra. Selv om denne partisjonen er uavhengig av operativsystemet, blir den ofte brukt til å lagre Linux-kjernen på (som “mount”-punktet /root), samt en “initial RAM file system” (også kalt initramfs) tilhørende kjernen.

Linux-kjernen får stafettpinnen etter EFI bootloader, og pakker ut det som finnes i initramfs til et midlertidig “root”-filsystem i primært minne. Deretter vil `/init` bli kjørt, som fullfører oppstartsprosessen før det virkelige filsystemet blir “mounted” over det midlertidige. `init` har tradisjonelt vært mange små shell-scripts som fungerer i synergi for å starte opp ting som nettverk og filsystem, samtidig er `init` “PID 1”, som kjører fra oppstart til power off.
I senere tid har flere alternativer til den tradisjonelle `init` fått oppmerksomhet, og systemd er mest kjent, både på godt og vondt. Systemd er kontroversiell på grunn av at den tar over mange forskjellige ting, og ikke følger den tradisjonelle “UNIX-filosofien”; ett program skal gjøre én ting. Det er et mer altomfattende rammeverk, som tar hånd om ikke bare oppstart, men også service-administrasjon, (timer) en erstatning for `cron`, (journald) logging, (logind) brukerinnlogging, og så videre.
Et eksempel på prosessen videre kan være at brukeren logger inn via konsoll. Når dette har skjedd, vil et script kalt `.bashrc` bli lastet av bash (et shell), som blir startet automatisk når brukeren logger inn. Der kan flere konfigurasjoner være definerte, som for eksempel, dersom .bashrc blir lastet, og variabelen $DISPLAY ikke har blitt satt, vil `startx` bli kjørt. Xorg er hva som blir kalt en “display server”, som brukes som et rammeverk for å vise et grafisk miljø. Når `startx` starter, vil et shell script kalt .xinitrc (eller .xprofile) bli startet. I den filen, kan en window manager bli startet (som i tur har sine egne oppstartsscripts), samt andre prosesser som brukeren vil skal starte når Xorg starter. Hele dette resulterer i et hierarki av prosesser, det init “eier” alt, gjennom Xorg, en window manager, og helt ut til et vindu som for eksempel en nettleser.

For å observere kjørende prosesser i systemet, kan brukeren kalle et program som `top`, `ps`, eller `htop`i terminalen. Dersom et program har fryst og ikke mottar instruksjoner om å lukkes, kan brukeren bruke kommandoer som `kill` med en prosessid, eller `pkill` med et prosessnavn.

## Oppgave 2
Her er en YouTube-video som dokumenterer grunnleggende steg ved å kross-kompilere og overføre filer fra maskin til maskin. https://youtu.be/QjzKUhw7DEY


## Oppgave 3
GitHub repository: https://github.com/chinatsu/is105-ica02

Her er følgende output for test i pakken `sum`. TestSumUint32 feiler på grunn av at expected er med vilje gjort som feil verdi, TestSumFloat64 passerer testen, selv om inputverdien skal gi feil svar, dette er på grunn av at presisjonen ikke er høy nok for så mange desimalposisjoner. TestSumInt64 feiler på grunn av to ting; verdiene summert fører til overflow, samt er expected verdi ikke riktig.

```
=== RUN   TestSumInt8
--- PASS: TestSumInt8 (0.00s)
=== RUN   TestSumInt32
--- PASS: TestSumInt32 (0.00s)
=== RUN   TestSumUint32
--- FAIL: TestSumUint32 (0.00s)
        sum_test.go:94: Sum(4294967293, 1) returned 4294967294, expected 4294967291
=== RUN   TestSumFloat64
--- PASS: TestSumFloat64 (0.00s)
=== RUN   TestSumInt64
--- FAIL: TestSumInt64 (0.00s)
        sum_test.go:115: Sum(9223372036854775807, 1) returned an error: 9223372036854775807 + 1 results in an overflow
```

I samme repository finnes main_sum.go, som er skrevet noenlunde etter spesifikasjoner. Vi konkluderte med at ved to argumenter, kan det ikke gå an å slå fast hvilken type brukeren vil regne ut. Koden akkurat nå vil anta at argumentene er int64, men om en error finnes (om et punktum finnes i argumentet, f.eks. `0.2`), vil den prøve float64 før den ikke prøver noe mer og printer en melding. Det er også en alternativ måte å bruke programmet på, ved å sende et tredje argument for å spesifisere type, for eksempel: `./main_sum 2 4 int8`. Koden er skrevet slik at en feilmelding vil komme opp dersom brukeren prøver å skrive inn noe feil. Det går likevel an i teorien å “overflowe” returnverdiene.

Den mest robuste løsningen for å fikse det er å bruke en større bit-dybde på int-typene for å sjekke om verdiene lagt sammen er mer enn en viss verdi. For eksempel, ved int8, sjekkes verdiene sammen som int64 for å være sikker på at de ikke er høyere enn 127 (eller lavere enn -128) til sammen. Dette er dog ikke særlig effektivt å bruke int64 for å sjekke verdien til int8 med tanke på minne, hvorfor ikke bare bruke int64 til alt da?

En annen løsning som ikke bruker en annen type er å sjekke om resultatet er negativt om begge argumentene er positive, eller om resultatet er positivt om begge argumentene er negative. I samme tilfelle med int8, dersom de er negative, vil de på det meste være -128 og -128. Om begge verdiene er negative, må det sjekkes om den resulterende verdien er positivt eller lik 0, om det er tilfellet vil koden gi error. Dersom begge verdiene er positive, vil 127 og 127 gi -2 på det meste. Dette er også error case.
Ved uint-typer, kan denne måten å sjekke ikke brukes. Her regnes ut a * b til en variabel c. Dersom c/b != a, vil dette kaste en error.

Denne errorsjekkingen har blitt implementert i sum/sum.go, samt i main_sum.go


## Oppgave 4

![Sorteringsgraf](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica02-sorting-comparison.png)

Vår implementasjon av Bubblesort sjekker om listen er sortert i hver iterasjon, og vil dermed stoppe. I tillegg, vil algoritmen flytte det største elementet til posisjon slutt av listen på første iterasjon, det nest største elementet til posisjon slutt-1. Derfor setter vi at lengden av iterasjonen blir 1 mindre for hver iterasjon.
Man kan likevel se at våre forsøk på optimisering har liten effekt, her representert i en graf med y-aksen i logaritmisk skala for lesbarhet. Det er uansett åpenbart at Bubblesort er nærmest ubrukelig i forhold til bedre algoritmer som Quicksort.
Vi har ikke nok statistikk for å trekke konklusjoner for big-O notasjon i denne grafen, men vi har ingen grunn til å tro at vår modifiserte Bubblesort-algoritme vil være noe annet enn Θ(n^2), som beskrevet hos http://bigocheatsheet.com/

## Oppgave 5
Her er en YouTube-video som viser stegene for å sjekke informasjon om en kjørende prosess på Windows og Linux: https://www.youtube.com/watch?v=PTXkiFz7M6w
