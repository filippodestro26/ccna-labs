# Laboratorio-2-Subnetting-VLSM-e-Routing-Statico

Laboratorio pratico di networking simulato su GNS3 basato sull'ottimizzazione degli indirizzi IP tramite **VLSM (Variable Length Subnet Mask)** a partire da un singolo blocco di rete Classful `192.168.1.0/24`. Il progetto prevede la configurazione delle interfacce con personalizzazione MAC, descrizioni e il routing statico tra i due router tramite interfacce d'uscita FastEthernet.

---

## 📐 Topologia di Rete

La rete è composta da due siti principali interconnessi direttamente tramite un link FastEthernet punto-punto:

* **Sito LAN-1**: Router `Lan-1` collegato via Ethernet a un PC tramite Switch.
* **Sito LAN-2**: Router `Lan-2` collegato via Ethernet a un PC tramite Switch.

---

## 📊 Calcolo Subnetting (VLSM)

A partire dall'indirizzo di rete base **`192.168.1.0 /24`**, sono state assegnate le seguenti subnet in base al numero di host richiesti:

1. **LAN-1 (Richiesti 70 Host)**:
   * **Subnet**: `192.168.1.0 /25` (Subnet Mask: `255.255.255.128`)
   * **Range IP utili**: `192.168.1.1` - `192.168.1.126`
   * **Gateway (Lan-1 e4/0)**: `192.168.1.1`

2. **LAN-2 (Richiesti 80 Host)**:
   * **Subnet**: `192.168.1.128 /25` (Subnet Mask: `255.255.255.128`) *(Supporta fino a 126 host)*
   * **Range IP utili**: `192.168.1.129` - `192.168.1.254`
   * **Gateway (Lan-2 e4/0)**: `192.168.1.129`

3. **Link Punto-Punto (Lan-1 ↔ Lan-2)**:
   * **Subnet**: `192.168.1.160 /30` (Subnet Mask: `255.255.255.252`)
   * **Range IP utili**: `192.168.1.161` - `192.168.1.162`

---

## 🌐 Schema di Indirizzamento IP e Parametri Hardware

| Dispositivo | Interfaccia | Indirizzo IP / Subnet | Indirizzo MAC | Descrizione / Ruolo |
| :--- | :--- | :--- | :--- | :--- |
| **Lan-1** | `e4/0` | `192.168.1.1 /25` | `0011.3456.7777` | Gateway LAN-1 |
| **Lan-1** | `f0/0` | `192.168.1.161 /30` | `0012.3456.7778` | Collegamento verso Lan-2 |
| **Lan-2** | `f0/1` | `192.168.1.162 /30` | `0013.3456.7779` | Link di connessione verso Lan-1 |
| **Lan-2** | `e4/0` | `192.168.1.129 /25` | `0014.3456.7781` | Gateway LAN-2 |

---

## ⚙️ Configurazione degli Apparati

### Configurazione Router Cisco (`Lan-1` e `Lan-2`)

```cisco

! 1. CONFIGURAZIONE ROUTER LAN-1

Lan-1# configure terminal

! Configurazione Gateway LAN-1
Lan-1(config)# interface e4/0
Lan-1(config-if)# ip address 192.168.1.1 255.255.255.128
Lan-1(config-if)# description Gateway
Lan-1(config-if)# mac-address 0011.3456.7777
Lan-1(config-if)# no shutdown
Lan-1(config-if)# exit

! Configurazione Interfaccia verso Lan-2
Lan-1(config)# interface f0/0
Lan-1(config-if)# ip address 192.168.1.161 255.255.255.252
Lan-1(config-if)# description Collegamneto verso Lan-2
Lan-1(config-if)# mac-address 0012.3456.7778
Lan-1(config-if)# no shutdown
Lan-1(config-if)# exit

! Routing Statico verso la LAN-2 tramite interfaccia d'uscita
Lan-1(config)# ip route 192.168.1.128 255.255.255.128 f0/0
Lan-1(config)# end
Lan-1# write

! 2. CONFIGURAZIONE ROUTER LAN-2

Lan-2# configure terminal

! Configurazione Interfaccia verso Lan-1
Lan-2(config)# interface f0/1
Lan-2(config-if)# ip address 192.168.1.162 255.255.255.252
Lan-2(config-if)# description Link verso Lan-1
Lan-2(config-if)# mac-address 0013.3456.7779
Lan-2(config-if)# no shutdown
Lan-2(config-if)# exit

! Configurazione Gateway LAN-2
Lan-2(config)# interface e4/0
Lan-2(config-if)# ip address 192.168.1.129 255.255.255.128
Lan-2(config-if)# description Gateway-Lan2
Lan-2(config-if)# mac-address 0014.3456.7781
Lan-2(config-if)# no shutdown
Lan-2(config-if)# exit

! Routing Statico verso la LAN-1 tramite interfaccia d'uscita
Lan-2(config)# ip route 192.168.1.0 255.255.255.128 f0/1
Lan-2(config)# end
Lan-2# write
```
### 3. Configurazione degli Host (VPCS)

**PC1 (Sito LAN-1):**

```text
PC1> ip 192.168.1.10 255.255.255.128 192.168.1.1
```

**PC2 (Sito LAN-2):**

```text
PC2> ip 192.168.1.140 255.255.255.128 192.168.1.129
```

### 4. Verifiche e Diagnostica

**Stato delle interfacce:**

```cisco
show ip interface brief
```

**Verifica configurazione delle interfacce:**

```cisco
show running-config interface e4/0
show running-config interface f0/0
```

**Test di connettività:**

```text
PC1> ping 192.168.1.140
```