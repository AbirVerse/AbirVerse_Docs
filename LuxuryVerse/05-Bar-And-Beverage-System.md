# Luxury Bar & Beverage Service System

The luxury bar system provides an interactive hospitality experience where players converse with an AI bartender and receive custom animated refreshments.

---

## 1. Script & Asset Placement

| Component | Location | Role & Security |
| :--- | :--- | :--- |
| **`Marco` Config** | `ServerScriptService.Configs.NPCs.Marco` | **Server-Only**. Defines drink menu, glass types, fallback phrases, and AI persona. |
| **`AV-AIEngine`** | `ServerScriptService.Packages.AV-AIEngine` | **Server-Only**. Analyzes player messages for drink orders, triggers bartender gestures, and clones drink tools. |
| **`DrinkTemplates`** | `ReplicatedStorage.DrinkTemplates` | Replicated storage folder containing 6 interactive `Tool` models. |
| **`DrinkClientHandler`** | `StarterPlayer.StarterPlayerScripts.DrinkClientHandler` | **Client-Side**. Listens for tool equipping and handles drinking animations/sips. |

---

## 2. The 100% Non-Alcoholic Luxury Menu

To ensure strict compliance with Roblox moderation policies, all drink names and liquid colors are matched to non-alcoholic luxury artisan refreshments:

| Drink Name | Liquid & Visual Aesthetic | Glass Type | Flavor Profile / Concept |
| :--- | :--- | :--- | :--- |
| **Golden Eclipse** | Pale golden sparkling with effervescence | Flute Glass | Sparkling white grape, gold leaf, elderflower |
| **Midnight Sapphire** | Deep ocean blue with crystal clarity | Crystal Glass | Blue curacao syrup (non-alcoholic), butterfly pea flower |
| **Velvet Stardust** | Rich amber caramel and warm tones | Rocks Glass | Smoked vanilla bean, spiced caramel apple cider |
| **Crimson Mirage** | Deep ruby and crimson red | Stemmed Glass | Pomegranate, hibiscus tea, crushed blood orange |
| **Emerald Frost** | Chilled emerald green with frosty rim | Chilled Glass | Kiwi-lime elixir, crushed fresh mint, cucumber |
| **Solar Radiance** | Warm sunrise orange-gold | Highball Glass | Blood orange, passionfruit, sparkling yuzu splash |

---

## 3. Serving Mechanics & Keyword Matching

Drink delivery is handled via server-side keyword matching against `Marco.luau`'s `DrinkMenu`:
1. **Keyword & Name Detection**:
   - When a player speaks to Marco, `AV-AIEngine` scans the player's input string for drink names and keywords (`velvet`, `stardust`, `amber`, `sapphire`, `blue`, `crimson`, `ruby`, `gold`, `eclipse`, `emerald`, `mint`, `solar`, `orange`).
2. **Serving Gesture & Delivery**:
   - If a keyword matches, fires `AIServeGesture` for the counter hand-off and clones the corresponding `Tool` from `ReplicatedStorage.DrinkTemplates` into the player's `Backpack`.

---

## 4. Realistic AI Expectations & Edge Cases

Developers must keep in mind that NPCs use generative Large Language Models (LLMs), not deterministic game logic:
- **Dialogue vs. Dispensed Item Disconnect**:
  - The LLM generates natural dialogue independently of the server's keyword matcher.
  - *Example*: A player might ask for a "fruity refresher". Marco's LLM reply might enthusiastically offer a "chilled blueberry fizz", but the keyword matcher serves an orange *Solar Radiance* tool.
- **Improving Accuracy**:
  - The more detailed, explicit, and structured you make Marco's system prompt in `Marco.luau`, the better he will align his dialogue with the 6 specific menu items.
  - Guide Marco in his prompt to only recommend drinks that exist on his menu.


---

## 5. Drinking Mechanics & Animation Scope

Each drink tool contains a `DrinkScript` and interacts with `DrinkClientHandler`:
- **Glass Grip & Holding**: Positions the beverage glass in the character's hand with clean tool welding.
- **Simplistic Drinking Animation**: Includes a lightweight, basic arm-raise sipping animation and audio sound effect. There are no complex cinematic or motion-captured animations included out-of-the-box.
- **Easy Custom Animation Swapping**: Developers wanting custom mocap or unique R15 drinking animations can easily swap the Animation ID inside `DrinkClientHandler.client.luau`.
- **Sip Lifecycle**: Each drink allows **3 sips**. On the final sip, the tool cleans up automatically from the player's inventory.

