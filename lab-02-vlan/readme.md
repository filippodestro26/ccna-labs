# Lab 02 – VLAN e test isolamento

## Obiettivo
Creare due VLAN su uno switch e testare l'isolamento tra host.

## Topologia
PC1 —— Switch —— PC3
PC2 ——/

### VLAN configurate
- VLAN 10: PC1, PC2 (Sales)
- VLAN 20: PC3 (HR)

### Configurazione IP
| PC  | IP           | Subnet |
|-----|--------------|--------|
| PC1 | 192.168.10.10| /24    |
| PC2 | 192.168.10.11| /24    |
| PC3 | 192.168.20.10| /24    |

## Procedura di test

### Test Layer 2
- VLAN create correttamente
- Porte switch assegnate correttamente

### Test Layer 3 – Ping
- Ping PC1 → PC2: riuscito ✅
- Ping PC1 → PC3: fallito ❌ (isolamento VLAN)

### Simulation Mode
- ICMP Echo Request/Reply tra PC1 e PC2 visibili
- Ping tra PC1 e PC3 non arriva

## Conclusioni
VLAN funzionanti e isolamento tra VLAN senza router confermato.
