# Lowball Offers — Claude Code Skill

A Claude Code skill that automates real estate lowball offers. Scrapes property listings, calculates offers, and sends personalized emails to listing agents — all on autopilot.

## What It Does

1. **Scrapes** property listings from Realtor.com (any US city, any price range)
2. **Filters** for properties with agent contact info (email + phone)
3. **Calculates** your offer price (default 60% of asking — configurable)
4. **Lets you write** or customize the offer email
5. **Creates** an Instantly email campaign with personalized offers for each agent
6. **Uploads** all leads with property-specific data (address, asking price, your offer)
7. **Pauses** the campaign so you can review before anything sends

Each agent gets a unique email referencing their specific listing, the asking price, and your exact offer amount. One campaign handles all of them.

## Install

Copy `lowball-offers.md` into your Claude Code commands directory:

```bash
# Global (available in all projects)
cp lowball-offers.md ~/.claude/commands/lowball-offers.md

# Or project-level (available only in one project)
cp lowball-offers.md .claude/commands/lowball-offers.md
```

Then run it in Claude Code:

```
/lowball-offers
```

## Requirements

You need two MCP servers connected to Claude Code. The skill will walk you through setup if they're missing.

### Apify (for scraping listings)
- Create a free account at [apify.com](https://apify.com)
- Get your API token from Settings → API Tokens
- Add the MCP server to Claude Code settings:
  - Name: `apify`
  - Command: `npx -y @anthropic-ai/apify-mcp-server`
  - Env: `APIFY_TOKEN=<your token>`

### Instantly (for sending emails)
- Create an account at [instantly.ai](https://instantly.ai)
- Get your API key from Settings → API
- Add the MCP server to Claude Code settings:
  - Name: `instantly`
  - Command: `npx -y @anthropic-ai/instantly-mcp-server`
  - Env: `INSTANTLY_API_KEY=<your key>`
- Connect at least one sender email account in Instantly

## How It Works

The skill asks you for:
- **Location** — any US city (Houston TX, Phoenix AZ, Miami FL, etc.)
- **Price range** — min and max (e.g., $200k-$500k)
- **Offer %** — what percentage of asking price to offer (default 60%)
- **Days on market filter** — optionally target only stale listings
- **Your name and phone** — for the email signature and callback number

Then it scrapes listings, shows you the results, lets you customize the email copy, creates the campaign in Instantly, and loads all the leads. Campaign starts **paused** — nothing sends until you review and activate it manually.

## Cost

- **Scraping:** ~$0.0015 per listing via Apify. 1,000 properties = ~$1.50
- **Emails:** Covered by your Instantly subscription

## Example Output

```
LOWBALL OFFER CAMPAIGN READY

Campaign:      "Lowball Offers - Houston TX - 2026-03-23"
Status:        PAUSED (will not send until you activate)
Leads loaded:  47 agents
Emails:        2-step sequence (initial offer + 3-day follow-up)
Schedule:      Mon-Fri, 8am-5pm CT
Daily limit:   30 emails/day
Scraping cost: ~$0.07
```

## Follow

Built by [@tenfoldmarc](https://instagram.com/tenfoldmarc). Follow for more AI automation builds — real systems, not theory.
