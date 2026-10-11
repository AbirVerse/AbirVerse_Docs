# Script Directory & Security Architecture Guide

This guide details the complete LuxuryVerse script layout, the responsibilities of each component, and the strict security practices that protect your experience from tampering, exploiters, and prompt theft.

---

## 1. Script Directory Map

```
LuxuryVerse
├── ServerScriptService
│   ├── AIService.server.luau            # Server orchestrator for NPC AI dialogue
│   ├── NPCController.server.luau        # NPC roaming, proximity prompts, interaction hooks
│   ├── TryOn.server.luau                # Server try-on event router for mannequins & catalog
│   ├── Mannequins.server.luau           # Mannequin display stands, auto-rotation timer
│   ├── ShelfAccessories.server.luau     # Interactive boutique shelf props & try-ons
│   ├── LobbyAutoDoors.server.luau       # Proximity-based automated doors
│   ├── LobbyAutoSit.server.luau         # Smooth seating trigger for luxury lounge seats
│   ├── Configs
│   │   ├── AISettings.luau              # AI credentials, secret keys, model rosters, fallbacks
│   │   ├── BoutiqueSettings.luau        # Mannequin rotation intervals, default bundle IDs
│   │   ├── Outfits.luau                 # Curated outfits, asset IDs, categories
│   │   ├── Accessories.luau             # Individual accessories & shelf items
│   │   └── NPCs
│   │       ├── Grace.luau               # Stylist NPC config, prompt, fallback dialogue
│   │       ├── Marco.luau               # Bartender NPC config, drink menu, dialogue
│   │       └── Clara.luau               # Concierge NPC config & guest services
│   ├── Packages
│   │   ├── AV-AIEngine.luau             # Multi-provider LLM caller, failover engine, memory
│   │   ├── AV-OutfitEngine.luau         # Wardrobe applicator, 3D face detector, pricing cache
│   │   └── AV-NaturalNPC.luau           # Procedural NPC gestures, head tracking, eye contact
│   └── Utility
│       └── TypeValidation               # Runtime configuration type validators
│
├── ReplicatedStorage
│   ├── Packages
│   │   └── AV-Logger.luau               # Dual-environment logging module
│   ├── Remotes                          # Secured RemoteEvents & RemoteFunctions
│   └── DrinkTemplates                   # Non-alcoholic drink tool models & sip scripts
│
└── StarterPlayer.StarterPlayerScripts
    ├── AIClient.client.luau             # Client dialogue interface, typing indicators
    ├── BoutiqueClient.client.luau      # Luxury 6-tab boutique UI, price calculator, avatar sync
    └── DrinkClientHandler.client.luau   # Client-side drink holding animations & audio
```

---

## 2. Component Breakdown

### ServerScriptService (Server-Authoritative Only)

| File | Type | Role & Responsibilities |
| :--- | :--- | :--- |
| `AIService.server.luau` | Script | Listens for player chat inputs, validates interaction range, invokes `AV-AIEngine`, sanitizes output via `TextService`, and replicates dialogue to the client. |
| `NPCController.server.luau` | Script | Initializes NPC models in Workspace, configures ProximityPrompts/ClickDetectors, runs roaming logic, and invokes `AV-NaturalNPC` gestures. |
| `TryOn.server.luau` | Script | Connects boutique interaction remotes to `AV-OutfitEngine` for applying outfits and reverting player avatars. |
| `Mannequins.server.luau` | Script | Manages physical mannequin podiums, assigns visual looks from `Outfits.luau`, and rotates mannequins on a configurable timer. |
| `ShelfAccessories.server.luau` | Script | Populates interactive boutique shelves with accessories, creating click prompts that allow players to try individual items. |
| `LobbyAutoDoors.server.luau` | Script | Opens and closes automatic glass doors based on player distance. |
| `LobbyAutoSit.server.luau` | Script | Handles luxury sofa and armchair interaction seats. |
| `Packages.AV-AIEngine` | ModuleScript | Communicates with external LLM APIs via `HttpService`, executes failover cascades, maintains conversation memory, and enforces offline fallbacks. |
| `Packages.AV-OutfitEngine` | ModuleScript | Server authority for avatar modifications. Queries `HumanoidDescription`, handles layered clothing, detects 3D faces, hides original heads, and caches item prices. |
| `Packages.AV-NaturalNPC` | ModuleScript | Animates NPCs naturally (smooth head tracking, procedural talking gestures, and idle stance adjustments). |
| `Configs.AISettings` | ModuleScript | Defines secret key names, active provider sequence, temperature, max tokens, and rate limits. |
| `Configs.NPCs.*` | ModuleScript | Defines individual character personas, backstories, greetings, and fallback phrases. |

---

### ReplicatedStorage (Replicated Shared Assets)

| File | Type | Role & Responsibilities |
| :--- | :--- | :--- |
| `Packages.AV-Logger` | ModuleScript | Unified logging utility used on both server and client. Synchronizes log levels and redacts sensitive data. |
| `Remotes` | Folder | Contains strictly scoped `RemoteEvent` and `RemoteFunction` instances. Clients pass only request intents (e.g., item IDs, prompt messages). |
| `DrinkTemplates` | Folder | Contains non-alcoholic drink models equipped with `DrinkScript.server.luau` for interactive consumption. |

---

### StarterPlayerScripts (Client-Side Only)

| File | Type | Role & Responsibilities |
| :--- | :--- | :--- |
| `BoutiqueClient.client.luau` | LocalScript | Renders the boutique window, navigates 6 tabs, calculates dynamic total look prices, and fires wear/revert remotes. |
| `AIClient.client.luau` | LocalScript | Renders the chat HUD, displays real-time typing indicators, and submits player dialogue. |
| `DrinkClientHandler.client.luau` | LocalScript | Plays drinking sounds, particles, and holding animations when the player activates a beverage tool. |

---

## 3. Security Architecture: Server vs. ReplicatedStorage

A core security standard in LuxuryVerse is the strict isolation of AI configuration and credentials inside `ServerScriptService`.

```
        CLIENT (Untrusted)                     SERVER (Authoritative)
┌─────────────────────────────────┐      ┌─────────────────────────────────┐
│ StarterPlayerScripts            │      │ ServerScriptService             │
│ - BoutiqueClient                │      │ - AV-AIEngine (HttpService)     │
│ - AIClient                      │      │ - AISettings (Secrets & Keys)   │
│ - DrinkClientHandler            │      │ - NPC Personalities & Prompts   │
└────────────────┬────────────────┘      └────────────────▲────────────────┘
                 │                                        │
                 │ Fires Remotes (Intent / String only)   │ Executes Secure Logic
                 ▼                                        │
┌─────────────────────────────────────────────────────────┴────────────────┐
│ ReplicatedStorage.Remotes                                                │
│ - TryOn, RevertOutfit, SendAIMessage (No Secret Data Exposed)            │
└──────────────────────────────────────────────────────────────────────────┘
```

### Why AI Settings MUST NEVER Be in ReplicatedStorage

1. **Client Decompilation & Reverse-Engineering**:
   - Any script, module, or string placed in `ReplicatedStorage` is replicated directly to the memory of every connected player.
   - Exploiters using decompilers can read all source code, comments, tables, and asset references inside `ReplicatedStorage`.
   - By keeping `AISettings.luau` and `NPCs/*.luau` in `ServerScriptService`, client exploiters receive **zero bytes** of your AI configuration or system prompts.

2. **API Secret Protection**:
   - Roblox's `HttpService:GetSecret()` is only callable from the server.
   - If secret names or endpoint URLs are exposed in `ReplicatedStorage`, attackers learn your backend provider lineup, secret identifier names, and endpoint routes.

3. **Prompt Theft & Intellectual Property**:
   - Crafting realistic, luxury NPC personalities and jailbreak defenses requires extensive prompt engineering.
   - Keeping prompts on the server prevents competitors from copying your character definitions and system instructions.

4. **Jailbreak & Injection Defense**:
   - When players chat with an NPC, the client only transmits the raw string message across a `RemoteEvent`.
   - The server validates the distance between the player and NPC, enforces rate limits, wraps the message inside the server-authoritative system prompt, and filters the result using Roblox's `TextService:FilterStringAsync`.
   - The client never has access to the full conversation payload sent to the LLM.

5. **Commerce & Inventory Authority**:
   - `BoutiqueClient` cannot directly equip assets to the player's character.
   - It sends an asset ID or outfit name to `AV-OutfitEngine` via `RemoteEvent`.
   - The server verifies asset validity and applies it using server-side `HumanoidDescription` APIs, preventing malicious outfit injections.
