# jednoducho trackuj in your AI app

The trackuj plugin lets your AI app track your working time in [jednoducho trackuj](https://jednoduchotrackuj.sk) as you work. It brings two things:

- **trackuj's MCP server**, `https://app.jednoduchotrackuj.sk/api/mcp`, whose tools start and stop your Timers, add Manual Time Entries, and read your Clients, your work, the Summary and Reports;
- **four skills**, which turn what you say into the right tools:
  - **track** starts a Timer when you start on something ("let's work on Acme's checkout"), switches it, pauses it, and stops it when you're done;
  - **review-day** finds the gaps in today's work and fills each one you confirm;
  - **client-report** prepares last month's Client report of one Client, with the link where its PDF downloads and a draft cover email (for Owners);
  - **where-did-time-go** answers how much time went where, from the Summary.

The first time your AI app uses trackuj, it sends you to jednoducho trackuj in your browser: sign in as usual, and allow the connection. It then works as you, in each of your Organisations, with your Role there. You see it, and disconnect it, in the app under **Your account → Agents**.

## Install

Claude Code:

```
/plugin marketplace add monadial/trackuj-plugin
/plugin install trackuj@trackuj
```

Then open `/mcp`, choose trackuj, sign in to jednoducho trackuj, and allow the connection.

Claude, on the web or in the desktop app: under **Customize → Plugins**, add the marketplace `monadial/trackuj-plugin` and install trackuj.

Codex, and the ChatGPT desktop app:

```
codex plugin marketplace add monadial/trackuj-plugin
codex plugin add trackuj@trackuj
```

VS Code with Copilot: in the Command Palette, run **Chat: Install Plugin from Source**, and give it `https://github.com/monadial/trackuj-plugin`.

Another tool that reads Agent Skills: install a single skill from its folder, `skills/<name>`. It needs trackuj's MCP server connected in that tool too, and only the apps above can connect for now.

## This repo

It is published from jednoducho trackuj's own source, one way: a change made here is lost at the next publish.
