# ICA 07, nettverk og protokoller

## Innhold

* [Wireshark](#wireshark)
* [UDP](#udp)
  * [Lokal UDP-pakke](#lokal-udp-pakke)
  * [UDP-pakke over nettverk](#udp-pakke-over-nettverk)
  * [Analyse av UDP](#analyse-av-udp)
  * [Størrelse på UDP-pakke](#størrelse-på-udp-pakke)
* [TCP](#tcp)
  * [Analyse av TCP](#analyse-av-tcp)
  * [Størrelse på TCP-pakke](#størrelse-på-tcp-pakke)
  * [Fragmentering](#fragmentering)
* [UDP vs. TCP](#udp-vs-tcp)
* [Kryptering](#kryptering)
  * [TLS/DTLS](#tlsdtls)
  * [Diffie-Hellman](#diffie-hellman)

## Wireshark

Eksempel på Wireshark-filter: `ip.host == 10.228.41.168`
Her lar vi Wireshark filtrere ut alt som ikke blir mottatt eller sendt fra
IPv4-addressen.

Man kan også legge inn et filter for protokoll: `ip.proto == "UDP"`, men
ettersom at ingen andre forespørsler sendes til den addressen, er det nok med kun
`ip.addr`-filteret

## UDP

### Lokal UDP-pakke

På Linux vil man sette Wireshark til monitoring på loopback-interfacet.
Det er ganske stille der inne, så ekstra filtre er ikke nødvendig her.

Dette er et eksempel på en UDP-pakke sendt innen lokal node.
```
0000   00 00 00 00 00 00 00 00 00 00 00 00 08 00 45 00
0010   00 38 7c 6c 40 00 40 11 c0 46 7f 00 00 01 7f 00
0020   00 01 d7 c0 40 00 00 24 fe 37 4d c3 b8 74 65 20
0030   46 72 20 35 2e 35 20 31 34 3a 34 35 20 46 6c c3
0040   a5 6b 6c 79 70 61
```

### UDP-pakke over nettverk

Dette er et eksempel på en UDP-pakke sendt over nettverket.

```
0000   14 2d 27 6e 56 d1 44 1c a8 4e 01 a3 08 00 45 00
0010   00 38 56 38 00 00 80 11 82 77 0a e4 22 96 0a e4
0020   29 a8 cb bd 40 00 00 24 f9 bc 4d c3 b8 74 65 20
0030   46 72 20 35 2e 35 20 31 34 3a 34 35 20 46 6c c3
0040   a5 6b 6c 79 70 61
```

### Analyse av UDP

I eksempelene er header-lengden 42 bytes lang. Hele pakken er 70 bytes lang, og dermed utgjør headeren 60% av pakken. Headeren øker ikke i noen grad dersom dataen er lengre, og vil derfor utgjøre en mindre prosent når det er tilfellet.

#### MAC header (en del av Ethernet Type II frame)

Dette er det eneste i denne delen av headeren som utgjør en forskjell mellom pakkene lokalt og over nettverk.

I den lokale pakken kan vi se at både Destination og Source er nullet ut, men området er fremdeles nødvendig for å spesifisere type header etter dette området. Grunnen til at Destination/Source ikke har blitt utelatt helt fra headeren i lokalversjonen er fordi det ikke fins noen offset-data i protokollen som forteller om feltene er borte eller ikke. Altså leses 12 bytes, og så de 2 neste bytes for å vite hvilken type header ligger foran (IPv4).

```
[14 2d 27 6e 56 d1]  MAC Destination
[44 1c a8 4e 01 a3]  MAC Source
[08 00]              Ethernet Type (0x0800 == IPv4, og f.eks. 0x86DD == IPv6)
```

#### IPv4 Header

```
[45]                 0100 (IPv4 versjon 4)
                     0101 (IPv4 header length (5*32 bits == 160 bits == 20 bytes)
[00]                 0000 00 - Differentiated Services Codepoint: Default (0)
                     00 - Explicit Congestion Notification: Not ECN-Capable Transport (0)
[00 38]              Length (56 = self + UDP header + data)
[56 38]              Identification
[00]                 Flags
                          0... .... - reserved bit
                          .0.. .... - don't fragment
                          ..0. .... - more fragments
[00]                 Fragment offset
[80]                 Time to live (128)
[11]                 Protocol (17 = UDP) // forteller hva som kommer etter IPv4 header
[82 77]              Header checksum
[0a e4 22 96]        Source
[0a e4 29 a8]        Destination
```

I motsetning til MAC headeren, vil IPv4 source/destination være alltid eksplisitt satt på lokal node (til `7f 00 00 01`/127.0.0.1).

#### User Datagram Protocol header

```
[cb bd]              Source port (0xcbcd == 52157)
[40 00]              Destination port (0x4000 == 16384)
[00 24]              Length (36 = self + data)
[f9 bc]              Checksum
```

#### Data

```
[4d c3 b8 74 65 20
 46 72 20 35 2e 35
 20 31 34 3a 34 35
 20 46 6c c3 a5 6b
 6c 79 70 61]        []bytes("Møte Fr 5.5 14:45 Flåklypa")
```

### Størrelse på UDP-pakke
Wikipedia spesifiserer at UDP pakker har en teoretisk størrelse på 65535 bytes, minus størrelsen på headerdata, det vil si 8 bytes for UDP header, og 20 bytes for IP header, noe som resulterer til 65507 bytes.

Det som er viktig å forstå her er at dette ikke tar Ethernet 2's MTU (maximum transmission unit) i betraktning, noe som kommer inn i spill når man sender pakker over nettverket. Ethernet 2's MTU spesifiserer at pakkene må brytes opp i biter av maks 1500 bytes. Altså, om dataen i en pakke utgjør at pakken er over 1500 bytes, vil den fragmenteres i flere biter, sånn at den passer inn i MTU. Etter dette vil protokoll-stacken sette sammen fragmentene til den "originale" pakkens data.

Med dette i betraktning, kan man si at den største UDP-pakken man kan sende over uten fragmentering er 1472 bytes (1500 - 8 bytes (UDP header) - 20 bytes (IP header)). MAC-headeren (14 bytes) teller ikke med i Ethernet 2's MTU.

Dette er spesifikt viktig å vite i situasjoner hvor man vil oppnå høy ytelse, for eksempel dersom dataen man vil sende over er 1473 bytes, vil man sende to pakker; en som er 1514 bytes lang (1472 bytes data + 8 + 20 + 14), og en som er 35 bytes lang (1 byte data + 8 + 20 + 14).

---

## TCP

### Analyse av TCP

I hver TCP-pakke fins MAC header, og IPv4 header. Disse er lik UDP-versjonen, men Protocol-feltet i IPv4-header er satt til `0x06` for å representere TCP, i stedet for `0x11` som er UDP. Det som følger IPv4 er TCP-header med følgende struktur (med eksempeldata fra en SYN-pakke)

```
[ad 78]             Source port (44408)
[40 01]             Destination port (16385)
[b4 00 c1 d9]       Sequence number
[00 00 00 00]       Acknowledgment number
// Følgende blir representert i bits
[1010]              Header length (0b1010 = 10, 10 * 4 = 40)
[000]               Reserved for future use
[00000010]          Flags [NS, CWR, ECE, URG, ACK, PSH, RST, SYN, FIN]
// Følgende blir representert heksadesimalt
[aa aa]             Window Size value (43690)
[fe 30]             Checksum
[00 00]             Urgent pointer
// Options (har forskjellig betydning alt etter header length)
  // Maximum segment size
    [02]            Option kind (maximum segment size)
    [04]            Option length (4)
    [ff d7]         Option value (65495)
  // TCP SACK Permitted Option
    [04]            Option kind (SACK permitted)
    [02]            Option length (2)
  // Timestamps
    [08]            Option kind (Time stamp option)
    [0a]            Option length (10)
    [14 65 2b 9c]   Option value (Timestamp value)
    [00 00 00 00]   Option value (Timestamp echo reply)
  // No-Operation
    [01]            Option kind (No operation)
  // Window scale
    [03]            Option kind (Window Scale)
    [03]            Option length (3)
    [07]            Option value (shift count, 7)
```


### Størrelse på TCP-pakke

Størrelsen på TCP-pakker er avhengig av Maximum Segment Size-feltet (MSS) i headeren. Dette blir satt under koblingshandshake, og i vårt eksempeltilfelle er 65495 bytes. Dog, dette vil over nettverket, i likhet til UDP, føre til fragmentering på IP-laget, da Ethernet 2's MTU er 1500 bytes.

### Fragmentering

Fragmentering kan også skje på TCP-laget, når MSS er satt til noe lavere. Distinksjonen mellom fragmentering på de to forskjellige lagene er viktig, fordi dersom fragmentering skjer på TCP-laget, vil TCP-protokollen alltid holde styr på hvor mange segmenter har blitt mottatt og sendt (med ACK-pakker). Dersom et segment ikke blir mottatt av serveren, vil klienten prøve på nytt. Dette fører til at alle parter kan være sikre på at alt ble overført ordentlig, noe som ikke skjer på IP-laget.

---

## UDP vs. TCP

TCP er veldig forskjellig fra UDP i den forstand at TCP kontrollerer om data har blitt mottatt eller ei,
hvor UDP ikke bryr seg om dataen den sender har kommet frem til destinasjonen.

I en situasjon hvor UDP sender én pakke med data, vil TCP sende 8 pakker. Her er sekvensen:

```
// 3-stegs koblingshandshake
SYN                 Klient -> Server (klient venter på kobling med server)
SYN, ACK            Server -> Klient (server sier ifra at koblingen er godtatt)
ACK                 Klient -> Server (klient sier ifra at koblingen er godtatt)
// Her vet klient at server er klar for overføring av data
PSH, ACK            Klient -> Server (klient sender data til server)
ACK                 Server -> Klient (server sier ifra at data er mottatt)
// 3-stegs avslutningshandshake
FIN, ACK            Klient -> Server (klient vil avslutte koblingen)
FIN, ACK            Server -> Klient (server godtar avslutningen)
ACK                 Klient -> Server (klient godtar avslutningen)
// Dette er egentlig 4 steg, da server ikke avslutter koblingen før den siste ACK-pakken har blitt mottatt
```

Et spesifikt eksempel på TCP og UDP er i spill som Rocket League (fotball med biler, enkelt sagt), der både TCP og UDP spiller en viktig rolle.

TCP-protokollen brukes til serverkoblinger når en bruker vil søke etter en spillserver hos master-serveren, som får serverinformasjon til spillserver i retur. TCP brukes også inne i spillserveren, for chat-kommunikasjon mellom spillere, samt poenghåndtering (mål og spillerinfo). Der hvor serveren og spillerne må være sikre på at informasjonen er riktig og distribuert, brukes TCP.

UDP spiller en større rolle i selve spillet, da posisjonen på ballen, og hver spiller blir sendt til serveren via UDP, og distribuert tilbake til spillerne. UDP blir også brukt i forbindelse med voice chat. Altså, der hvor høy ytelse trengs, men errorhandling på transportlaget ikke er så viktig, brukes UDP.

---

## Kryptering

Vi har tatt informasjon fra et dokument kalt [Cryptographic Right Answers](https://gist.github.com/tqbf/be58d2d39690c3b366ad) angående kryptering,
noe som har ført oss i noen andre retninger enn hva ICA-dokumentet antyder er lurt, spesielt med tanke på TCP.

### TLS/DTLS

Go har allerede god støtte for Transport Layer Security (TLS) i `crypto/tls`, en type kryptering som er vidspredt spesielt i nettverkslesernes rike.
Siden dette er tilfellet, samt at det er generelt høyt anbefalt blant folk innen krypteringssirkler,
gjør at vi valgte å bruke det i stedet for Diffie-Hellman pluss en annen slags public-key crypto.

TLS er kun støttet over TCP, dog det fins et ekvivalent alternativ for UDP kalt [Datagram Transport Layer Security](https://wiki.wireshark.org/DTLS) (DTLS).
Go har ingen slags førstepartibibliotek for DTLS, og alle tredjepartibibliotek virker verken fullførte eller særlig til å stole på.

### Diffie-Hellman

Siden situasjonen med Go og DTLS ikke er noe å forfølge, har vi like godt en mulighet til å implementere Diffie-Hellman nøkkelutveksling over UDP.
I forbindelse med faktisk kryptering, er valget [NaCl](https://nacl.cr.yp.to/index.html), med `golang.org/x/crypto/nacl/box` for å generere nøkkelpar og
kryptere/dekryptere meldinger.

Løsningen vår gjøres med følgende 4-stegs prosess over nettverket.

1. Klient sender public key til server
  * Klient lager sharedKey med server key og sin egen private key
2. Server sender public key til Klient
  * Server lager sharedKey med klient key og sin egen private key
    * sharedKey er nå lik hos begge parter
3. Klient sender et tilfeldig generert nonce
  * Klienten krypterer hemmelig melding med nonce og sharedKey
5. Klient sender kryptert melding
  * Server bruker nonce og sharedKey for å dekryptere meldingen

Det skal påpekes at UDP er ikke særlig robust til akkurat dette, siden hele prosessen avhenger på at alle pakker mottas, og i riktig rekkefølge.
