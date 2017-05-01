# [ICA 05](https://darn.site/)

**Gruppe 4: Crippling Depression**

Brage Sydskogen, Martin Hovet, Tønnes Røren, Marius Kaurin, Kent Daleng, Stian Simonsen

Repository: https://github.com/crippling-depression/is105-ica05

## Innhold

* [Systemarkitektur](#systemarkitektur)
* [Brukerhistorier](#brukerhistorier)
* [Ekstra informasjon](#ekstra-informasjon)
* [Installasjonsinstruksjoner](#installasjonsinstruksjoner)
* [Avhengigheter](#avhengigheter)

----

En konsolideringsside for været i Kristiansand. Ved gjennomsnittsverdier av data fra flere kilder siktes det mot mer "pålitelig" informasjon.
Informasjonen vi får fra gjennomsnittsverdiene er: Temperatur, vindretning og vindstyrke.
Værtypen har vi valgt å ta informasjon fra er Yr.no, fordi de gir oss typen på norsk.
Temperatur er målt i °C, siden det er Temperaturverdien vi benytter i Norge.
Vindstyrke er målt i m/s, som er den mest vanlige verdien for måling av vind.
Vindretningsverdiene er hentet fra [climate.umn.edu](http://climate.umn.edu/snow_fence/components/winddirectionanddegreeswithouttable3.htm)
og forteller i hvilken retning det blåser fra.

Hvis vi holder musen over de forskjellige verdiene, vil vi få informasjonen fra hver enkelt side vi bruker informasjon fra.
Kartet, som er tatt fra Google Maps, er satt til å vise Kristiansand og omegn.
Under kartet finner vi linker til de forskjellige værtjenestene, samt en link til
vårt GitHub repository.

Siden er (stort sett) å finne her: https://darn.site/

Sidene som blir spurt etter informasjon er [Yr.no](http://yr.no/), [OpenWeatherMap](http://openweathermap.org/), [Wunderground](https://www.wunderground.com/), [AccuWeather](http://www.accuweather.com/) og [DarkSky](https://darksky.net/app/).

Henting av informasjon er satt opp som goroutines i `server.go`, da for både å hedre API-restriksjoner, samt å vise et klart eksempel for concurrency.

I bunnen av dette dokumentet finnes instruksjoner for å kunne sette opp dette systemet selv.

## Systemarkitektur

Konsolideringsserveren henter periodevis data fra forskjellige API-er og mellomlagrer informasjonen. Når en bruker vil se på siden, er det data som sist ble hentet som vises.
Dermed blir det ikke unødvendig mange kall til API-ene bak om det er mange brukere som vil se på den konsoliderte siden.
Her er en illustrasjon som viser logikken, både henting og sending av data blir sendt over HTTP eller HTTPS.

![Systemarkitektur-sketch](https://raw.githubusercontent.com/crippling-depression/is105-ica05/master/assets/system-arch-sketch.png)

## Brukerhistorier

### Scenario 1
Ola skal på en helgtur til Kristiansand, der det sies at det ofte er så fint
og deilig vær. Han vil gjerne finne den mest presise værmeldingen, og sjekker dermed
siden vår for å finne ut hva værforholdene er.

### Scenario 2
Lise liker å seile, men i det siste har det vært for lite vind for det. Hun er lei av å dra helt ned til
havna for å så finne ut at vinden en for svak. Hun kan nå ved hjelp av vår side, se hvor kraftig  
vinden er, og hvilken retning den går.

### Scenario 3
Tobias skal ut på piknik med kjæresten sin senere på dagen. Han har blitt fortalt at vår side
er presis når det kommer til værvarsler. Han sjekker siden, men er allikevel skeptisk. Tobias ser
at han enkelt kan navigere seg til Yr.no, som er en side han er mer kjent med.


## Ekstra informasjon

Yr.no's API er kun XML, men ved hjelp av et lite script i `Python` har vi konvertert det til
JSON, dog ikke for offentlig bruk da Yr.no's retningslinjer sier at data må mellomlagres i minst 10 minutter.
På grunn av dette, mellomlagres dataen vi bruker fra Yr.no i 20 minutter før fornyelse.

## Installasjonsinstruksjoner

* Lag en mappe kalt `responses`. De nedlastede json-filene vil bli lagret her for senere bruk
* Kopier `config_example.toml` til `config.toml`
* I `config.toml`, sett inn API nøkler i stedet for `<your api key>`

## Avhengigheter

Bruk `go get` for å installere biblioteker (forutsetter at `$GOPATH` er satt).

* `github.com/BurntSushi/toml`
* `github.com/go-martini/martini`
* `github.com/martini-contrib/render`
