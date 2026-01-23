# LAB3 - Router-on-a-Stick

## Obiettivo
Configurare il routing inter-VLAN tra VLAN 10 (Staff) e VLAN 20 (Guest) tramite un router Cisco 2811 in modalità Router-on-a-Stick.  
Il laboratorio permette di verificare la comunicazione tra PC nella stessa VLAN e tra VLAN diverse tramite routing.

## Topologia
PC1 (192.168.10.2) --- Fa0/1 ---|
PC2 (192.168.10.3) --- Fa0/2 ---| Switch --- Fa0/4 (trunk) --- Gi0/0 Router 2811
PC3 (192.168.20.2) --- Fa0/3 ---|


- Switch: porte Fa0/1, Fa0/2 → VLAN 10, Fa0/3 → VLAN 20, Fa0/4 → trunk verso router  
- Router: Gi0/0.10 → 192.168.10.1, Gi0/0.20 → 192.168.20.1  

## Configurazione Router
- Subinterfacce Gi0/0.10 e Gi0/0.20 con encapsulation dot1Q  
- Nessun IP sulla porta principale Gi0/0  
- Comandi completi disponibili in `Config_Router.txt`  

## Configurazione Switch
- Porte dei PC in modalità access e assegnate alla VLAN corretta  
- Porta verso router in modalità trunk  
- Comandi completi disponibili in `Config_Switch.txt`  

## PC IPs e Gateway
| PC   | IP           | Subnet       | Gateway       |
|------|--------------|-------------|---------------|
| PC1  | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| PC2  | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |
| PC3  | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |

## Test Ping
- Stessa VLAN (PC1 → PC2):  Successo 100%  
- VLAN diverse (PC1 → PC3):  80–100% in Packet Tracer (simulazione)  
- Ping dal router verso PC1 e PC3:  conferma routing inter-VLAN  

## Note
- Il ping tra VLAN diverse dai PC può mostrare percentuali inferiori al 100% in Packet Tracer.  
- Questo comportamento è un bug di simulazione e non indica errore nella configurazione.

## File inclusi
- `Config_Router.txt` → configurazione completa del router  
- `Config_Switch.txt` → configurazione completa dello switch  
- `PC_IPs.txt` → IP e gateway dei PC  
- `Ping_Results.txt` → risultati dei ping  
- `Topology.png` (opzionale) → screenshot della topologia