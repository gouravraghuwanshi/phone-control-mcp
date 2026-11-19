# Privacy

Phone Control doesn't collect, store or send your data anywhere.

- It runs an MCP server on `127.0.0.1` (this phone only). Requests need the token shown in the app, and requests from web pages (with an `Origin` header) are refused.
- The app makes no outgoing network connections. No analytics, ads, crash reporting or accounts.
- It reads the screen and acts only when an agent you connected calls one of its tools. It doesn't record the screen in the background. The accessibility events it receives are used only to tell when the screen has settled.
- It keeps a short list of recent actions (up to 30, in memory) so you can see what agents did. It's gone when the app's process ends.
- It saves three small settings on the phone: the token, the port, and the pause switch.

What it reads (screen text, and screenshots when asked) is returned to the agent that asked. That agent may send it to its AI provider. Check the privacy terms of the agent and model you use.

Questions: open an issue at https://github.com/gouravraghuwanshi/phone-control-mcp/issues
