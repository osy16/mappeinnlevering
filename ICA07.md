# UDP-pakker

## Wireshark

Eksempel på Wireshark-filter: `ip.host == 10.228.41.168`
Her lar vi Wireshark filtrere ut alt som ikke blir mottatt eller sendt fra
IPv4-addressen.

Man kan også legge inn et filter for protokoll: `ip.proto == "UDP"`, men
ettersom at ingen andre forespørsler sendes til den addressen, er det nok med kun
`ip.addr`-filteret


## Lokal UDP-pakke

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

## UDP-pakke over nettverk

Dette er et eksempel på en UDP-pakke sendt over nettverket.

```
0000   14 2d 27 6e 56 d1 44 1c a8 4e 01 a3 08 00 45 00
0010   00 38 56 38 00 00 80 11 82 77 0a e4 22 96 0a e4
0020   29 a8 cb bd 40 00 00 24 f9 bc 4d c3 b8 74 65 20
0030   46 72 20 35 2e 35 20 31 34 3a 34 35 20 46 6c c3
0040   a5 6b 6c 79 70 61
```

---

I eksempelene er header-lengden 42 bytes lang. Hele pakken er 70 bytes lang, og dermed utgjør headeren 60% av pakken. Headeren øker ikke i noen grad dersom dataen er lengre, og vil derfor utgjøre en mindre prosent i det tilfellet.

### MAC header (en del av Ethernet Type II frame)

Dette er det eneste i headeren som utgjør en forskjell mellom pakkene lokalt og over nettverk.

I den lokale pakken kan vi se at både Destination og Source er nullet ut, men området er fremdeles nødvendig for å spesifisere type header etter dette området. Grunnen til at Destination/Source ikke har blitt utelatt helt fra headeren i lokalversjonen er fordi det ikke fins noen offset-data i protokollen som forteller om feltene er borte eller ikke. Altså leses 12 bytes, og så de 2 neste bytes for å vite hvilken type header ligger foran (IPv4).

```
[14 2d 27 6e 56 d1]  MAC Destination
[44 1c a8 4e 01 a3]  MAC Source
[08 00]              Ethernet Type (0x0800 == IPv4, og f.eks. 0x86DD == IPv6)
```

### IPv4 Header

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

### User Datagram Protocol header

```
[cb bd]              Source port (0xcbcd == 52157)
[40 00]              Destination port (0x4000 == 16384)
[00 24]              Length (36 = self + data)
[f9 bc]              Checksum
```

### Data

```
[4d c3 b8 74 65 20
 46 72 20 35 2e 35
 20 31 34 3a 34 35
 20 46 6c c3 a5 6b
 6c 79 70 61]        []bytes("Møte Fr 5.5 14:45 Flåklypa")
```
