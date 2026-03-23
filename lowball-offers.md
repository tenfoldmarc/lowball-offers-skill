---
description: "Scrape real estate listings, calculate lowball offers, and send personalized emails to listing agents via Instantly. Use when someone says 'lowball offers', 'real estate offers', 'property offers', or wants to automate outreach to real estate agents with cash offers."
---

# Lowball Offer Automation

Scrapes US property listings from Realtor.com, calculates a lowball offer for each property, and sends personalized emails to every listing agent via Instantly. One campaign, all agents — each email is personalized with the specific property, asking price, and offer amount.

---

## Step 0: Check Required Tools

Before doing anything else, verify the user has the required MCP servers connected.

### Check Apify
Try calling `mcp__apify__search-actors` with keywords "test" and limit 1. If it works, Apify is connected. If the tool doesn't exist or errors:

Tell the user:
```
You need to connect Apify to use this skill. Here's how:

1. Go to https://apify.com and create a free account
2. Go to Settings → API Tokens and copy your token
3. Install the Apify MCP server. Add this to your Claude Code settings
   (Settings → MCP Servers → Add):

   Name: apify
   Type: npx
   Command: npx -y @anthropic-ai/apify-mcp-server
   Env: APIFY_TOKEN=<your token>

4. Restart Claude Code and run /lowball-offers again
```
Stop here if Apify is not connected.

### Check Instantly
Try calling `mcp__instantly__list_campaigns` with limit 1. If it works, Instantly is connected. If the tool doesn't exist or errors:

Tell the user:
```
You need to connect Instantly to use this skill. Here's how:

1. Go to https://instantly.ai and create an account
2. Go to Settings → API → copy your API key
3. Install the Instantly MCP server. Add this to your Claude Code settings
   (Settings → MCP Servers → Add):

   Name: instantly
   Type: npx
   Command: npx -y @anthropic-ai/instantly-mcp-server
   Env: INSTANTLY_API_KEY=<your key>

4. Make sure you have at least one sender email account connected in Instantly
5. Restart Claude Code and run /lowball-offers again
```
Stop here if Instantly is not connected.

If both are connected, tell the user and proceed to Step 1.

---

## Step 1: Collect Parameters

Ask the user ALL of the following in a single message. Use defaults in brackets if they don't specify:

1. **Location** — US city and state (e.g., "Houston, TX", "Phoenix, AZ", "Miami, FL")
2. **Price range** — min and max (e.g., "$200k-$500k")
3. **Number of properties** — how many to scrape [default: 50]
4. **Offer percentage** — what % of asking price to offer [default: 60%]
5. **Minimum days on market** — only target stale listings? [default: 0 = all listings]
6. **Buyer name** — name to sign the emails with
7. **Buyer phone number** — phone number for agents to call back

**Do NOT ask about email content yet — that comes in Step 3.**

After they answer, confirm with a summary table:

```
Location:         Houston, TX
Price range:      $200,000 - $500,000
Properties:       50
Offer:            60% of asking
Min days listed:  30
Buyer:            John Smith | (555) 123-4567
```

Wait for confirmation before proceeding.

---

## Step 2: Scrape Properties

Construct the Realtor.com search URL:

```
https://www.realtor.com/realestateandhomes-search/{City}_{State}/price-{min}-{max}
```

Format rules:
- Replace spaces in city names with hyphens for multi-word cities: "San Antonio" → "San-Antonio"
- State is always 2-letter code: TX, AZ, FL, CA, etc.
- Prices are raw numbers, no commas: 200000, 500000

Examples:
- `https://www.realtor.com/realestateandhomes-search/Houston_TX/price-200000-500000`
- `https://www.realtor.com/realestateandhomes-search/San-Antonio_TX/price-150000-400000`
- `https://www.realtor.com/realestateandhomes-search/Miami_FL/price-400000-800000`

Call `mcp__apify__call-actor`:
```json
{
  "actor": "memo23/realtor-search-cheerio",
  "input": {
    "startUrls": [{"url": "<constructed URL>"}],
    "maxItems": <number of properties>
  },
  "callOptions": {"timeout": 300}
}
```

After the run completes, fetch results using `mcp__apify__get-actor-output` with these fields:
```
address_line,address_city,address_state_code,address_postal_code,list_price,beds,baths,sqft,days_on_market,mls_agent_name,mls_agent_email,mls_agent_phone,primary_office_name,href
```

For large result sets (100+), fetch in batches of 100 using offset/limit pagination.

Tell the user how many properties were found while you process them.

---

## Step 3: Filter, Calculate, and Design the Email

### 3a: Filter and Calculate

Filter the scraped listings:
1. **Must have agent email** — skip any listing where `mls_agent_email` is empty, null, or less than 4 characters
2. **Days on market filter** — if user specified a minimum, only keep listings at or above that threshold
3. **Deduplicate by agent email** — if the same agent email appears multiple times, keep only the listing with the highest days on market (most motivated seller)

For each remaining listing, calculate:
- **Offer price** = `list_price × offer_percentage` (rounded to nearest dollar)
- **Full address** = `address_line, address_city, address_state_code address_postal_code`

Show the user a summary table (first 10 rows if more than 10, with a count of total):

```
Found 47 properties with agent emails (from 82 total scraped)

| # | Property Address          | Asking    | Offer (60%) | Agent            | Days |
|---|--------------------------|-----------|-------------|------------------|------|
| 1 | 2308 Bastrop St, Houston | $390,000  | $234,000    | Kerry Ann C.     | 108  |
| 2 | 10622 Alderford Ct...    | $360,000  | $216,000    | Tammy Rea        | 19   |
...showing 10 of 47
```

### 3b: Design the Email

Now ask the user what they want the email to say. Present it like this:

```
Now let's design your email. Here's a proven template that works well for lowball offers:

SUBJECT: Cash offer on [property address]

---
Hi [agent first name],

I came across the listing at [property address] currently listed at [asking price]
and I'd like to make an offer.

I'm a cash buyer looking to close quickly. I'm prepared to offer [offer price]
with a 14-day close, no contingencies, no financing delays.

I understand this is below asking, but I'm serious and ready to move immediately.
If your seller is motivated for a fast, clean close, this could be a win-win.

If you'd like to discuss, please call me directly at [buyer phone].
I'm available anytime.

Best regards,
[buyer name]
---

FOLLOW-UP (sent 3 days later if no reply):

SUBJECT: Following up - [property address]

---
Hi [agent first name],

Just following up on my cash offer for [property address].

Offer: [offer price]
Close: 14 days
No contingencies, no financing delays.

If the property is still available and your seller wants a quick, hassle-free close,
I'm ready to move. Give me a call at [buyer phone].

Thanks,
[buyer name]
---

Want to use this template as-is, or would you like to change anything?
You can:
- Rewrite the whole email in your own words
- Adjust the tone (more formal, more casual, more aggressive)
- Change the closing terms (settlement period, conditions, etc.)
- Add or remove the follow-up email
- Change the follow-up delay (currently 3 days)

The personalization variables ([property address], [offer price], etc.)
will be automatically filled in for each agent.
```

Wait for the user to either approve the default or provide their own version.

If they provide custom copy, convert it to HTML for Instantly (wrap paragraphs in `<p>` tags, use `<b>` for emphasis, `<br>` for line breaks). Make sure all personalization variables use Instantly's `{{variable}}` syntax:
- `{{firstName}}` — agent's first name
- `{{propertyAddress}}` — full property address
- `{{askingPrice}}` — formatted asking price (e.g., "$390,000")
- `{{offerPrice}}` — calculated offer price (e.g., "$234,000")
- `{{buyerPhone}}` — buyer's phone number
- `{{buyerName}}` — buyer's name

---

## Step 4: Create Instantly Campaign

### 4a: Discover sender accounts

Create the campaign using `mcp__instantly__create_campaign` with the user's approved email copy:

```json
{
  "name": "Lowball Offers - {Location} - {today's date}",
  "subject": "<user's subject line with {{variables}}>",
  "body": "<user's email body as HTML with {{variables}}>",
  "daily_limit": 30,
  "email_gap": 10,
  "stop_on_reply": true,
  "stop_on_auto_reply": true,
  "track_opens": false,
  "track_clicks": false,
  "timing_from": "08:00",
  "timing_to": "17:00",
  "timezone": "America/Chicago",
  "sequence_steps": 2,
  "step_delay_days": 3,
  "sequence_subjects": ["<subject 1>", "<subject 2>"],
  "sequence_bodies": ["<body 1 HTML>", "<body 2 HTML>"]
}
```

If the user opted for no follow-up, use `"sequence_steps": 1` and omit `sequence_subjects`/`sequence_bodies`.

### 4b: Select sender account

The first call returns eligible sender accounts. Show the user a numbered list:

```
Which email do you want to send from?

1. john@smithinvestments.com
2. john@cashoffer.com
3. john@realestatecash.com
```

If there's only one account, confirm it. Then call `mcp__instantly__create_campaign` again with the same parameters plus `"email_list": ["<chosen email>"]`.

Save the returned campaign ID for Step 5.

---

## Step 5: Upload Leads to Campaign

Use `mcp__instantly__add_leads_to_campaign_or_list_bulk` to upload all leads in one batch (up to 1,000 per call). For more than 1,000 leads, batch into groups of 1,000.

For each filtered property, create a lead object:

```json
{
  "email": "<mls_agent_email>",
  "first_name": "<first name parsed from mls_agent_name>",
  "last_name": "<remaining name parts from mls_agent_name>",
  "company_name": "<primary_office_name>",
  "phone": "<mls_agent_phone>",
  "custom_variables": {
    "propertyAddress": "<full formatted address>",
    "askingPrice": "$<list_price with commas>",
    "offerPrice": "$<calculated offer with commas>",
    "buyerPhone": "<buyer phone from Step 1>",
    "buyerName": "<buyer name from Step 1>"
  }
}
```

Call with:
```json
{
  "campaign_id": "<campaign ID from Step 4>",
  "skip_if_in_campaign": true,
  "leads": [<array of lead objects>]
}
```

---

## Step 6: Final Summary

After uploading, show the user:

```
LOWBALL OFFER CAMPAIGN READY

Campaign:      "Lowball Offers - Houston TX - 2026-03-23"
Status:        PAUSED (will not send until you activate)
Leads loaded:  47 agents
Emails:        2-step sequence (initial offer + 3-day follow-up)
Sending from:  john@smithinvestments.com
Schedule:      Mon-Fri, 8am-5pm CT
Daily limit:   30 emails/day
Scraping cost: ~$0.07 (47 listings × $0.0015)

NEXT STEPS:
1. Log into Instantly and review the campaign
2. Check that the email preview looks right
3. Activate the campaign when you're ready to start sending
4. Replies automatically stop the sequence for that agent

The campaign will NOT send anything until you manually activate it.
```

**IMPORTANT: The campaign is ALWAYS created in PAUSED state. NEVER activate a campaign automatically. The user must review and activate manually in Instantly.**

---

## Error Handling

- **Apify not connected:** Show setup instructions (Step 0) and stop
- **Instantly not connected:** Show setup instructions (Step 0) and stop
- **No listings found:** Suggest broadening the price range, trying a nearby city, or checking the URL format
- **No agent emails in results:** Report how many listings were found vs. how many had emails. Suggest trying a different price range — higher-end listings tend to have more agent email data
- **Instantly has no sender accounts:** Tell user they need to connect at least one email account in Instantly (Settings → Email Accounts) before they can create a campaign
- **Apify run timeout:** Suggest reducing maxItems or retrying. The actor can handle up to ~1,000 listings in 5 minutes
- **More than 1,000 leads:** Batch the upload into groups of 1,000 using multiple `add_leads_to_campaign_or_list_bulk` calls

---

## Reference: Available Personalization Variables

These variables can be used in subject lines and email bodies. Instantly replaces them with each lead's actual data when sending:

| Variable | What it contains | Example |
|----------|-----------------|---------|
| `{{firstName}}` | Agent's first name | Kerry Ann |
| `{{lastName}}` | Agent's last name | Carrington |
| `{{companyName}}` | Agent's brokerage | Coldwell Banker Realty |
| `{{email}}` | Agent's email | kerry@coldwell.com |
| `{{phone}}` | Agent's phone | 713-371-2553 |
| `{{propertyAddress}}` | Full property address | 2308 Bastrop St, Houston, TX 77004 |
| `{{askingPrice}}` | Formatted asking price | $390,000 |
| `{{offerPrice}}` | Calculated offer price | $234,000 |
| `{{buyerPhone}}` | Buyer's callback number | (555) 123-4567 |
| `{{buyerName}}` | Buyer's name | John Smith |

---

## Credits

Built by [@tenfoldmarc](https://instagram.com/tenfoldmarc). Follow for more AI automation builds like this — real systems, not theory.
