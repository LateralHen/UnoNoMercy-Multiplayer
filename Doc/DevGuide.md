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

**Effetti:**

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
