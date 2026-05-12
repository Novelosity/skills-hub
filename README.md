# Skills Hub

> One repo. Every skill. Your personal AI command center.

Skills Hub is a unified collection of AI skills for Claude Code, Windsurf, and other AI coding assistants. Instead of juggling scattered skill files, everything lives here — with a master concierge (`SKILL.md`) that figures out what you need and activates the right skill automatically.

---

## How It Works

Start by telling the concierge who you are and what you want to build. It asks 3 quick questions, then routes you to the right skill, shows you a personalized briefing, and executes — no manual skill switching required.

```
"Build a SaaS landing page"       → ui-ux-pro-max
"Fix my Meta ads ROAS"            → meta-ads-optimizer
"I want to sell digital products" → digital-product-market-analyzer
"Grow my blog traffic"            → backlink-analyzer
"Launch a Google Ads campaign"    → google-ads-campaign-builder
"Build a go-to-market strategy"   → marketing-strategy-builder
```

---

## Skills

| Skill | What It Does |
|---|---|
| `ui-ux-pro-max` | UI/UX design intelligence — 67 styles, 96 palettes, 57 font pairings, 25 chart types, 13 stacks (React, Next.js, Vue, Svelte, Tailwind, shadcn/ui, Flutter, SwiftUI + more) |
| `meta-ads-optimizer` | Audit and optimize Meta (Facebook/Instagram) ad campaigns — 10-dimension scoring, AI-generated copy rewrites, ROAS improvement |
| `google-ads-campaign-builder` | Build Google Ads campaigns from scratch — keyword strategy, ad copy, campaign structure |
| `digital-product-market-analyzer` | Find what digital products sell best right now — creation guides, platform strategy (Etsy, Gumroad, Payhip, KDP) |
| `backlink-analyzer` | SEO link gap analysis, outreach templates, domain authority building |
| `marketing-strategy-builder` | Full go-to-market strategy — positioning, channels, messaging, launch playbook |

---

## Installation

### Claude Code
```bash
uipro init --ai claude
```

### Windsurf
```bash
uipro init --ai windsurf
```

### Other assistants
```bash
uipro init --ai cursor      # Cursor
uipro init --ai copilot     # GitHub Copilot
uipro init --ai roocode     # Roo Code
uipro init --ai gemini      # Gemini
uipro init --ai all         # Install for everything
```

> Requires [uipro-cli](https://www.npmjs.com/package/uipro-cli): `npm install -g uipro-cli`

---

## Manual Setup

Clone this repo into your AI assistant's skills directory:

```bash
# Claude Code
git clone https://github.com/Novelosity/skills-hub.git ~/.claude/skills/skills-hub

# Windsurf
git clone https://github.com/Novelosity/skills-hub.git ~/.windsurf/skills/skills-hub
```

---

## Usage

Once installed, just talk to your AI assistant naturally:

- *"Build me a landing page for a fitness SaaS"*
- *"Analyze my Meta ads and tell me what's killing my ROAS"*
- *"What digital products should I sell in 2026?"*
- *"Create a backlink strategy for my blog"*

The `SKILL.md` concierge will handle routing, briefing, and execution.

---

## Tech Stack Support

React · Next.js · Vue · Nuxt · Svelte · SvelteKit · HTML + Tailwind · shadcn/ui · React Native · Flutter · SwiftUI · Jetpack Compose · Astro

---

Built with [uipro-cli](https://www.npmjs.com/package/uipro-cli) + [Claude Code](https://claude.ai/code)
