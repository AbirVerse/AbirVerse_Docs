# Boutique, Wardrobe & Mannequin Architecture

LuxuryVerse features a complete fashion and wardrobe system that combines physical interactive mannequins, real-time avatar customization, and in-game UGC catalog checkout.

---

## 1. File Locations & Security Architecture

To protect commercial assets and prevent exploiters from tampering with catalog prices or stealing backend configurations, the system is strictly split between the server and the client:

| Script / Asset | Location | Security & Purpose |
| :--- | :--- | :--- |
| **`AV-OutfitEngine`** | `ServerScriptService.Packages.AV-OutfitEngine` | **Server-Only**. Manages avatar descriptions, purchase prompts, 3D face adjustments, and item price verification. Exploiters cannot access this script. |
| **`BoutiqueSettings`** | `ServerScriptService.Configs.BoutiqueSettings` | **Server-Only**. Rotation intervals and default body bundle IDs. |
| **`Outfits`** | `ServerScriptService.Configs.Outfits` | **Server-Only**. Curated outfit lists, asset IDs, and catalog definitions. |
| **`Mannequins`** | `ServerScriptService.Mannequins` | **Server-Only**. Spawns mannequin rigs, listens for click-to-wear, and cycles looks. |
| **`BoutiqueClient`** | `StarterPlayer.StarterPlayerScripts.BoutiqueClient` | **Client-Side**. Renders the boutique catalog GUI, handles category tabs, and sends try-on requests to the server. |

> **Security Rule**: Sensitive configurations, pricing caches, and outfit definitions reside in `ServerScriptService`—**never** in `ReplicatedStorage`. The client only receives sanitized display data via `GetBoutiqueCatalog` RemoteFunction.

---

## 2. Boutique Catalog Tabs

The client interface (`BoutiqueCatalogGui`) provides 6 tabs:

1. **`👗 Outfits`**: Full curated looks. Shows dynamic real-time look cost in `R$` and allows one-click **Wear Full Look** or purchase.
2. **`💎 Items`**: Individual designer accessories, hair, tops, and bottoms with live pricing and purchase prompts.
3. **`✨ My Current Look`**: Shows all items equipped on the player's avatar (clothes, UGC accessories, and body parts) with individual `🗑️ Remove` buttons and dynamic `Total Look: X R$` calculation.
4. **`👤 MyAvatar`**: Isolates the avatar's body parts and features (Hair, Head, Face, Torso, Left/Right Arm, Left/Right Leg) without any clothing clutter. Each item features a full-width `🗑️ Remove` button.
5. **`🔍 Custom`**: Allows players to paste any public Roblox UGC Asset ID or catalog URL to preview, try on, or buy directly in-game.
6. **`🕒 Recent`**: History of all items and looks tried on during the current play session with quick re-equip buttons.

---

## 3. Interactive Mannequins

Located at `workspace.LuxuryHotelLobby.BoutiqueArea.MannequinStands`:
- **Periodic Rotation**: Mannequins rotate through outfits every `RotationInterval` seconds (configured in `BoutiqueSettings.luau`).
- **Direct Click-to-Wear**: Clicking a mannequin stand or the mannequin model directly equips the full outfit on the player's avatar instantly.
- **Stylist AI Event**: Trying on an outfit automatically fires a reaction event to the Stylist NPC (Grace), who compliments the player's drip.

---

## 4. 3D Face Detection & Head Hiding

Modern Roblox outfits frequently incorporate custom 3D makeup masks, faceless heads, or **Dynamic Heads** (`AssetTypeId == 79`). If worn on a default avatar, the player's original head and default face decal would clip through and ruin the aesthetic.

### How LuxuryVerse Handles 3D Faces:
1. When applying an outfit, `AV-OutfitEngine` analyzes the asset types and metadata names for keywords (`3d face`, `faceless`, `mask`, `head`, `DynamicHead`).
2. If a 3D face replacement is detected:
   - Sets the player's physical `Head.Transparency = 1`.
   - Sets all child decals and textures on the head to `Transparency = 1`.
3. If an outfit does not have a 3D face, the player's own head, face decals, and face accessories remain visible.
4. When the player clicks **`🔄 Reset Avatar`**, head and decal transparency is restored to `0`.

---

## 5. Avatar Reset (`🔄 Reset Avatar`)

When players click **`🔄 Reset Avatar`** at the bottom of the boutique interface:
- Restores all original body parts (Head, Torso, Arms, Legs).
- Re-enables head and face decal visibility (`Transparency = 0`).
- Restores all original clothing, layered clothing, accessories, hair, skin tones, and scales.
- Preserves the player's canonical appearance as loaded when they first joined the server.

---

## 6. Server-Side Price Caching & Marketplace Throttling Protection

To prevent hitting Roblox `MarketplaceService:GetProductInfo()` rate limits (HTTP 429 throttling) while providing instantaneous UI responsiveness, `AV-OutfitEngine` maintains an in-memory server cache (`itemPriceCache`):

### How Caching Works:
1. **Background Startup Pre-Fetch**:
   - On server init, `AV-OutfitEngine` pre-fetches prices for all configured Outfits and Individual Items in a staggered background loop (`task.wait(0.04)`).
   - Prevents request bursts and server throttling.
2. **Instant UI Delivery**:
   - When a player opens the boutique, `GetBoutiqueCatalog` returns the cached price map directly from memory. Zero delay, zero external requests.
3. **Live Refresh on Purchase**:
   - When a player clicks to purchase an item (`HandlePurchase`), the engine queries `MarketplaceService:GetProductInfo()` on the spot to ensure the prompt reflects current live Robux pricing.
   - Automatically updates `itemPriceCache` with any changes.

### Edge Cases (Rare):
- **Mid-Session Price Change**: If a UGC creator updates their item price or removes it from sale while a server is running, the boutique displays the cached price until a player initiates a purchase, at which point the live prompt confirms current Roblox data.
- **MarketplaceService Outages / Rate Limits**: If Roblox marketplace servers experience downtime, `pcall` guards prevent errors, falling back safely to cached prices.
- **Deleted/Moderated Assets**: If a UGC asset is removed from Roblox, `GetProductInfo` safely fails and marks the item as off-sale without breaking the boutique.

