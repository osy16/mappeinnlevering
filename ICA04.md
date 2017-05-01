# ICA 04

**Gruppe 4: Crippling Depression**

Brage Sydskogen, Martin Hovet, Tønnes Røren, Marius Kaurin, Kent Daleng, Stian Simonsen

Repository: https://github.com/crippling-depression/is105-ica04

## Innhold

* [Oppgave 1a](#oppgave-1a)
* [Oppgave 2b](#oppgave-2b)
* [Oppgave 2c](#oppgave-2c)
* [Oppgave 3c](#oppgave-3c)
* [Oppgave 4](#oppgave-4)
  * [4a](#4a)
  * [4b](#4b)
  * [4c](#4c)
  * [4d](#4d)
  * [4e](#4e)

---

## Oppgave 1a
I oppgave 1 skal vi se på de to filene text1.txt og text2.txt.
Text1.txt er 65 bytes
Text2.txt er 61 bytes
Legg merke til at forskjellige operativsystemer kan ha andre resultater. Disse tallene er ifølge windows 10.
Hvis vi kjører text1 og text2 inn i byteslice (%c) får vi dette resultatet:
```
Tester linjeskift.
Og en til ...
Og en til ...
Og en til ...
```

Forskjellene ser vi når vi bytter %c med %+q.

Text1.txt: `Tester linjeskift.\r\nOg en til ...\r\nOg en til ...\r\nOg en til ...\r\n`

Text2.txt: `Tester linjeskift.\nOg en til ...\nOg en til ...\nOg en til ...\n`

Totalt kan vi se at text1 inneholder /r i tilegg til /n, mens text2 kun inneholder /n.
/n betyr “newline” som i ny linje.
/r betyr “return”

Dette fenomenet kan føre til annerledes formatering ut fra hva slags program du bruker for å lese den. Ta for eksempel notepad for windows.

Text2.txt vil se slik ut: `Tester linjeskift.Og en til ... Og en til ... Og en til ...`

Som vi ser blir alt formatert på en linje fordi `\r` mangler, mens Text1.txt blir seende ut som forventet.


## Oppgave 2b
Informasjonen vi får fra /dev/stdin er:
```
Size: 0 Bytes
Det er ikke et directory
Det er ikke en vanlig fil
Det har unix permissions bits: Dcrw--w----
Det er ikke bare append
Det er en device fil
Det er et Unix character device
Det er ikke et Unix block device
Det er ikke en symbolic link

Information about '/dev/ram0'
Size: 0B, 0.000000KiB, 0.000000MiB, 0.000000000GiB
is not a directory
is not a regular file
Has Unix permission bits: Drw-rw----
Is not append only
Is a device file
Is not a Unix character device
Is Unix block device
Is not a symbolic link
```

## Oppgave 2c
Vi har mindre rettigheter, selv med administratorrettigheter (Access is denied)
Ingen filer er device files, dermed blir det heller ikke Unix character device
eller Unix block device.

## Oppgave 3a
* Grunnleggende operasjoner
  * Opprett en tom fil
  * Forkorte en fil
  * Få fil info
  * Gi nytt navn og flytte en fil
  * Slett filer
  * Åpne og lukke filer
  * Sjekk om filen eksisterer
  * Sjekk lese- og skriverettigheter
  * Endre tillatelser, eierskap, og tidsstempler
  * Lag harde lenker og Symlinks
* Lesing og skriving
  * Kopiere en fil
  * Oppsøk posisjoner i filen
  * Skriv bytes til en fil
  * Hurtigskriv til fil
  * Bruk bufret Writer
  * Les opp til x bytes fra fil
  * Les nøyaktig x bytes
  * Les minst x bytes
  * Les alle bytes av filen
  * Hurtigles hele filen til minne
  * Bruk bufret Reader
  * Les med en skanner
  * Arkivering (Zipping)
  * Arkiv (Zip-filer)
  * Extract (Pakk ut) arkiverte filer
  * Komprimering
  * Komprimere en fil
  * Dekomprimere en fil
* Diverse
  * Midlertidige filer og kataloger
  * Laste ned en fil over HTTP
  * Hashing og sjekksummer

## Oppgave 4

### 4a

| Fakultet | Antall | Symbol | log(1/p) | Kode | # av bits | i % |
|---------:|--------|--------|----------|------|-----------|-----|
| Helse- og idrettsfag | 1829 | A || 000 | 5487 | 17.35% |
| Humaniora og pedagogikk | 1525 | B || 001 | 4575 | 14.47% |
| Kunstfag | 420 | C || 111 | 1260 | 3.99% |
| Teknologi og realfag | 2166 | D || 10 | 4332 | 20.55% |
|Lærerutdanningen | 1506 | E || 110 | 4518 | 14.29% |
| Økonomi og samfunnsvitenskap | 3093 | F || 01 | 6186 | 29.35% |

| | Antall | Symbol|
|-------:|------|---|
| C+E | 1926 | P1 |
| B+A | 3354 | P2 |
| P1+D | 4092 | P3 |
| P2+F | 6447 | P4 |
| P3+P4 | 10539 | P5 |

![Graf av Huffman-kode](https://github.com/crippling-depression/mappeinnlevering/blob/master/assets/ica04-huffman.png)

### 4b

Vi får minst informasjon om økonomi og samfunnsvitenskap fakultetet og teknologi og realfag fakultetet. De er også de største og første. Så blir alt etterpå delt opp videre.

### 4c

`0110000001111`

### 4d

* 000 = 17
* 001 = 14
* 111 = 4
* 10 = 21
* 110 = 14
* 01 = 29

Gjennomsnittslengden for en melding som inneholder fakultets koder for 100 tilfeldig valgte studenter er 250.

### 4e

For å dekode en sekvens av binærtall i Huffman-kode, trenger funksjonen først å ta inn sekvensen for å bearbeide den. For å gjøre dette enklest mulig, vil binærtallene være representert som 0 og 1 tegn i en string, for å dermed bruke metoder på stringen.

Nøkkelen til Huffman-koden vår er hardkodet i programmet i `huffman/huffman.go`, hvor den blir hentet fra getKey()-funksjonen. Dette er en map som har huffman-kode som nøkkel til symbolverdien som sett ovenfor. Grunnen til at den ikke ligger som en konstantvariabel, er på grunn av følgende feilmelding: “const initializer map[string]rune literal is not a constant”. Dermed var det enklest å bare hardkode en funksjon som returnerte verdien.

Det følgende skjer i dekodingsfunksjonen vår:

* Først sjekker algoritmen om lengden på koden er kortere enn 2, dersom den er det, vil koden ikke være gyldig ettersom alle Huffman-kodene er minst 2 i lengde.
* Deretter prøver algoritmen å hente nøkkelverdien av de to første kodene i koden.
  * Dersom denne ikke returnerer noe gyldig verdi, vil den returnere 0. Om dette skjer, skrur vi opp til å lete etter nøkkelverdien av de tre første kodene i koden.
* Før vi prøver å nå tak i nøkkelverdien, sjekker vi om lengden på koden er kortere enn 3. Dersom den er det, er koden igjen ugyldig.
* Den siste sjekken er deretter å hente nøkkelverdien av de tre første kodene i koden. Og igjen om den ikke returnerer en gyldig verdi, vil koden være ugyldig.
* Om koden har lykkes å hoppe gjennom disse ringene, vil det bety at en gyldig verdi finnes, som dermed kan legges til en string som heter “decoded”.
* Den krypterte koden kan da trimmes ned med 2 (eller 3, alt ettersom) tegn fra starten, sånn at algoritmen kan gå på nytt.
* Helt til slutt, etter nedtrimming, testes koden om lengden er lik 0. Om den er det, vil løkken brytes, og den dekodede versjonen returneres.

Kodefunksjonen er betydelig enklere i struktur.

* Inn-sekvensen er gjort til store bokstaver, slik at “abc” blir til “ABC”. Dette bare for å la input være litt mer fleksibelt.
* getKey() blir reversert sånn at det som er nøkler blir til verdier, og det som er verdier blir til nøkler. Dette gjøres sånn at vi kan bruke “samme” map som dekodingsfunksjonen.
* Dermed går det en løkke over inn-sekvensen av bokstaver, og prøver for å se om de har korresponderende koder i nøkkelkartet.
* Dersom dette ikke er tilfellet, vil løkken brytes, da inn-sekvensen ikke kan kodes med bokstaver som ikke finnes i kartet.
  * Derimot, om den korresponderende koden finnes, vil den legges til en string-variabel kalt “encoded”
* Når løkken har lykkes med å gå helt gjennom, vil encoded-stringen bli returnert.

Programmet i `oppgave4/` benytter seg av disse to funksjonene, og kan kjøres etter bygging slik:

* `./huffman -d 101100001000111011001` (for å dekode en sekvens)
  * Decoded sequence: DEADBEEF
* `./huffman -e CAFEBABE`
  * Encoded sequence: 11100001110001000001110
