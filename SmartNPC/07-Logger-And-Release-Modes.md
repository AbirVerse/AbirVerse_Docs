# Logger, Dev Mode & Production Release Guide

LuxuryVerse includes **`AV-Logger`** (`ReplicatedStorage.Packages.AV-Logger`), a dual-environment logging utility that controls runtime diagnostics across both the server and client.

---

## 1. Logger Overview & Dev Mode Toggle

`AV-Logger` synchronizes the diagnostic mode across the server and client using the `IsDevMode` attribute on `ReplicatedStorage`.

```luau
-- Toggle DevMode programmatically or in AISettings:
AVLogger.SetDevMode(true)  -- Verbose diagnostics enabled
AVLogger.SetDevMode(false) -- Production Release mode (silent)
```

In `ServerScriptService.Configs.AISettings`:
```luau
local AISettings = {
    DevMode = false, -- Set to false for live production builds
    -- ...
}
```

---

## 2. Log Level Behaviors

| Method | Behavior in Dev Mode (`DevMode = true`) | Behavior in Production (`DevMode = false`) | Purpose |
| :--- | :--- | :--- | :--- |
| `AVLogger.Info(tag, msg)` | Prints `ℹ️ [Tag] Message` | **Silent (Suppressed)** | General operational updates |
| `AVLogger.Debug(tag, msg)` | Prints `🔍 [Tag] Message` | **Silent (Suppressed)** | Low-level payload & timing data |
| `AVLogger.Warn(tag, msg)` | Warns `⚠️ [Tag] Message` | **Silent (Suppressed)** | Non-breaking notices & retries |
| `AVLogger.Error(tag, msg)` | Warns `❌ [Tag] Message` | **Active (Outputs Warning)** | Critical errors and failures |

---

## 3. API Secret Key Redaction

`AV-Logger` and `AV-AIEngine` enforce key redaction:
- Secret tokens retrieved via `HttpService:GetSecret()` are never logged to console output.
- Only provider names (`Google`, `Groq`, etc.) and HTTP status codes (`200`, `429`, `500`) are printed during dev runs.
- In production (`DevMode = false`), console output is completely silent unless a critical unhandled engine error occurs.

---

## 4. Preparing for Release

Before publishing your game to the public:
1. Open `ServerScriptService.Configs.AISettings.luau`.
2. Ensure `DevMode = false`.
3. Verify that `HttpService` is enabled in Game Settings.
4. Verify your API secrets are registered under Creator Hub Credentials.

