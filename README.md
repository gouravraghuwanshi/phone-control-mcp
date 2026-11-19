# Phone Control

An MCP server that lets AI agents on your Android phone read the screen and tap, type and swipe in your apps.

It's a small app with an accessibility service inside. Turn it on, paste a config into your agent, and the agent can do things like "open Maps and find the nearest pharmacy" or "reply to the last message in Signal" by actually using the apps, one step at a time. You stay in charge: there's a Stop button on screen the whole time, and it asks you before anything that sends, pays, deletes or installs.

## Who it's for

People who already run an AI agent on their phone (in Termux, or an app like [Saturn Agent](https://github.com/gouravraghuwanshi/saturn-agent)) and want it to reach beyond the terminal. If your agent speaks MCP over Streamable HTTP, it can use this.

It's not a remote-control tool. The server only listens on `127.0.0.1`, so only apps on the same phone can talk to it.

## Tools

| Tool | What it does |
| --- | --- |
| `phone_screen` | Reads what's on screen: the app in front and a numbered list of elements (buttons, fields, text, lists) with positions. |
| `phone_tap` | Taps or long-presses an element by number, by its text, or at x,y. Returns the new screen. |
| `phone_type` | Sets the text of a field (by number, or the focused one). Refuses password fields. |
| `phone_enter` | Presses the keyboard's enter/search/send key in the focused field. Asks you first unless it's a search or address box. |
| `phone_swipe` | Swipes up/down/left/right, or between exact coordinates. |
| `phone_scroll` | Scrolls a list forward or back. |
| `phone_key` | Presses back, home, recents, or opens notifications or quick settings. |
| `phone_open_app` | Opens an installed app by name or package (`"Maps"`, `"com.whatsapp"`). |
| `phone_list_apps` | Lists the apps that can be opened, with their packages. |
| `phone_wait` | Waits (100 ms to 15 s) for the screen to change, then returns it. |
| `phone_screenshot` | Takes a half-size JPEG screenshot, for when the element list isn't enough. |

Every action returns the screen as it looks afterwards, so the agent can check what happened before doing the next thing.

## Safety rules

These are built in. No client can turn them off.

- **Stop button.** While an agent is acting, a small pill floats at the top of the screen: "An agent is using your phone · Stop". Tap Stop and every tool call fails until you open Phone Control and tap Resume.
- **It asks before big taps.** Taps on things labelled Send, Pay, Buy, Delete, Install, Transfer, Confirm, Post, Share, Sign out and similar show an Allow / Don't allow card. If you don't answer in 45 seconds, the answer is no. Enter in a normal text field (which might send a message) asks too.
- **No passwords.** It won't type into password fields, and password text shows up as `••••` when the screen is read.
- **Not on the lock screen.** If the phone is locked, nothing works.
- **Hands off Android's own prompts.** Permission dialogs and the package installer are off limits. Only you answer those.
- **Doesn't control itself.** It won't act while the Phone Control app is on screen, so an agent can't flip its own switches.
- **Pause switch.** "Let agents use the phone" in the app turns everything off without touching Accessibility settings.
- **Recent actions.** The app shows the last actions agents took, with times, and which ones failed.

It also never acts on its own. It only does something when an agent calls a tool.

## Privacy

- The MCP server listens on `127.0.0.1` only. Nothing outside the phone can reach it.
- Every request needs the Bearer token shown in the app. You can make a new one any time (old configs stop working).
- Requests that carry an `Origin` header are refused, so web pages in your browser can't talk to it.
- The app makes no outgoing connections. No analytics, no crash reporting, no accounts. The `INTERNET` permission is only there to open the local socket.

One thing to be clear about: what Phone Control reads (screen text, screenshots) goes to the agent that asked for it. That agent then sends it to whichever AI provider it uses. So the privacy of your screen is really up to the agent and model you pick.

See [PRIVACY.md](PRIVACY.md) for the short version.

## Install

1. Download `phone-control-0.1.0.apk` from [Releases](https://github.com/gouravraghuwanshi/phone-control-mcp/releases) and install it. Android 11 or newer.
2. Open Android Settings → Accessibility → (Installed apps) → **Phone Control (MCP)** and turn it on. The app has a button that takes you there.
3. On Android 13 and newer the switch may be greyed out with a "Restricted setting" note. That's normal for apps installed from a file. Go to App info → ⋮ (top right) → **Allow restricted settings**, then try again.

When it's on, the app says "Ready for agents" and shows the address it's listening on.

## Connect an agent

In the app, tap **Copy config**. You get something like this:

```json
{
  "mcpServers": {
    "phone": {
      "url": "http://127.0.0.1:47811/mcp",
      "headers": { "Authorization": "Bearer <token>" }
    }
  }
}
```

(The copied version also has a short `description` field. If port 47811 is taken, the app picks another one and the config shows it.)

**pi / Saturn Agent:** put it in `~/.pi/agent/mcp.json` (merge it with any servers already there), then run `/reload`.

**Anything else:** it works with any MCP client that supports the Streamable HTTP transport. The server answers JSON on `POST /mcp`. It doesn't offer the optional SSE stream (`GET` returns 405), which the spec allows.

A good first prompt: "Use the phone tools to open Settings and tell me which Android version I'm on."

## Why it's not on Google Play

Google Play's Accessibility API policy (updated January 2026) doesn't allow apps that use the API to autonomously plan and execute actions. That's exactly what this is for, so it lives here instead. The APK on the Releases page is the real thing, signed, built from this repo.

## Build from source

You need JDK 17 and the Android SDK (compile SDK 37).

```sh
git clone https://github.com/gouravraghuwanshi/phone-control-mcp
cd phone-control-mcp
export ANDROID_HOME=$HOME/Android/Sdk   # wherever your SDK lives
./gradlew :app:assembleRelease
```

The APK lands in `app/build/outputs/apk/release/`. Without a signing config it's unsigned, so either sign it yourself or build `:app:assembleDebug` to try it out. To sign release builds, put a `keystore.properties` (with `storeFile`, `storePassword`, `keyAlias`, `keyPassword`) in the project root. It's in `.gitignore`.

The code is small. Four Kotlin files do the work:

- `McpServer.kt`: the HTTP + JSON-RPC server, token and Origin checks.
- `PhoneControl.kt`: the tools and the guard rails.
- `PhoneControlService.kt`: the accessibility service, the Stop pill and the Allow card.
- `MainActivity.kt`: the app screen.

## Companion: Saturn Agent

[Saturn Agent](https://github.com/gouravraghuwanshi/saturn-agent) runs the pi coding agent as a standalone Android app. Point it at Phone Control and it can use your other apps, not just its own terminal. They're separate on purpose: any MCP client can use Phone Control, and Saturn stays free of the Accessibility API.

## License

Apache-2.0. See [LICENSE](LICENSE). The bundled Inter and JetBrains Mono fonts are under the SIL Open Font License 1.1, see [NOTICE-fonts.txt](NOTICE-fonts.txt).
