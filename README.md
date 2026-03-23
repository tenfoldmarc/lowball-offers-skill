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

---

## Install (3 steps)

### Step 1: Download the skill file

Open Claude Code and paste this command:

```
! curl -o ~/.claude/commands/lowball-offers.md https://raw.githubusercontent.com/tenfoldmarc/lowball-offers-skill/main/lowball-offers.md
```

That's it — the skill is now installed. You can verify by typing `/lowball-offers` in Claude Code.

### Step 2: Connect Apify (scrapes the property listings)

1. Go to [apify.com](https://apify.com) and create a free account
2. Once logged in, click your profile icon (top right) → **Settings** → **API Tokens**
3. Click **+ Create Token**, give it any name, and copy the token
4. In Claude Code, type this:

```
/update-config add an MCP server called "apify" with command "npx -y @anthropic-ai/apify-mcp-server" and env var APIFY_TOKEN set to <paste your token here>
```

5. Restart Claude Code (close and reopen it)

### Step 3: Connect Instantly (sends the emails)

1. Go to [instantly.ai](https://instantly.ai) and create an account
2. Once logged in, go to **Settings** (gear icon) → **API** → copy your API key
3. **Important:** Make sure you have at least one email account connected in Instantly (Settings → Email Accounts). This is the email address your offers will be sent from.
4. In Claude Code, type this:

```
/update-config add an MCP server called "instantly" with command "npx -y @anthropic-ai/instantly-mcp-server" and env var INSTANTLY_API_KEY set to <paste your API key here>
```

5. Restart Claude Code one more time

### You're done! Run it:

```
/lowball-offers
```

The skill will check that everything is connected and walk you through the rest.

---

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

---

## Follow

Built by [@tenfoldmarc](https://instagram.com/tenfoldmarc). Follow for more AI automation builds — real systems, not theory.
