# Routing-Statico-e-Subnetting-VLSM

Laboratorio pratico Cisco realizzato in **GNS3** per esercitarsi su **subnetting VLSM**, configurazione delle interfacce e **routing statico** con **next-hop**, **exit interface**, **default route** e **route summarization**.

L'obiettivo è permettere a chi legge di ricostruire il laboratorio da zero, seguendo il ragionamento dalla scelta delle subnet fino ai test finali.

---

## 📌 Consegna

1. Organizzare le reti in modo ordinato.
2. Sfruttare le **default route** dove possibile.
3. Configurare le route usando il **next-hop** sui router:
   - Filiale
   - Filiale1
   - Filiale2
   - Gate
4. Configurare le route usando l'**interfaccia di uscita** sui router:
   - IT
   - Research
   - Magazzini
   - Human-Resource

Concetti utilizzati:

- **Network route**: route verso il Network ID di una rete.
- **Host route**: route verso un singolo host, normalmente con prefisso `/32`.
- **Default route**: route usata quando nessuna route più specifica corrisponde alla destinazione.

---

## 📐 Topologia

```text
                         Filiale2
                            |
                           Gate
                          /    \
                         /      \
                  Filiale1------Filiale
                     |             |
                    S1            S2
                   /  \          /  \
                 IT  Research  Mag.  HR
```

- **S1** collega IT, Research e Filiale1.
- **S2** collega Filiale, Magazzini e Human-Resource.
- Filiale1, Gate, Filiale e Filiale2 sono collegati tramite link punto-punto.

---

# 1. Scelta delle subnet

Si utilizza lo spazio privato `192.168.x.x`.

Per scegliere il prefisso si cerca la subnet più piccola capace di contenere tutti gli host richiesti.

Formula:

```text
Host utilizzabili = 2^(bit host) - 2
```

I due indirizzi sottratti sono **Network ID** e **Broadcast**.

| LAN | Host | Subnet scelta | Host utilizzabili | Network ID | Gateway |
| :--- | ---: | :---: | ---: | :--- | :--- |
| Mensa | 700 | `/22` | 1022 | `192.168.0.0` | `192.168.0.1` |
| Uffici-1 | 254 | `/24` | 254 | `192.168.4.0` | `192.168.4.1` |
| Uffici-2 | 120 | `/25` | 126 | `192.168.5.0` | `192.168.5.1` |
| Dock-In | 200 | `/24` | 254 | `192.168.6.0` | `192.168.6.1` |
| Dock-Out | 150 | `/24` | 254 | `192.168.7.0` | `192.168.7.1` |
| Contabilità | 23 | `/27` | 30 | `192.168.8.0` | `192.168.8.1` |
| Head-Hunter | 96 | `/25` | 126 | `192.168.8.128` | `192.168.8.129` |
| Networking | 62 | `/26` | 62 | `192.168.9.0` | `192.168.9.1` |
| Assistenza | 50 | `/26` | 62 | `192.168.9.64` | `192.168.9.65` |
| Test | 40 | `/26` | 62 | `192.168.9.128` | `192.168.9.129` |
| Ricerca e Sviluppo | 35 | `/26` | 62 | `192.168.9.192` | `192.168.9.193` |

Esempio: Networking richiede 62 host. Una `/26` lascia 6 bit agli host:

```text
2^6 - 2 = 62
```

quindi è la dimensione minima corretta.

---

# 2. Reti tra i router

Per i link punto-punto vengono usate subnet `/30`, perché servono solo due indirizzi utilizzabili.

| Collegamento | Network | Lato A | Lato B |
| :--- | :--- | :--- | :--- |
| Filiale1 ↔ Gate | `192.168.10.16/30` | Filiale1 `192.168.10.17` | Gate `192.168.10.18` |
| Filiale1 ↔ Filiale | `192.168.10.20/30` | Filiale1 `192.168.10.21` | Filiale `192.168.10.22` |
| Gate ↔ Filiale | `192.168.10.24/30` | Gate `192.168.10.25` | Filiale `192.168.10.26` |
| Gate ↔ Filiale2 | `192.168.11.16/30` | Gate `192.168.11.17` | Filiale2 `192.168.11.18` |

S1 e S2 non sono link punto-punto: su ciascuno switch ci sono **tre router**, quindi una `/30` non basta. Si usa una `/29`.

### Segmento S1 — `192.168.12.0/29`

| Router | IP |
| :--- | :--- |
| Research | `192.168.12.1` |
| IT | `192.168.12.2` |
| Filiale1 | `192.168.12.3` |

### Segmento S2 — `192.168.13.0/29`

| Router | IP |
| :--- | :--- |
| Filiale | `192.168.13.1` |
| Magazzini | `192.168.13.2` |
| Human-Resource | `192.168.13.3` |

---

# 3. Configurazione delle interfacce

Prima delle route si configurano **tutte le interfacce**, si attivano con `no shutdown` e si verifica che risultino `up/up`.

## IT

```cisco
interface f1/0
 ip address 192.168.9.65 255.255.255.192
 no shutdown

interface f0/1
 ip address 192.168.9.1 255.255.255.192
 no shutdown

interface f0/0
 ip address 192.168.12.2 255.255.255.248
 no shutdown
```

## Research

```cisco
interface f1/0
 ip address 192.168.9.129 255.255.255.192
 no shutdown

interface f0/1
 ip address 192.168.9.193 255.255.255.192
 no shutdown

interface f0/0
 ip address 192.168.12.1 255.255.255.248
 no shutdown
```

## Filiale1

```cisco
interface f0/0
 ip address 192.168.12.3 255.255.255.248
 no shutdown

interface s6/0
 ip address 192.168.10.17 255.255.255.252
 no shutdown

interface s6/1
 ip address 192.168.10.21 255.255.255.252
 no shutdown
```

## Gate

```cisco
interface s6/2
 ip address 192.168.10.18 255.255.255.252
 no shutdown

interface s6/1
 ip address 192.168.10.25 255.255.255.252
 no shutdown

interface s6/0
 ip address 192.168.11.17 255.255.255.252
 no shutdown
```

## Filiale

```cisco
interface s6/1
 ip address 192.168.10.22 255.255.255.252
 no shutdown

interface s6/0
 ip address 192.168.10.26 255.255.255.252
 no shutdown

interface f0/0
 ip address 192.168.13.1 255.255.255.248
 no shutdown
```

## Filiale2

```cisco
interface s6/0
 ip address 192.168.11.18 255.255.255.252
 no shutdown

interface f1/0
 ip address 192.168.0.1 255.255.252.0
 no shutdown

interface f0/0
 ip address 192.168.4.1 255.255.255.0
 no shutdown

interface f0/1
 ip address 192.168.5.1 255.255.255.128
 no shutdown
```

## Magazzini

```cisco
interface f0/1
 ip address 192.168.7.1 255.255.255.0
 no shutdown

interface f1/0
 ip address 192.168.6.1 255.255.255.0
 no shutdown

interface f0/0
 ip address 192.168.13.2 255.255.255.248
 no shutdown
```

## Human-Resource

```cisco
interface f0/1
 ip address 192.168.8.1 255.255.255.224
 no shutdown

interface f1/0
 ip address 192.168.8.129 255.255.255.128
 no shutdown

interface f0/0
 ip address 192.168.13.3 255.255.255.248
 no shutdown
```

Verifica:

```cisco
show ip interface brief
```

Le interfacce utilizzate devono risultare:

```text
Status   Protocol
up       up
```

Prima di configurare il routing conviene fare `ping` tra gli IP delle interfacce direttamente collegate.

---

# 4. Ragionamento sulle route

Un router conosce automaticamente solo le reti **direttamente connesse**, indicate con `C` in:

```cisco
show ip route
```

Le reti remote devono essere aggiunte manualmente.

La strategia è:

- sui router periferici, che hanno una sola strada verso il resto della rete, usare una **default route**;
- sui router centrali, indicare le reti che devono seguire un percorso specifico e sfruttare la default route dove utile;
- usare **summarization** quando più reti consecutive hanno lo stesso next-hop.

---

# 5. Router periferici — Exit Interface

La consegna richiede l'uso dell'**interfaccia di uscita** su IT, Research, Magazzini e Human-Resource.

Tutti hanno una sola direzione verso il resto della topologia, quindi basta una default route.

## IT

```cisco
ip route 0.0.0.0 0.0.0.0 FastEthernet0/0
```

## Research

```cisco
ip route 0.0.0.0 0.0.0.0 FastEthernet0/0
```

## Magazzini

```cisco
ip route 0.0.0.0 0.0.0.0 FastEthernet0/0
```

## Human-Resource

```cisco
ip route 0.0.0.0 0.0.0.0 FastEthernet0/0
```

La route significa: se non esiste una corrispondenza più specifica, inviare il pacchetto attraverso `FastEthernet0/0`.

---

# 6. Filiale2 — Next-Hop

Filiale2 ha una sola uscita verso il resto della rete: **Gate**.

La consegna richiede il next-hop, quindi si usa l'IP di Gate `192.168.11.17`:

```cisco
ip route 0.0.0.0 0.0.0.0 192.168.11.17
```

---

# 7. Filiale — Next-Hop

Filiale ha due possibili direzioni:

- verso **Filiale1** per raggiungere IT e Research;
- verso **Gate** per il resto della topologia.

Le quattro reti `192.168.9.x` sono consecutive e, viste da Filiale, hanno tutte lo stesso next-hop. Possono quindi essere riassunte in `192.168.9.0/24`.

Le reti `192.168.6.0/24` e `192.168.7.0/24` sono entrambe dietro Magazzini e diventano `192.168.6.0/23`.

```cisco
ip route 192.168.9.0 255.255.255.0 192.168.10.21
ip route 192.168.6.0 255.255.254.0 192.168.13.2
ip route 192.168.8.0 255.255.255.224 192.168.13.3
ip route 192.168.8.128 255.255.255.128 192.168.13.3
ip route 0.0.0.0 0.0.0.0 192.168.10.25
```

La default route verso Gate viene usata solo quando nessuna delle route più specifiche corrisponde alla destinazione.

---

# 8. Filiale1 — Next-Hop

Filiale1 deve distinguere le LAN di IT da quelle di Research.

Dietro **IT**:

```text
192.168.9.0/26
192.168.9.64/26
```

Summary:

```text
192.168.9.0/25
```

Next-hop IT: `192.168.12.2`.

Dietro **Research**:

```text
192.168.9.128/26
192.168.9.192/26
```

Summary:

```text
192.168.9.128/25
```

Next-hop Research: `192.168.12.1`.

Le reti di Magazzini e Human-Resource vengono invece raggiunte tramite Filiale (`192.168.10.22`). Tutto il resto viene inviato a Gate.

```cisco
ip route 192.168.9.0 255.255.255.128 192.168.12.2
ip route 192.168.9.128 255.255.255.128 192.168.12.1
ip route 192.168.6.0 255.255.254.0 192.168.10.22
ip route 192.168.8.0 255.255.255.224 192.168.10.22
ip route 192.168.8.128 255.255.255.128 192.168.10.22
ip route 192.168.13.0 255.255.255.248 192.168.10.22
ip route 0.0.0.0 0.0.0.0 192.168.10.18
```

---

# 9. Gate — Next-Hop

Gate è il router centrale.

Tutte le LAN `192.168.9.x` si trovano nella stessa direzione, cioè verso Filiale1, quindi da Gate possono essere riassunte in:

```text
192.168.9.0/24
```

Le reti di Magazzini e Human-Resource sono raggiungibili tramite Filiale.

Le reti Mensa, Uffici-1 e Uffici-2 sono raggiungibili tramite Filiale2.

```cisco
ip route 192.168.9.0 255.255.255.0 192.168.10.17
ip route 192.168.12.0 255.255.255.248 192.168.10.17

ip route 192.168.6.0 255.255.254.0 192.168.10.26
ip route 192.168.8.0 255.255.255.224 192.168.10.26
ip route 192.168.8.128 255.255.255.128 192.168.10.26
ip route 192.168.13.0 255.255.255.248 192.168.10.26

ip route 192.168.0.0 255.255.252.0 192.168.11.18
ip route 192.168.4.0 255.255.254.0 192.168.11.18
```

`192.168.4.0/23` riassume l'area `192.168.4.0 - 192.168.5.255`. Nel laboratorio comprende Uffici-1 e Uffici-2; parte dello spazio rimane inutilizzata.

---

# 10. Perché le summary funzionano

Alcune route possono essere aggregate perché le subnet sono consecutive e hanno lo stesso percorso.

```text
192.168.6.0/24 + 192.168.7.0/24
→ 192.168.6.0/23
```

```text
192.168.9.0/26 + 192.168.9.64/26
→ 192.168.9.0/25
```

```text
192.168.9.128/26 + 192.168.9.192/26
→ 192.168.9.128/25
```

Da Gate e Filiale, tutte e quattro le subnet `192.168.9.x` possono essere viste come:

```text
192.168.9.0/24
```

La route più specifica viene sempre preferita alla default route grazie al **Longest Prefix Match**.

---

# 11. Verifica finale

Controllare la routing table:

```cisco
show ip route
```

Codici principali:

```text
C   Connected
S   Static
S*  Static Default Route
```

Esempio di default route con exit interface:

```text
S* 0.0.0.0/0 is directly connected, FastEthernet0/0
```

Esempio con next-hop:

```text
S* 0.0.0.0/0 via 192.168.11.17
```

Testare poi la raggiungibilità tra reti lontane:

```cisco
ping 192.168.6.1
ping 192.168.8.129
ping 192.168.0.1
ping 192.168.9.1
```

Se un ping non funziona:

```cisco
traceroute <indirizzo-ip>
```

e controllare sul router dove il traffico si ferma:

```cisco
show ip route
```

Bisogna verificare sia il **percorso di andata** sia quello **di ritorno**.

Nel laboratorio tutti i test di connettività sono stati completati con successo.

---

## 💾 Salvataggio

```cisco
write
```

oppure:

```cisco
copy running-config startup-config
```

---

## 🧠 Concetti CCNA trattati

- IPv4 addressing
- Subnetting
- VLSM
- Network ID e Broadcast
- Default Gateway
- Static Routing
- Network Route
- Host Route `/32`
- Default Route
- Next-Hop
- Exit Interface
- Longest Prefix Match
- Route Summarization
- Point-to-Point Networks
- Routing Table
- `ping`
- `traceroute`
- Cisco IOS CLI

---

## 🛠️ Tecnologie

- GNS3
- Cisco IOS
- Visual Studio Code
- Git
- GitHub

---

## 🎯 Scopo

Il laboratorio è stato realizzato a scopo didattico per consolidare le competenze di **subnetting IPv4 e routing statico** richieste nel percorso Cisco CCNA.

Seguendo le sezioni in ordine è possibile ricostruire autonomamente la topologia, scegliere le subnet, configurare tutte le interfacce, aggiungere le route e verificarne il funzionamento.
