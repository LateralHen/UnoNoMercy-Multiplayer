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

**Effects:**

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

   *(This guide will be expanded as development progresses.)*
