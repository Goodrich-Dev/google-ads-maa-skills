---
name: google-ads-copy-optimizer
description: >
  Generate specific RSA headline/description recommendations and extension/asset
  improvements for Google Ads campaigns based on MAA findings, Quality Score component
  data, and search term intelligence. Trigger when users want to improve ad copy,
  fix Quality Score issues through copy changes, write new RSA headlines or descriptions,
  improve ad strength ratings, optimize sitelinks/callouts/structured snippets, or
  act on MAA ad copy recommendations. Also trigger on "write headlines for [campaign]",
  "improve my ad copy", "fix ad relevance", "RSA recommendations", "what should my
  headlines say", "ad strength is Poor/Average", "rewrite these ads", "extension
  suggestions", or any request to produce specific Google Ads copy based on performance
  data. This is the copywriting skill — it produces the actual text. Use
  google-ads-analyzer for diagnosis and google-ads-script for automation scripts.
---

# Google Ads Copy Optimizer

## What This Skill Does

This skill produces ready-to-paste RSA headlines, descriptions, and extension text
for Google Ads campaigns. It bridges the gap between diagnosis ("your ad relevance
is Below Average") and implementation ("here are 15 headlines to test").

Every recommendation connects back to data — search terms people actually used,
Quality Score components that need improvement, or competitive gaps the current
copy doesn't address. This isn't generic copywriting advice. It's specific text
tied to what the account data shows.

**Where this fits in the pipeline:** The google-ads-analyzer (MAA) diagnoses the
account and identifies what needs fixing. This skill takes those findings —
especially QS component breakdowns, search term data, and ad strength ratings —
and produces the actual text. If the changes need to be applied in bulk, the
google-ads-change-scripts skill can turn this output into executable scripts.

The skill covers three output types:

- **RSA Headlines & Descriptions**: Full sets of 15 headlines and 4 descriptions,
  or targeted replacements for underperforming assets
- **Extension/Asset Text**: Sitelinks, callouts, and structured snippets aligned
  with the campaign's top services and search intent
- **Pin Recommendations**: When specific headlines should be locked to positions
  (e.g., brand name pinned to H1, CTA pinned to H2)

## Before Writing Any Copy

### 1. Gather the Inputs

Good ad copy comes from data, not imagination. Before writing a single headline,
collect as much of the following as possible:

- **Search term report** (or top search terms from the MAA): What are people
  actually typing? This is the single most important input. Headlines that echo
  real search language get higher CTR and better Ad Relevance scores.
- **Quality Score components**: Which of the three (Expected CTR, Ad Relevance,
  Landing Page Experience) are Below Average? Each one points to a different copy
  fix.
- **Current ad copy**: What headlines and descriptions are already running? Don't
  duplicate — improve and diversify.
- **Ad strength rating**: Poor or Average usually means not enough thematic
  diversity across headlines. Google wants to see distinct angles, not 15
  variations of the same phrase.
- **Landing page content**: Headlines that promise something the landing page
  doesn't deliver will tank Landing Page Experience. Check alignment.
- **Service area and business context**: Local service businesses need geographic
  relevance in their copy — city names, neighborhood references, service area
  signals.

If the user comes in with an MAA already completed, most of this is already
available in the MAA output and the data that fed it. If they're starting fresh,
ask for at least the search term report and current ad copy.

### 2. Read the Narrative (if available)

If a client narrative document exists (`{Client}_MAA-Narrative.md`), read it
before writing copy. The narrative may contain:

- Prior ad copy changes and whether they worked
- DKI tests in progress
- Specific language the client prefers or avoids
- Themes that have been tested before

Don't suggest changes that were already tried and failed unless the context has
shifted enough to warrant a retry.

## The QS-to-Copy Fix Map

Quality Score has three components. Each one tells you something different about
what the copy needs. This mapping is the core diagnostic logic of the skill.

### Expected CTR: Below Average

The ads aren't compelling enough to earn clicks relative to competitors.

**What to fix:**
- Headlines need stronger hooks — specificity beats generality. "24/7 Emergency
  Plumber in Denver" beats "Professional Plumbing Services"
- Add urgency or scarcity where appropriate: "Same-Day Service", "Book Today —
  Slots Filling Fast"
- Include the core keyword naturally in Headline 1 or 2. Google weights early
  headlines heavily for relevance matching
- Test emotional angles: problem-aware ("Leaking Pipes?"), solution-aware
  ("Licensed Plumber Near You"), outcome-aware ("Get Your Water Flowing Again")
- Check if competitors are using offers or social proof that you aren't —
  "Rated 4.9 Stars", "500+ 5-Star Reviews", "$50 Off First Service"

### Ad Relevance: Below Average

The ad copy doesn't match what people are searching for closely enough.

**What to fix:**
- Pull exact phrases from the search term report into headlines. If people search
  "dumpster rental Dallas", a headline should say "Dumpster Rental in Dallas" —
  not "Waste Management Solutions"
- Create keyword-specific ad groups if one ad group serves too many intents.
  Generic copy trying to cover "dumpster rental" and "junk removal" and "roll off
  container" won't be relevant to any of them
- Check for intent mismatch: are the headlines answering a different question than
  what the searcher asked? Commercial intent searches ("cost of", "near me",
  "hire a") need commercial copy, not informational content
- Use Dynamic Keyword Insertion (DKI) in Headline 1 as a bridge while building
  out more specific ad groups: `{KeyWord:Default Headline}`

### Landing Page Experience: Below Average

The landing page doesn't deliver what the ad promises, or has UX issues.

**What copy can do (limited — this is mostly a landing page problem):**
- Align ad language with landing page language. If the landing page says "house
  painting" but the ad says "residential painting services," that mismatch hurts
- Remove promises from headlines that the landing page doesn't back up. Don't
  say "Free Estimates" if the landing page has no estimate form
- In the description, set expectations for what the user will find: "See Our
  Pricing | View Our Work | Get a Free Quote"

**What to flag for a separate LP fix:**
- Slow load times, poor mobile experience, thin content, missing trust signals —
  these are landing page issues, not copy issues. Note them for the AM but don't
  try to solve them through ad copy alone

## Writing RSA Headlines

### The 15-Headline Framework

Google allows up to 15 headlines per RSA. A strong set covers these angles:

**Slots 1-3: Core service + location (high pin priority)**
These are the workhorses. They include the primary keyword and geographic modifier.
At least one should be pin-worthy for H1.

> "Dumpster Rental in Dallas TX"
> "Roll Off Dumpsters — Dallas Area"
> "Same-Day Dumpster Delivery Dallas"

**Slots 4-6: Differentiators**
What makes this business different from the next Google result?

> "Family-Owned Since 2005"
> "Rated 4.9 Stars on Google"
> "Licensed, Bonded & Insured"

**Slots 7-9: Urgency / CTA**
Drive the click with action language.

> "Call Now — Same-Day Service"
> "Book Online in 2 Minutes"
> "Get Your Free Quote Today"

**Slots 10-12: Benefits / outcomes**
What does the customer get? Think past the service to the result.

> "Clean Up Your Property Fast"
> "Hassle-Free Rental Process"
> "Transparent Pricing — No Hidden Fees"

**Slots 13-15: Offer / social proof / seasonal**
Rotational slots that can be swapped as offers or seasons change.

> "$25 Off Your First Rental"
> "500+ Happy Customers in DFW"
> "Spring Cleanup Special — Book Now"

### Headline Rules

- **30 characters max** per headline. This is a hard limit — not a suggestion.
  Count characters carefully. Headlines over 30 characters will be rejected by
  Google Ads.
- **No redundancy.** Google penalizes RSAs where multiple headlines say the same
  thing differently. "Professional Plumbing" and "Expert Plumbing Services" are
  too similar. Each headline should introduce a new idea.
- **At least 3 headlines should be combinable with any other.** Google assembles
  RSAs dynamically — some headline pairs will look awkward together. Write with
  combinations in mind.
- **Include the keyword in at least 2-3 headlines**, but don't stuff it into all
  15. Google needs variety to test combinations.
- **Use title case** consistently unless the brand style is different.

## Writing RSA Descriptions

### The 4-Description Framework

Google allows up to 4 descriptions (90 characters max each). These expand on the
headlines and give the searcher a reason to click.

**Description 1: Value proposition + CTA**
The primary selling statement. This shows most often.

> "Dallas's top-rated dumpster rental service. Same-day delivery, transparent
> pricing, and hassle-free pickup. Get your free quote today."

**Description 2: Trust signals + differentiators**
Why choose this business over competitors?

> "Family-owned and locally operated since 2005. Licensed, bonded, and insured
> with 500+ five-star reviews from DFW homeowners."

**Description 3: Process / logistics**
How does it work? Reduce friction by previewing the experience.

> "Choose your dumpster size online, schedule delivery, and we handle the rest.
> Flexible rental periods with no surprise fees."

**Description 4: Offer / seasonal / specific use case**
Rotational slot that speaks to specific segments or promotions.

> "Perfect for home renovations, construction cleanups, and estate cleanouts.
> Multiple sizes available — call for same-day availability."

### Description Rules

- **90 characters max** per description. Count carefully.
- Each description should stand on its own — they don't always show together.
- Include a CTA in at least one description (ideally Description 1).
- Reference the geographic area in at least one description.

## Extensions and Assets

Extensions expand the ad's footprint and improve CTR. They're free real estate —
every campaign should have them.

### Sitelinks (minimum 4, ideally 8)

Each sitelink needs: a title (25 chars max), two description lines (35 chars each),
and a destination URL.

**Write sitelinks for the top services or actions:**

> **Title**: "Our Services"
> **Desc 1**: "Residential & commercial dumpster"
> **Desc 2**: "rental for any project size."
> **URL**: /services

> **Title**: "Pricing & Sizes"
> **Desc 1**: "See transparent pricing for"
> **Desc 2**: "10, 20, 30 & 40 yard dumpsters."
> **URL**: /pricing

> **Title**: "Get a Free Quote"
> **Desc 1**: "Request your custom quote in"
> **Desc 2**: "under 2 minutes — no obligation."
> **URL**: /quote

> **Title**: "Customer Reviews"
> **Desc 1**: "See why 500+ customers rated"
> **Desc 2**: "us 4.9 stars on Google."
> **URL**: /reviews

### Callout Extensions (minimum 4)

Short phrases (25 chars max each) that highlight benefits. No links.

> "Same-Day Delivery" | "No Hidden Fees" | "Licensed & Insured" |
> "Family-Owned" | "Flexible Rental Periods" | "Free Estimates"

### Structured Snippets

Category-based lists that show beneath the ad. Pick the header that fits:

> **Header**: "Services"
> **Values**: "Residential Rentals, Commercial Rentals, Construction Cleanup,
> Estate Cleanouts, Roofing Debris"

> **Header**: "Neighborhoods"
> **Values**: "Downtown Dallas, Plano, Frisco, McKinney, Allen, Richardson"

## Output Format

When delivering copy recommendations, structure the output clearly so the AM can
act on it directly. Use this format:

```markdown
# Ad Copy Recommendations — [Client Name]
## Generated: [Date]

### Context
[2-3 sentences on what triggered this: which QS components are Below Average,
what the search term data shows, what the MAA flagged]

### RSA Headlines (Campaign: [Campaign Name])
| # | Headline | Angle | Chars |
|---|----------|-------|-------|
| 1 | [headline text] | Core service + location | 28 |
| 2 | [headline text] | Core service variant | 26 |
| ... | ... | ... | ... |

### RSA Descriptions (Campaign: [Campaign Name])
| # | Description | Chars |
|---|-------------|-------|
| 1 | [description text] | 87 |
| ... | ... | ... |

### Pin Recommendations
[Which headlines to pin and why]

### Extensions
#### Sitelinks
[Table with title, desc 1, desc 2, URL]

#### Callouts
[Comma-separated list]

#### Structured Snippets
[Header + values]

### Implementation Notes
[Any caveats: "wait for DKI test to mature before swapping", "coordinate
with landing page update", etc.]
```

Save this as a markdown file in the client's output folder (see `CONFIGURATION.md`
at the repo root for the configurable base location):
`{OUTPUT_DIR}/{Client Name}/{Client}_AdCopy_{YYYY-MM-DD}.md`

## What This Skill Does NOT Do

- **Diagnose problems.** Use google-ads-analyzer (MAA) for that. This skill
  assumes the diagnosis is already done and acts on it.
- **Generate scripts.** Use google-ads-script for automation. This skill produces
  text, not code.
- **Fix landing pages.** If Landing Page Experience is the issue, this skill flags
  it and suggests copy alignment, but the actual LP fix is out of scope.
- **Write for Meta/Facebook, Display image ads, or Video.** This is Google Search
  text copy — RSAs, extensions, and assets.
