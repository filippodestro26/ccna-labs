# Connessione-Tra-Router-e-DNS

Un laboratorio pratico di networking Cisco simulato su GNS3, incentrato sulla configurazione di una topologia multi-site a 3 router interconnessi tramite collegamenti seriali punto-punto, con integrazione di un Server DNS/DHCP dedicato e la sicurezza base delle periferiche (AAA locale, banner e cifratura credenziali).

---

## 📐 Topologia di Rete

La rete è composta da tre siti principali collegati tramite interfacce Seriali:

* **LAN-1 Site**: Router `Lan-1` con connessione LAN a Switch e PC.
* **LAN-2 Site (Hub Centrale)**: Router `Lan-2` posizionato al centro, collegato via seriale sia a `Lan-1` che a `Lan-3`. La sua LAN ospita uno Switch, un PC e un **Server DNS-DHCP** (istanza IOS dedicata).
* **LAN-3 Site**: Router `Lan-3` con connessione LAN a Switch e PC.

## 🌐 Schema di Indirizzamento IP

| Dispositivo / Interfaccia | Indirizzo IP / Subnet | Descrizione / Ruolo |
| :--- | :--- | :--- |
| **Lan-1** - `e4/0` | `192.168.1.1 /24` | Gateway LAN 1 |
| **Lan-1** - `s6/0` | `192.168.0.1 /30` | Link Seriale verso Lan-2 |
| **Lan-2** - `s6/1` | `192.168.0.2 /30` | Link Seriale verso Lan-1 |
| **Lan-2** - `e4/0` | `192.168.2.1 /24` | Gateway LAN 2 |
| **Lan-2** - `s6/0` | `192.168.0.5 /30` | Link Seriale verso Lan-3 |
| **Lan-3** - `s6/1` | `192.168.0.6 /30` | Link Seriale verso Lan-2 |
| **Lan-3** - `e4/0` | `192.168.3.1 /24` | Gateway LAN 3 |
| **Server DNS-DHCP** - `e4/0` | `192.168.2.2 /24` | Server DNS & Host locale |

---

## ⚙️ Configurazioni Applicate

### 1. Interfacce e Connettività base
Sui router `Lan-1`, `Lan-2` e `Lan-3` sono state configurate le interfacce Ethernet di Gateway e le interfacce Seriali di punto-punto per l'interconnessione WAN.

### 2. Configurazione Client DNS sui Router
Tutti i router sono stati configurati per risolvere i nomi di dominio appoggiandosi al Server DNS di rete (`192.168.2.2`):
```cisco
ip domain-lookup
ip name-server 192.168.2.2
```
### 3. Hardening & Sicurezza Accessi (Configurato su Lan-1)
* **Utenze e Accesso VTY**: Creazione utente amministratore (`Filippo`) e autenticazione locale sulle linee VTY.
* **Gestione Password**:
  * Abilitazione della password di Enable sia in chiaro che cifrata (`enable secret` in formato MD5).
  * Attivazione del servizio `service password-encryption` per la cifratura delle credenziali nel file di configurazione.
* **Banner Personalizzati**: Impostazione dei messaggi `banner login`, `banner exec` e banner generale di benvenuto/manutenzione.

### 4. Configurazione Server DNS (Apparato DNS-DHCP)
L'apparato dedicato funge da Host/Server di rete:
* **Disabilitazione Routing**: Trasformato in un vero e proprio End-Device tramite il comando `no ip routing` e l'impostazione del `ip default-gateway 192.168.2.1`.
* **Servizio DNS ed Host Statici**: Attivazione del demone DNS (`ip dns server`) e mappatura statica nome-IP per l'infrastruttura:
  * `lan-1` -> `192.168.0.1`
  * `lan-2` -> `192.168.0.2`
  * `lan-3` -> `192.168.0.3`
  * `DNS` / `DHCP` -> `192.168.2.2`
  * **Forwarder esterno:** `8.8.8.8`