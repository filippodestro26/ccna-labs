# Lab 01 – Static Routing GNS3

## Obiettivo
Configurare una rete base con due router e due PC utilizzando rotte statiche, verificando la comunicazione tra tutte le subnet.

## Topologia
PC1 —— Router1 —— Router2 —— PC2


**Dettagli interfacce:**

| Dispositivo | Interfaccia | IP Address      | Subnet Mask      | Gateway      |
|------------|-------------|----------------|-----------------|-------------|
| PC1        | —           | 192.168.1.2    | 255.255.255.0   | 192.168.1.1 |
| R1         | Fa0/0       | 192.168.1.1    | 255.255.255.0   | —           |
| R1         | Fa0/1       | 10.0.0.1       | 255.255.255.0   | —           |
| R2         | Fa0/0       | 10.0.0.2       | 255.255.255.0   | —           |
| R2         | Fa0/1       | 192.168.2.1    | 255.255.255.0   | —           |
| PC2        | —           | 192.168.2.2    | 255.255.255.0   | 192.168.2.1 |

---

## Configurazione Router

### Router 1 (R1)

```bash

enable
configure terminal

! LAN verso PC1
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

! Link verso R2
interface FastEthernet0/1
 ip address 10.0.0.1 255.255.255.0
 no shutdown
exit

! Rotta statica verso LAN di R2
ip route 192.168.2.0 255.255.255.0 10.0.0.2

end
write memory
```

### Router 2 (R2)

```bash

enable
configure terminal

! Link verso R1
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 no shutdown
exit

! LAN verso PC2
interface FastEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

! Rotta statica verso LAN di R1
ip route 192.168.1.0 255.255.255.0 10.0.0.1

end
write memory
```

## Comandi PC

- PC1: 

```bash
ip 192.168.1.2 255.255.255.0 192.168.1.1
```

- PC2: 

```bash
ip 192.168.2.2 255.255.255.0 192.168.2.1
```

## Procedura di test
1 Verifica collegamenti fisici.

2 Ping dal PC1:

    ping 192.168.2.2

3 Ping dal PC2:

    ping 192.168.1.2

## Conclusioni
- La rete funziona correttamente in entrambe le direzioni.
- L’uso di rotte statiche è sufficiente per reti piccole.
- Tutti i dispositivi sono configurati con IP, subnet mask e gateway corretti.
- Questo lab può essere usato come base per aggiungere VLAN, NAT o routing dinamico nei lab successivi.
