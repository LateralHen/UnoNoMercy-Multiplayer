# ⚙️ UnoNoMercy – Developer Guide

This document explains the internal logic and technical design of **UnoNoMercy-Multiplayer**.

---

## 🧱 Project Overview

UnoNoMercy is designed as an **online multiplayer card game**.  
The architecture (still in design phase) will support either:

- **Client-hosted matches**;
- **Dedicated server hosting** (for long-term scalability).

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
