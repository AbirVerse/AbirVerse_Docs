# NPC Customization & Personality Guide

Adding or modifying NPCs in LuxuryVerse requires **zero code changes** to the core engines. All NPCs are auto-discovered from `ServerScriptService.Configs.NPCs`.

---

## 1. Anatomy of an NPC Configuration

To create a new NPC, simply create a new `ModuleScript` inside `ServerScriptService.Configs.NPCs` (e.g., `Concierge.luau`) and return a dictionary:

```luau
--!strict
local Concierge = {
    Name = "Arthur",
    Role = "Head Concierge",
    Icon = "🛎️",
    ThemeColor = Color3.fromRGB(212, 175, 55),
    ModelTag = "ConciergeNPC",     -- Tag or Model name in workspace
    ParentArea = "HotelLobby",
    InteractionMode = "UseGUI",    -- "UseGUI" (modal conversation) or "UseChat" (overhead bubbles)
    TriggerDistance = 18,
    CloseDistance = 26,
    
    NaturalMovement = {
        Enable = true,
        Area = "WalkBounds",       -- Boundary folder/part name in workspace
        RestMinSec = 3,
        RestMaxSec = 7,
        WalkSpeed = 5,
        ApproachSpeed = 7,
        ApproachDistance = 6,      -- Distance to stop from player when approaching
    },
    
    Greeting = "Welcome to the Royal Palace. How may I be of service today?",
    ReturnGreetings = {
        "Welcome back %s! Need a reservation or luggage assistance?",
        "Good day %s! A pleasure to see you again.",
    },
    FallbackReplies = {
        "Certainly! Please allow our staff to assist you.",
        "A magnificent choice! Let me know if you need any accommodations.",
        "Right away, sir! Enjoy your stay at our grand hotel.",
    },
    
    Personality = "You are Arthur, the distinguished and polite head concierge of a 5-star grand luxury hotel. You speak with high-class British elegance, courtesy, and poise. Keep your responses under 2 sentences. Always remain helpful and address guests with refined respect.",
    
    QuickChips = {
        "Book a suite",
        "Where is the boutique?",
        "Where is the lounge bar?",
        "Hotel amenities",
    },
}

return Concierge
```

---

## 2. Interaction Modes: `UseGUI` vs. `UseChat`

Each NPC can be configured with one of two distinct interaction models via `InteractionMode` in their config file:

### 1. `UseGUI` (Modal & Clickable Options)
- **When to Use**: Best when you want to guide players with pre-set options, menus, and clickable chips rather than forcing them to type everything manually.
- **How It Works**: Opens a luxury frosted dialogue modal on the player's screen.
- **Key Advantages**:
  - **Clickable QuickChips**: Players can simply tap predefined options—such as ordering specific drinks from Marco (*"Velvet Stardust"*, *"Golden Eclipse"*) or asking about hotel amenities—with a single click.
  - **Console & Mobile Friendly**: Perfect for mobile, tablet, and gamepad players where typing long sentences in chat is slow or tedious.
  - **Full History**: Preserves the dialogue scrollback and shows real-time typing indicators.

### 2. `UseChat` (Direct In-World Spatial Chat)
- **When to Use**: Best when you want a realistic, seamless, and immersive experience that feels like walking up and chatting directly with a real person or friend.
- **How It Works**: Uses Roblox's native overhead chat bubble system without opening any screen-blocking UI.
- **Key Advantages**:
  - **Full Player Mobility**: Players can converse while walking around, exploring the lounge, or sitting on furniture.
  - **Immersive Presence**: The NPC turns to face the player, approaches naturally, and speaks in speech bubbles directly above their head in 3D world space.
  - **Casual & Ambient**: Ideal for roaming stylists, concierge greeters, and ambient venue staff.


---

## 3. Natural Movement & Proximity Behavior

LuxuryVerse NPCs are not static mannequins; they roam naturally within designated boundaries:
- **`Area`**: Place a Part or Folder named `WalkBounds` in the NPC's area. The NPC chooses random points inside this bounding box and walks at `WalkSpeed`.
- **`RestMinSec` / `RestMaxSec`**: Duration the NPC pauses between roaming paths.
- **`ApproachDistance`**: When a player walks within `TriggerDistance`, the NPC stops wandering, turns toward the player, and approaches to this exact distance. Setting `ApproachDistance = 8` ensures the NPC leaves polite personal space and never crowds or collides with the player.
- **Animation Scope**: NPCs use lightweight procedural joint orientation (head tracking toward players) and basic built-in gestures. The framework intentionally uses standard animations to remain universally compatible and lightweight, without heavy custom mocap rigs. Developers can easily attach custom AnimationTracks if desired.

---

## 4. Prompt Engineering & Personality Guardrails

When writing the `Personality` string:
1. **Define the persona clearly**: Name, occupation, luxury aesthetic tone, and background.
2. **Length constraint**: Add `"Keep your replies strictly under 2 sentences."` to keep interactions fast and dynamic for mobile and console players.
3. **Style matching**: You can prompt the NPC to adapt: `"Dynamically match the player's slang if casual, or mirror their poise if formal."`
4. **Policy guardrails**: Always include: `"Never mention illegal or regulated content such as alcohol, drugs, or weapons. All drinks served are non-alcoholic refreshments."`

---

## 5. Realistic AI Expectations & Edge Cases

When deploying AI NPCs in Roblox, keep in mind that external LLMs generate dialogue probabilistically, not through hardcoded game states:

- **Visual Sensory Awareness**:
  - NPCs do not possess computer vision or real-time awareness of every specific accessory or body part a player is currently wearing.
  - *Example*: If a player is already wearing a cowboy hat and asks Grace (the stylist) for a "cowgirl look", Grace might suggest wearing a cowboy hat because she generates suggestions from text concepts rather than scanning the player's 3D character mesh.
- **Dialogue vs. Game Object Disconnects**:
  - An NPC might mention an imaginative flavor or creative concept in dialogue that is not an actual game item (e.g. Marco enthusiastically describing a blueberry fizz while the server dispenses an orange drink based on keyword match).
- **Developer Prompt Tuning**:
  - The more explicit, specific, and constrained you make an NPC's system prompt in their config file, the more reliably they will adhere to intended boundaries.
  - Always instruct NPCs to stick strictly to their defined menus, services, and inventory.

