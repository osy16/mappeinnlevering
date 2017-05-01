# ICA 03

**Gruppe 4: Crippling Depression**

Brage Sydskogen, Martin Hovet, Tønnes Røren, Marius Kaurin, Kent Daleng, Stian Simonsen

Repository: https://github.com/crippling-depression/is105-ica03

## Innhold
* [Oppgave 1](#oppgave-1)
* [Oppgave 2](#oppgave-2)
* [Oppgave 3](#oppgave-3)
  * [3a](#3a)
  * [3b](#3b)
  * [3c](#3c)
* [Oppgave 4](#oppgave-4)

---

## Oppgave 1
Funksjonene som er skrevet til oppgave a og b fungerer som forventet både på Windows og Linux, og sannsynligvis også på OS X. Hovedårsaken til dette er at ASCII er det nærmeste man kommer en universelt akseptert standard for tegnsett. Selv om ASCII er gammelt, blir det fremdeles brukt ettersom nyere standarder for tegnsett har ASCII i grunnen.

Windows’ `powershell` og `cmd` bruker et tegnsett som heter Code page 437. Dette tegnsettet er ikke fullstending bakvendt kompatibelt med ASCII, da kodepunktene ved 0x01-0x1F, samt 0x7F, har tekstkarakterer i stedet for kontrollkarakterer som i ASCII.
Likevel er de kodepunktene som ASCII bruker til tekst (0x20-0x7E) lik i ASCII og i CP437.
Vanligvis bruker terminal-emulatorene i både OS X og Linux UTF-8 som tegnsett. Samtidig som at UTF-8 har kapasitet for alle mulige karakterer, ble det også utformet slik at det er bakvendt kompatibelt med ASCII. Dermed vil programmet vi har laget vise alle ASCII-tegn som printes.

| Windows (CP437) | Linux (UTF-8) | OS X (UTF-8) |
|---|---|---|
| ![Windows](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica03-windows-term.png) | ![Linux](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica03-linux-term.png) | picture |

Forøvrig, i oppgave b er funksjonen utformet slik at den sjekker input []byte hver byte er innenfor 0x20-0x7E (printable ASCII), og vil dermed ikke printe de tegnene som er utenfor rekkevidden. Dermed er det ikke lenger opp til miljøet som programmet printer ut til, om tegnet skal vises eller ikke. Altså, hvis []byte tilhørende “Hello! :-)” inneholdt en “ulovlig” byte, vil en error oppstå, men brukeren av funksjonen kan velge å ikke håndtere erroren, men bare printe den resulterende “lovlige” stringen.

## Oppgave 2
Under vanlige omstendigheter, vil ikke tegnene fra 0x80-0xFF vises i kommandovinduet. Resultatet er likt på Windows, OS X og Linux, dog om terminalen bruker tegnsett “Code page 1252”, som har karakterer i denne rekkevidden, vil disse bli vist som forventet. Det som er interessant er at tegnsettet Code page 437 som Windows-terminalene bruker, vil ikke noen av tegnene vises (bare en placeholder ?-karakter), selv om standarden sier at det fins tegn i den rekkevidden.

Programmet vi har skrevet printer ut det som er forventet, men det er altså opp til miljøet om tegnene vises. Dette gjelder for både oppgave a og b. På Windows kan man installere `git-bash`, som har muligheter til å forandre brukt tegnsett til CP1252. `cmd` og `powershell` har en kommando som heter `chcp`, som kan skifte tegnsett til Code page 1252, men dette ser uansett ikke ut til å ha noe utslag om hvorvidt tegn kan vises.
På Linux fins det flere terminal-emulatorer som kan skifte tegnsett, og det er også muligheter for å skifte tegnsett til CP1252 i terminalprofilen på OS X.

## Oppgave 3:

### 3a
Kodeeksempel:
```go
func Oppgave3a() {
  s := "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"
  fmt.Printf("%s\n", s)
  fmt.Printf("%q\n", s)
  fmt.Printf("%+q\n", s)
  fmt.Printf("%c\n", s)
  fmt.Printf("%x\n", s)
  fmt.Printf("%s\n", s)
}
```

Output:

![Oppgave 3a output](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica03-output.png)


* `%s` brukes for ikke å tolke bytesekvensen som er definert. Hvis vi bruker Golang får vi en output av: `??=? ⌘`. Dette skjer fordi vi bruker UTF-8 (Unicode). Spørsmålstegnene representere noe Golang ikke forstår. Hvis vi prøver å bruke ISO-8859-1 isteden får vi en output av: `½²= ¼ â`.
* `%q` får en output som ser slik ut: `\xbd\xb2=\xbc ⌘`. Her lar Golang de delene av bytesekvensen som den ikke forstår være som de var. Vi kan se at \xbd\xb2\xbc er det samme som vi ser i bytesekvensen. Golang klarer altså ikke å lese bd, b2. og bc. Hvis vi finner en ASCII tabell med referanser kan vi se at dette er ½, ², og ¼.
* `%+q` finner fram til de forskjellige kodene, selv om man ikke nødvendigvis bruker akkurat det kodesettet. I dette tilfellet vil vi få en output av: \xbd\xb2=\xbc \u2318. Her kan vi se at Golang har funnet koden for ⌘ i Unicode, selv om den vet hva slags tegn det er.
* `%x` skriver alle kodene til de forskjellige tegnene uten `0x` foran. Forskjellen mellom `%X` og `%x` er uppercase og lowercase.
* `%c` vil reagere på string-input, fordi den er ment på typer som kan kastes til integer. Det den vanligvis gjør er å representere et Unicode code point fra en numerisk verdi, men siden vår input ikke er en enkel verdi, får vi en feil (`%!c(string=??=? ⌘)`)

Hvis vi skal få til å ha `½?=? ⌘`. Må vi først å fremst finne fram til hva ½ er i Unicode istedenfor utvidet ASCII. Vi ser at `\u00BD` er ½ i unicode. Derfor tar vi å endrer på litt av koden får til dette:
```go
fmt.Printf("%s\n", "\u00BD\xb2\x3d\xbc\x20\xe2\x8c\x98")
```
Her har vi endret xbd til u00BD. Dermed klarer Golang å tolke hva som står. Outputen her er: `½?=? ⌘`.


### 3b
I denne oppgaven skal vi se på tre forskjellige filer som inneholder tegn for forskjellige tegnsett, “lang01.wl”, “lang02.wl” og “lang03.wl”. For å gjøre livet mest mulig enkelt, åpnet vi hver fil i Atom, og satte tegnsettet til Auto Detect. For “lang01.wl” fikk vi KOI8-R (Kyrillisk), og for de to andre fikk vi Windows 1252 (Vestlig).

Dermed kan vi lage to funksjoner for å konvertere en fil fra et tegnsett til UTF-8, en for Windows 1252 og en for KOI8-R. I disse funksjonene bruker vi funksjoner fra pakkene `"golang.org/x/text/encoding/charmap"` og `"golang.org/x/text/transform"` for å gjøre selve konverteringen, da bruk av `bytes.Replace` ville tatt en god del unødvendig arbeid.

Dermed brukes `fileutils.FileToByteslice()` for å lese filen til []byte, og funksjonene `fileutils.CP1252BytesToUTF8String()` og `fileutils.KOI8RBytesToUTF8String()` for å konvertere til UTF-8 string. Deretter kan vi printe returverdien fra de funksjonene til en terminal med UTF-8 tegnsett.

### 3c
Lik oppgave 3b, trenger vi å finne ut hvilket tegnsett en tekst er. For å gjøre dette enklest mulig, hardkodet vi byteverdiene i “treasure/treasure.txt” til en const string som vi skrev til en fil kalt “treasure/encodedtreasure.txt”. Denne kan vi dermed åpne i Atom og finne ut hvilket tegnsett som brukes. Ettersom Atom rapporterer at dette også er Windows 1252, kan vi bruke akkurat samme const string konvertert til []byte, og samme funksjon som i 3b for å konvertere fra Windows 1252 til UTF-8.


## Oppgave 4
Oppgave 4a vises som forventet ikke riktig på Windows’ `cmd` eller `powershell`, men fungerer fint i `git-bash`, samt på OS X og Linux, så lenge terminalen bruker en font som har støtte for alle kodepunktene som printes.

I oppgave 4b er det ikke lenger opp til terminalvinduet å printe karakterer, men heller en nettleser. Her kan vi se at alle moderne nettlesere støtter UTF-8, ettersom at klokken (`\u23F0`) vises som forventet.
