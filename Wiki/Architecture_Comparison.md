# 🏗️ UnoNoMercy – Confronto tra Architetture

Questo documento illustra le **due possibili architetture** per il sistema multiplayer di *UnoNoMercy*:

- **Dedicated Server Hosting**
- **Client-Hosted Matches**

Ogni approccio presenta vantaggi, svantaggi e diverse implicazioni sullo sviluppo, sulle prestazioni e sulla scalabilità del progetto.

---

## ⚙️ Panoramica

| Aspetto | **Dedicated Server Hosting** | **Client-Hosted Matches** |
|----------|------------------------------|---------------------------|
| **Chi ospita la partita** | Un server remoto dedicato (VPS o cloud) | Uno dei giocatori diventa “host” |
| **Connessioni** | Tutti i giocatori si collegano al server centrale | Tutti i giocatori si collegano all’host |
| **Stato di gioco** | Gestito dal server | Gestito dal client host |
| **Persistenza** | Possibile salvare partite e statistiche | La partita termina se l’host si disconnette |
| **Esempi noti** | Among Us, Rocket League | Minecraft P2P, giochi LAN locali |

---

## 🧠 Come funzionano

### 🏢 Dedicated Server Hosting

- Esiste un **server centrale autoritativo** che esegue tutta la logica di gioco.
- I giocatori si connettono tramite TCP o WebSocket a questa istanza.
- Il server:
  - Valida ogni azione (giocate, penalità, turni…)
  - Aggiorna e sincronizza lo stato della partita
  - Gestisce disconnessioni e riconnessioni

> ✅ Ideale per partite pubbliche, protezione anti-cheat e scalabilità.  
> ⚠️ Richiede un’infrastruttura di rete stabile e manutenzione continua.

---

### 🧍‍♂️ Client-Hosted Matches

- Un giocatore crea una stanza e diventa **host temporaneo**.  
- L’host agisce come un piccolo server locale, gestendo la logica e inviando aggiornamenti agli altri giocatori.  
- Se l’host abbandona, la partita di solito termina (a meno che non sia implementato un sistema di *host migration*).

> ✅ Perfetto per test rapidi o partite tra amici.  
> ⚠️ Dipende dalla connessione e stabilità dell’host.

---

## 🔍 Pro e Contro

| Criterio | **Dedicated Server** | **Client-Hosted** |
|-----------|----------------------|-------------------|
| **Prestazioni** | Stabili per tutti | Dipendono dalla connessione dell’host |
| **Latenza** | Bilanciata per tutti i giocatori | Ottima per l’host, variabile per gli altri |
| **Sicurezza** | Alta (regole sul server) | Bassa (l’host può manipolare lo stato) |
| **Scalabilità** | Elevata (più istanze server) | Limitata (una partita per host) |
| **Complessità di sviluppo** | Maggiore (serve backend e deploy) | Minore (nessun backend necessario) |
| **Costi** | Richiede hosting o VPS ma esistono opzioni gratuite (replit) | Nessun costo infrastrutturale |
| **Affidabilità** | Partite persistenti e recuperabili | Fragile: se l’host esce, finisce tutto |
| **Compatibilità** | Più semplice da mantenere | Più difficile per NAT/firewall |

---

## 🧩 Differenze di sviluppo

| Aspetto tecnico | **Dedicated Server** | **Client-Hosted** |
|------------------|----------------------|-------------------|
| **Networking** | Server con socket multipli (multi-client) | Peer-to-peer o host con socket singoli |
| **Architettura del codice** | Separazione netta tra client e server | Logica ibrida (l’host funge da server) |
| **Testing** | Richiede un ambiente server locale | Rapido per prototipi |
| **Deploy** | Necessita CI/CD e aggiornamenti remoti | Nessun deploy, solo client aggiornati |
| **Matchmaking** | Centralizzato (database di stanze) | Manuale o tramite inviti tra amici |

---

## 🧭 Quale modello è più adatto a UnoNoMercy?

| Obiettivo | Soluzione consigliata |
|------------|----------------------|
| **Prototipo tra amici** | 🟢 Client-hosted → semplice e veloce da testare |
| **Versione online stabile e pubblica** | 🟢 Dedicated server → stabile, sicura e scalabile |
| **Approccio progressivo** | Iniziare con client-hosted e migrare poi a dedicated server |

---

## 🗺️ Mappa concettuale

```text
DEDICATED SERVER
│
├── Server centrale (autorità unica)
│   ├── Gestisce regole e stato di gioco
│   ├── Sincronizza tutti i client
│   └── Impedisce modifiche e cheat
│
└── Client → inviano input e ricevono stato aggiornato

CLIENT-HOSTED MATCH
│
├── Un client = host
│   ├── Crea la stanza
│   ├── Gestisce turni e regole
│   ├── Aggiorna gli altri giocatori
│   └── La partita termina se l’host esce
│
└── Gli altri client si collegano via P2P o socket diretti
