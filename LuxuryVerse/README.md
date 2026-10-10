# LuxuryVerse Developer Guide & Architecture

Welcome to the **LuxuryVerse** developer documentation. LuxuryVerse is an enterprise-grade commercial framework for Roblox that integrates intelligent AI-driven NPCs, interactive luxury hospitality mechanics, curated fashion boutiques, real-time avatar customization, and catalog commerce.

---

## Architecture Overview

LuxuryVerse follows a decoupled, modular design pattern:
- **Centralized Configuration**: All game tuning, API credentials, NPC behaviors, and boutique items live inside `ServerScriptService.Configs`.
- **Standalone Core Engines**: Modular engine packages (`AV-AIEngine`, `AV-OutfitEngine`, `AV-Logger`, `AV-NaturalNPC`) execute independently with zero hardcoded dependencies.
- **Client Controllers**: Lightweight client scripts in `StarterPlayerScripts` communicate strictly via secured RemoteEvents and RemoteFunctions in `ReplicatedStorage.Remotes`.

```
ServerScriptService
├── Configs
│   ├── AISettings.luau          # Multi-provider AI pools, models, and failover
│   ├── BoutiqueSettings.luau    # Stand rotation, default mannequin body parts
│   ├── Outfits.luau             # Curated looks and UGC catalog items
│   ├── Accessories.luau         # Standalone accessories and shelf items
│   └── NPCs
│       ├── Grace.luau           # Stylist NPC config & custom dialogue
│       ├── Marco.luau           # Bartender NPC config & drink menu
│       └── Clara.luau           # Concierge NPC config & guest services
└── Packages
    ├── AV-AIEngine.luau         # AI dialogue, provider rotation, offline fallback
    ├── AV-OutfitEngine.luau     # Real-time wardrobe try-on, 3D face handling, checkout
    └── AV-NaturalNPC.luau       # Procedural NPC gestures, head tracking, eye contact

StarterPlayer.StarterPlayerScripts
├── BoutiqueClient.client.luau   # Full luxury catalog UI & wardrobe manager
├── AIClient.client.luau         # Dialogue interface & typing indicators
└── DrinkClientHandler.client.luau # Tool holding & interactive drink sips
```

---

## Documentation Index

1. **[Roblox Policy & 18+ Experience Guidelines](01-Roblox-Policy-And-Age-Rating.md)**
   - Understanding mandatory 18+ questionnaire flags for Generative AI.
   - Strict content filtering, chat sanitization, and prohibited content rules (alcohol, drugs, tobacco).
2. **[AI NPC Engine & Multi-Provider Setup](02-AI-Engine-And-Providers.md)**
   - API Secret setup in Roblox Creator Hub Credentials.
   - Free vs. paid AI providers (Google AI Studio, Groq, DeepSeek, OpenRouter, OpenAI, Anthropic).
   - Dynamic fallbacks and zero-credit automatic offline mode.
3. **[NPC Customization & Personality Guide](03-NPC-Personalities-And-Configs.md)**
   - Creating new NPCs without modifying engine code.
   - Crafting personalities, system prompts, greetings, and fallback phrases.
   - Interaction modes (`UseGUI` vs `UseChat`).
   - Realistic AI expectations (visual awareness limitations and prompt tuning).
4. **[Boutique & Wardrobe System](04-Boutique-And-Wardrobe-System.md)**
   - Mannequin stands, rotation timers, and click-to-wear try-on.
   - The 6-tab Boutique catalog interface.
   - 3D face detection, dynamic head compatibility, and head transparency handling.
   - Full avatar reset workflow.
   - In-memory server price caching, live purchase updates, and rate-limit throttling prevention.
5. **[Bar & Beverage Service System](05-Bar-And-Beverage-System.md)**
   - Non-alcoholic luxury refreshment menu.
   - Keyword-matched drink delivery & realistic AI edge cases.
   - Basic drink tool consumption mechanics & animation scope.
6. **[Script Directory & Security Architecture](06-Script-Directory-And-Security.md)**
   - Full inventory of all scripts across `ServerScriptService`, `ReplicatedStorage`, and `StarterPlayerScripts`.
   - Security principles: Why AI settings, secret names, and prompts must remain server-side.
   - Exploit prevention and anti-decompilation design.
7. **[Logging, Dev Mode vs. Release Mode](07-Logger-And-Release-Modes.md)**
   - Developer mode diagnostics vs. production zero-log silence.
   - API key privacy safeguards.
   - Performance optimizations.
