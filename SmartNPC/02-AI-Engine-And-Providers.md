# AI Engine & Multi-Provider Integration Guide

The core AI conversation system is managed by **`AV-AIEngine`** (`ServerScriptService.Packages.AV-AIEngine`). It connects your Roblox NPCs to industry-standard AI models with enterprise failover resilience.

---

## 1. Provider Tier Breakdown (Free vs. Paid)

LuxuryVerse supports 6 AI providers out-of-the-box. You can mix and match free tiers and paid models in `ServerScriptService.Configs.AISettings`.

| Provider | Cost / Tier | Recommended Models | Description & Best Use Case |
| :--- | :--- | :--- | :--- |
| **Google AI Studio** | **Generous Free Tier** (15 RPM free) | `gemini-3.1-flash-lite`, `gemini-3.5-flash-lite`, `gemini-3.6-flash` | Best overall choice for free deployment. Extremely fast response latency with zero cost for standard traffic. |
| **Groq Cloud** | **Free Tier Available** | `openai/gpt-oss-120b`, `llama-3.3-70b-versatile` | Ultra-fast LPU inference (often sub-100ms). Perfect for rapid conversational dialogue. |
| **OpenRouter** | **Free & Paid Routes** | `gryphe/mythomax-l2-13b`, `meta-llama/llama-3-8b-instruct:free` | Aggregator provider that hosts multiple free community endpoints as well as premium pay-as-you-go models. |
| **DeepSeek** | **Paid (Extremely Low Cost)** | `deepseek-chat`, `deepseek-flash`, `deepseek-v4-pro` | Extremely cheap tokens (fractions of a cent per 1,000 requests) with strong roleplay depth. |
| **OpenAI** | **Paid (Credits Required)** | `gpt-4.1-nano`, `gpt-4o-mini` | High reliability and consistency. Requires funded prepaid balance. |
| **Anthropic** | **Paid (Credits Required)** | `claude-haiku-5-5`, `claude-3-5-haiku-latest` | Exceptional dialogue nuance and style mimicry. Requires funded account. |

---

## 2. Setting Up API Secrets in Roblox Creator Hub

Never hardcode raw API keys into Luau scripts. LuxuryVerse uses Roblox's native **Secret Management Service** (`HttpService:GetSecret`):

1. Open your game in the [Roblox Creator Dashboard](https://create.roblox.com/dashboard/creations).
2. Navigate to **Game Settings** > **Credentials / Secrets**.
3. Create new secrets matching the names specified in `AISettings.luau`:
   - `GoogleAPI1` (Your Google AI Studio Gemini API Key)
   - `GoogleAPI2` (Optional secondary key for pool rotation)
   - `GroqAPI1` (Your Groq Cloud API Key)
   - `DeepSeekAPI1` (Your DeepSeek API Key)
   - `OpenRouterAPI1` (Your OpenRouter API Key)
   - `OpenAI1` (Your OpenAI API Key)
   - `AnthropicAPI1` (Your Anthropic API Key)
4. Enable **Allow HTTP Requests** under **Game Settings** > **Security** in Roblox Studio.

---

## 3. Failover Engine & The Permutational Sequence

In `ServerScriptService.Configs.AISettings`, the failover sequence is defined by:
```luau
local AISettings = {
    ActiveProvider = { 
        "Google", 
        "Groq", 
        "DeepSeek", 
        "OpenRouter", 
        "OpenAI", 
        "Anthropic", 
    },
    SequencePriority = { "Provider", "Model", "API" },
    MaxRetries = 8,
    RequestTimeout = 30,
    MaxTokens = 150,
    Temperature = 0.75,
}
```

### How Failover Works:
1. `AV-AIEngine` attempts the first active provider (e.g., `Google`) with its primary model.
2. If Google returns HTTP 429 (rate-limit) or HTTP 402 (insufficient credits), it automatically switches to the secondary key or next model.
3. If all Google models fail, it instantly jumps to `Groq`, then `DeepSeek`, and so forth.
4. **All retries maintain the player's conversation history seamlessly**.

---

## 4. Automatic Zero-Credit Offline Mode

What happens if all your API providers run out of credits, hit rate limits, or Roblox loses internet connectivity to external CDNs?

**The game never breaks or throws error dialogues to players.**

1. When all providers fail or `MaxRetries` is reached, `AV-AIEngine` automatically engages **Offline Standalone Mode**.
2. The NPC selects an appropriate response from its `FallbackReplies` array defined in its NPC config module:
   ```luau
   FallbackReplies = {
       "Coming right up! Enjoy your drink at the lounge.",
       "Freshly poured for you! Always good having you at the bar.",
       "Classic choice! Let me know if you need anything else.",
       "Here you go! Best luxury refreshment in the house.",
   }
   ```
3. Game mechanics (such as pouring drinks or reacting to outfits) continue to execute normally via local Luau heuristics.
