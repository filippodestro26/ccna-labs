# Lab 02 – OSPF Single Area (Area 0) – GNS3

## Obiettivo
Configurare una rete con due router e due PC utilizzando OSPF, verificando la comunicazione tra tutte le subnet

## Topologia
PC1 —— R1 —— R2 —— PC2


**Dettagli interfacce:**

| Dispositivo | Interfaccia | IP Address   | Subnet Mask   | Gateway      |
| ----------- | ----------- | ------------ | ------------- | ------------ |
| PC1         | —           | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| R1          | Fa0/0       | 192.168.10.1 | 255.255.255.0 | —            |
| R1          | Fa0/1       | 10.0.0.1     | 255.255.255.0 | —            |
| R2          | Fa0/0       | 10.0.0.2     | 255.255.255.0 | —            |
| R2          | Fa0/1       | 192.168.20.1 | 255.255.255.0 | —            |
| PC2         | —           | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |

---

## Configurazione Router

### Router 1 (R1)

```bash

enable
configure terminal

! LAN verso PC1
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

! Link verso R2
interface FastEthernet0/1
 ip address 10.0.0.1 255.255.255.0
 no shutdown
exit

! Abilito OSPF
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 0
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
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

! Abilito OSPF
router ospf 1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
end
write memory
```

## Verifica configurazione OSPF

### Su R1
show ip route
show ip ospf neighbor

### Su R2
show ip route
show ip ospf neighbor

## Comandi PC

- PC1: 

```bash
ip 192.168.10.2 255.255.255.0 192.168.10.1
```

- PC2: 

```bash
ip 192.168.20.2 255.255.255.0 192.168.20.1
```

## Procedura di test
1 Verifica collegamenti fisici su GNS3.

2 Ping dal PC1 verso PC2:

    ping 192.168.20.2

3 Ping dal PC2 verso PC1:

    ping 192.168.10.2

## Conclusioni
- La rete funziona correttamente in entrambe le direzioni.
- OSPF stabilisce correttamente l'adiacenza tra R1 e R2.
- Le rotte vengono apprese dinamicamente tramite protocollo OSPF.
- La connettività end-to-end è verificata tramite ICMP.
