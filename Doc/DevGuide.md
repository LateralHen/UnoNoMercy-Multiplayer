# ⚙️ UnoNoMercy – Developer Guide

This document explains the internal logic and technical design of **UnoNoMercy-Multiplayer**.

---

## 🧱 Project Overview

UnoNoMercy is designed as an **online multiplayer card game**.  
The architecture (still in design phase) will support either:

- **Client-hosted matches**;
- **Dedicated server hosting** (for long-term scalability).
[differenze tra le opzioni](Architecture_Comparison.md)

## OPTION 1 - CLIENT-HOSTED

In the **client-hosted model**, one player temporarily acts as the host and runs the game logic locally.
To preserve integrity, persistence, and identity across matches without using a server, *UnoNoMercy* implements a **distributed synchronization system** built on local databases and digital signatures.

---

### 🧱 Core Concepts

#### 1. Local Databases (Hybrid Backups)

Each player maintains a small local database (e.g. SQLite) containing:

- **Players table** → profiles of known players (ID, nickname, stats, public key).
- **Matches table** → verified matches (match ID, host ID, signature, summary).

Every database is a **hybrid backup** of the others — data propagates automatically between friends after each match.
If a player loses their data, the system rebuilds their stats as soon as they play again with friends.

---

#### 2. Identity via Passphrase

- Each player defines a personal *passphrase* the first time they start the game.
- This passphrase deterministically generates their digital identity (public/private key pair).
- Using the same passphrase on another device restores the same identity and keeps all past signatures valid.
- No account, login, or export file is required — the passphrase *is* the identity.

-*Effects:**

- Portability → move to another PC, same player ID.
- Security → nobody can impersonate you without your phrase.
- Offline-friendly → works in LAN or full offline mode.

---

#### 3. Match Signing & Verification

- At the end of each game, the **host** creates a signed match file (`match_result.json`).
- The signature is generated with the host’s private key derived from their passphrase.
- All clients verify the signature using the host’s public key before updating their local databases.

This guarantees:

- **Authenticity** → the match truly comes from the real host.
- **Integrity** → any edit invalidates the signature.
- **Uniqueness** → every match has one valid signature and cannot be duplicated.

---

#### 4. Distributed Synchronization

After each match:

1. The host broadcasts the signed match result.
2. Each client verifies it and updates its local database.
3. Players exchange missing verified matches or profiles automatically.

The network behaves as a **peer-to-peer archive**:

- all participants own verified copies of shared matches;
- lost data can be recovered from others;
- tampered records are detected and ignored.

---

#### 5. Integrity & Cross-Check

When players reconnect:

- their databases exchange known `match_id`s and signatures;
- discrepancies (extra matches, invalid signatures) are flagged;
- everyone’s local data converges toward the verified collective history.

If one player’s DB claims matches that others don’t have → those records are considered *unverified*.

---

#### 6. Practical Lifecycle

| Stage          | Player State              | Effect                             |
| -------------- | ------------------------- | ---------------------------------- |
| First launch   | Empty DB, valid identity  | No stats yet                       |
| First match    | Receives verified results | DB starts syncing                  |
| Later sessions | Stats stay consistent     | Automatic updates                  |
| New device     | Enter same passphrase     | Identity and signatures stay valid |

---

### ✅ Benefits Summary

| Category            | Advantage                                   |
| ------------------- | ------------------------------------------- |
| **Simplicity**      | No server, no login, fully peer-to-peer     |
| **Persistence**     | Local DB with automatic distributed backup  |
| **Security**        | Signed matches prevent falsification        |
| **Portability**     | Same identity across devices via passphrase |
| **Resilience**      | Data loss recovered from other players      |
| **Offline Support** | Works in LAN, internet, or offline modes    |

---

### ⚠️ Limitations

- A new device starts with an empty database until the player joins a match.
- The system relies on social trust — it detects manipulation but doesn’t enforce bans.
- No central leaderboard (all stats remain local and shared within the friend group).
- If *every* player loses their data simultaneously, the history is lost (no cloud copy).

---

> 🧭 *In summary:*
> UnoNoMercy’s client-hosted mode forms a small, cooperative P2P network where each player acts as both a participant and a distributed backup node.
> Identity is mental (passphrase), data is social (shared DB), and fairness is guaranteed by cryptographic signatures — all without a server.

---

## 🧩 Core Components

1. **Game Logic Engine**
   - Handles rules, deck management, and turn sequencing.
2. **Networking Layer**
   - Manages player connections, message exchange, and game state sync.
3. **Frontend / UI**
   - Displays the cards, animations, and in-game actions.
4. **Matchmaking / Lobby**
   - (Optional) Allows users to create or join rooms.

---

## 🧠 Planned Features

- Real-time multiplayer (WebSocket / TCP)
- Configurable “No Mercy” rule set
- Persistent match state (resume after disconnect)
- Custom deck variations and themes

---

## 🚧 Development Notes

- Language and framework are **to be decided**.  
- Code should remain modular to allow easy future migration.  
- Open-source contributions and pull requests will be welcomed.

---

Perfetto 👍
Ecco la **versione italiana** del `DevGuide.md` già corretta: tutti gli elenchi ora usano `-` invece di `*` per i bullet list.

---

# ⚙️ UnoNoMercy – Guida per gli Sviluppatori

Questo documento spiega la logica interna e la progettazione tecnica di **UnoNoMercy-Multiplayer**.

---

## 🧱 Panoramica del Progetto

UnoNoMercy è progettato come un **gioco di carte multigiocatore online**.
L’architettura (ancora in fase di progettazione) supporterà due modalità principali:

- **Partite ospitate dal client (client-hosted)**
- **Server dedicato** (per una futura scalabilità a lungo termine)
  [vai alle differenze tra le opzioni](Architecture_Comparison.md)

---

## 🕹️ OPZIONE 1 – CLIENT-HOSTED

Nel modello **client-hosted**, un giocatore agisce temporaneamente come *host* e gestisce la logica di gioco in locale.
Per preservare **integrità**, **persistenza** e **identità** tra le partite senza usare un server, *UnoNoMercy* implementa un sistema di **sincronizzazione distribuita**, basato su database locali e firme digitali.

---

### 🧱 Concetti Fondamentali

#### 1. Database Locali (Backup Ibridi)

Ogni giocatore mantiene un piccolo database locale (es. SQLite) contenente:

- **Tabella dei giocatori** → profili dei giocatori noti (ID, nickname, statistiche, chiave pubblica)
- **Tabella delle partite** → partite verificate (ID partita, ID host, firma, riepilogo)

Ogni database funge da **backup ibrido** degli altri — i dati si propagano automaticamente tra amici dopo ogni partita.
Se un giocatore perde i propri dati, il sistema ricostruisce le sue statistiche non appena gioca di nuovo con altri.

---

#### 2. Identità tramite Passphrase

- Ogni giocatore definisce una *passphrase personale* al primo avvio del gioco.
- La passphrase genera in modo deterministico la propria identità digitale (coppia di chiavi pubblica/privata).
- Usando la stessa passphrase su un altro dispositivo si ripristina la stessa identità e si mantengono valide tutte le firme precedenti.
- Non servono account, login o file di esportazione — la passphrase *è* l’identità.

-*Effetti:**

- Portabilità → puoi cambiare dispositivo mantenendo lo stesso ID.
- Sicurezza → nessuno può impersonarti senza la tua frase.
- Compatibilità offline → funziona anche in LAN o senza connessione.

---

#### 3. Firma e Verifica delle Partite

- Al termine di ogni partita, l’**host** genera un file firmato (`match_result.json`).
- La firma viene creata con la chiave privata dell’host derivata dalla sua passphrase.
- Tutti i client verificano la firma usando la chiave pubblica dell’host prima di aggiornare i loro database.

-*Garantisce:**

- **Autenticità** → la partita proviene realmente dall’host dichiarato.
- **Integrità** → ogni modifica invalida la firma.
- **Unicità** → ogni partita ha una sola firma valida e non può essere duplicata.

---

#### 4. Sincronizzazione Distribuita

Dopo ogni partita:

1. L’host trasmette il risultato firmato.
2. Ogni client lo verifica e aggiorna il proprio database locale.
3. I giocatori si scambiano automaticamente partite o profili mancanti.

La rete si comporta come un **archivio peer-to-peer**:

- Tutti i partecipanti possiedono copie verificate delle partite condivise.
- I dati persi possono essere recuperati dagli altri.
- I record manomessi vengono rilevati e ignorati.

---

#### 5. Integrità e Controllo Incrociato

Quando i giocatori si ricollegano:

- I loro database confrontano gli `match_id` e le firme note.
- Eventuali discrepanze (partite extra o firme non valide) vengono segnalate.
- I dati locali convergono verso una **storia collettiva verificata**.

Se un database contiene partite che gli altri non riconoscono → quelle vengono considerate *non verificate*.

---

#### 6. Ciclo di Vita Pratico

| Fase                | Stato del giocatore            | Effetto                         |
| ------------------- | ------------------------------ | ------------------------------- |
| Primo avvio         | DB vuoto, identità valida      | Nessuna statistica              |
| Prima partita       | Riceve risultati verificati    | Inizia la sincronizzazione      |
| Sessioni successive | Statistiche coerenti           | Aggiornamenti automatici        |
| Nuovo dispositivo   | Inserisce la stessa passphrase | Identità e firme restano valide |

---

### ✅ Riepilogo dei Vantaggi

| Categoria            | Vantaggio                                      |
| -------------------- | ---------------------------------------------- |
| **Semplicità**       | Nessun server o login, rete peer-to-peer pura  |
| **Persistenza**      | DB locale con backup distribuito automatico    |
| **Sicurezza**        | Firme digitali impediscono falsificazioni      |
| **Portabilità**      | Stessa identità su più dispositivi             |
| **Resilienza**       | Dati recuperabili dagli altri giocatori        |
| **Supporto offline** | Funziona in LAN, Internet o totalmente offline |

---

### ⚠️ Limitazioni

- Un nuovo dispositivo parte con un DB vuoto finché non partecipa a una partita.
- Il sistema si basa sulla fiducia tra giocatori — rileva le manipolazioni ma non impone sanzioni.
- Nessuna classifica globale (le statistiche restano locali e condivise solo tra amici).
- Se *tutti* i giocatori perdono i dati contemporaneamente, la cronologia viene persa (non esiste copia nel cloud).

---

> 🧭 *In sintesi:*
> La modalità client-hosted di UnoNoMercy forma una piccola rete P2P cooperativa dove ogni giocatore è al tempo stesso partecipante e nodo di backup distribuito.
> L’identità è mentale (passphrase), i dati sono sociali (DB condiviso), e la correttezza è garantita da firme crittografiche — tutto questo senza alcun server.

---

## 🧩 Componenti Principali

- **Motore Logico di Gioco**
  Gestisce le regole, il mazzo e la sequenza dei turni.

- **Livello di Rete**
  Gestisce connessioni, scambio messaggi e sincronizzazione dello stato di gioco.

- **Interfaccia Grafica (UI)**
  Mostra le carte, le animazioni e le azioni in partita.

- **Matchmaking / Lobby**
  (Opzionale) Permette agli utenti di creare o unirsi a stanze di gioco.

---

## 🧠 Funzionalità Pianificate

- Multiplayer in tempo reale (WebSocket / TCP)
- Set di regole “No Mercy” personalizzabile
- Stato di partita persistente (ripresa dopo disconnessione)
- Mazzi e temi personalizzati

---

## 🚧 Note di Sviluppo

- Linguaggio e framework sono **ancora da decidere**.
- Il codice dovrà restare modulare per facilitare future migrazioni.
- Saranno benvenuti contributi open-source e pull request.

---

(Questa guida verrà ampliata con l’avanzare dello sviluppo.)

---

Vuoi che ti preparo anche il file `.md` pronto da scaricare (così sostituisci direttamente quello nel repo)?
