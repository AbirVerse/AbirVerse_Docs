# Roblox Policy, Guidelines & 18+ Age Rating

## 1. Why Experiences with Smart AI NPCs Require an 18+ Rating

When publishing an experience that utilizes external Generative AI (LLMs) to generate dynamic dialogue for in-game characters, Roblox's Experience Guidelines Questionnaire will classify the experience as **18+ (Restricted)** or **17+**.

### The Experience Questionnaire
During the Creator Hub questionnaire process:
1. You will be asked: *"Does your experience contain user-facing generative AI that produces dynamic content?"*
2. Answering **Yes** triggers platform requirements because external AI APIs generate text that is generated at runtime rather than pre-scripted.
3. Roblox requires developers to acknowledge that players interacting with non-static language models must meet the age threshold set by platform compliance standards.

---

## 2. Roblox Terms of Service & Content Prohibitions

Even with an 18+ or 17+ age recommendation, **experiences must strictly adhere to the Roblox Community Standards**. The 18+ classification does **NOT** permit illegal, regulated, or policy-violating substances.

### Strictly Prohibited Topics on Roblox:
- **Alcohol, Tobacco, and Drugs**:
  - Roblox states: *"Roblox does not permit illegal or regulated content in any form. This can include alcohol, drugs, tobacco products, or weapon making."*
  - **No alcohol words**: Names such as *Champagne, Wine, Beer, Whiskey, Vodka, Martini, Spritz, Cocktail, Single Malt*, or slang like *"cold one"* are banned from NPC dialogue, promotional thumbnails, icons, and asset names.
  - All bar assets and dialogue in LuxuryVerse are designed around **100% non-alcoholic luxury refreshments** (*Golden Eclipse, Midnight Sapphire, Velvet Stardust, Crimson Mirage, Emerald Frost, Solar Radiance*).
- **Romantic & Inappropriate Roleplay**:
  - NPCs must remain professional and civil. While playful banter is configured in Marco's personality prompt, prompt guardrails strictly forbid explicit adult interactions or harassment.
- **Real-World Currency Transactions (Off-Platform)**:
  - All monetization in the Boutique must route through standard Roblox `MarketplaceService` (Robux purchases). External payment links or third-party checkouts are forbidden.

---

## 3. Text Moderation & The `CommonFiltering` Setting

Roblox requires that all dynamic text and external API responses displayed to players comply with Roblox Community Standards. LuxuryVerse provides a centralized **`CommonFiltering`** configuration in `ServerScriptService.Configs.AISettings.luau` that automatically applies across all NPCs:

```luau
-- ServerScriptService.Configs.AISettings.luau
CommonFiltering = {
    Enabled = true,                -- Master toggle for dialogue filtering
    UseRobloxTextService = true,   -- Runs TextService:FilterStringAsync in live servers
    FilterPlayerInput = true,      -- Moderates player messages before AI ingestion
    FilterAIOutput = true,         -- Moderates AI replies before display
    CustomBlockedWords = {         -- Developer-defined blacklist
        "discord.gg",
        ".com",
        ".net",
    },
    FilteredFallbackMessage = "I apologize, but I cannot discuss that topic.",
}
```

### Developer Responsibility & Custom Moderation:
- **Customize Your Filters**: Use `CommonFiltering` to configure your own custom keywords, domain blacklists, and fallback replies tailored to your experience.
- **Ensure Roblox TOS Compliance**: While LuxuryVerse provides this built-in filtering foundation and `TextService` hook, you as the game developer are solely responsible for ensuring your final production deployment, custom prompts, and player communications adhere strictly to Roblox's Terms of Service and Community Standards.
- **Studio vs. Live Servers**: Roblox `TextService:FilterStringAsync` is bypassed during local Studio testing (`RunService:IsStudio()`) to avoid test-ID errors, but automatically engages in live production servers. Always test your published experience in a live test server to verify moderation behavior.

