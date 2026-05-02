# 🎯 The Review Sniper — Automated Reputation Management Engine

> **Project Code:** `RRE-AUTO-01`
> **Version:** 1.0.0
> **Built with:** n8n · OpenAI GPT-4o · Google Business API · Facebook Graph API · Telegram Bot · Google Sheets

***

## 📌 Table of Contents

- [Overview](#-overview)
- [Core Objective](#-core-objective)
- [Tech Stack](#-tech-stack)
- [System Architecture](#-system-architecture)
- [Workflow Deep Dive](#-workflow-deep-dive)
  - [Chain 1 — Review Poller](#chain-1--review-poller)
  - [Chain 2 — Owner Approval Handler](#chain-2--owner-approval-handler)
- [AI Prompt Engineering (R.I.C.E. Framework)](#-ai-prompt-engineering-rice-framework)
- [Setup & Configuration](#-setup--configuration)
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Google Business Profile API](#google-business-profile-api)
  - [Facebook Graph API](#facebook-graph-api)
  - [OpenAI API](#openai-api)
  - [Telegram Bot Setup](#telegram-bot-setup)
  - [Google Sheets Ledger](#google-sheets-ledger)
- [Importing the Blueprint into n8n](#-importing-the-blueprint-into-n8n)
- [How It Looks in Action](#-how-it-looks-in-action)
- [The Reputation Shield Protocol](#-the-reputation-shield-protocol)
- [Google Sheets Ledger Schema](#-google-sheets-ledger-schema)
- [Business Impact & Outcomes](#-business-impact--outcomes)
- [Portfolio Highlights](#-portfolio-highlights)
- [License](#-license)

***

## 🔍 Overview

**The Review Sniper** is a fully automated, AI-powered reputation management system built for boutique resorts and hospitality businesses. It continuously monitors your **Google Business Profile** and **Facebook Page** for new guest reviews, drafts professional brand-consistent responses using GPT-4o, and routes them through a **Human-in-the-Loop** approval gate on Telegram — before a single word is published publicly.

The system runs in two parallel automation chains:

| Chain | Name | Trigger | Purpose |
|-------|------|---------|---------|
| **Chain 1** | Review Poller | Every 60 minutes (Schedule) | Fetch, normalise, filter, AI-draft, and notify |
| **Chain 2** | Owner Approval Handler | Telegram Callback | Route owner's decision: Approve / Edit / Skip |

***

## 🎯 Core Objective

> **Eliminate "Response Latency" on reviews.**

High response velocity signals to Google's Local Search algorithm that a business is active and engaged — directly improving **Google Maps rankings** and winning the **"100% Response Rate"** badge on both Google and Facebook profiles.

The three business outcomes this system delivers:

1. **Immediate** — A push notification on the owner's phone for every new review, within 60 minutes of it appearing.
2. **Operational** — A professionally written, brand-consistent reply published automatically after owner approval.
3. **Long-term** — Higher Map pack rankings, increased discoverability, and a stronger public reputation.

***

## 🛠 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Workflow Orchestrator** | n8n (self-hosted or cloud) | Automation backbone — all workflow logic lives here |
| **AI Engine** | OpenAI GPT-4.1-mini / GPT-4o | Drafts warm, on-brand review responses |
| **Review Source 1** | Google Business Profile API v4 | Fetches latest Google reviews via OAuth2 |
| **Review Source 2** | Facebook Graph API v19.0 | Fetches latest Facebook ratings/reviews |
| **Control Interface** | Telegram Bot API | Owner approval interface (APPROVE / EDIT / SKIP) |
| **Database / Ledger** | Google Sheets | Deduplication check + audit trail of all replies |

***

## 🏗 System Architecture

The diagram below illustrates the complete two-chain architecture:

```
┌─────────────────────────────────────────────────────────────────────┐
│  CHAIN 1 — REVIEW POLLER (runs every 60 min)                        │
│                                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐          │
│  │ Schedule │───▶│ Fetch Google │───▶│ Normalise Google │──┐       │
│  │ Trigger  │    │ Reviews      │    │ Reviews          │  │       │
│  └──────────┘    └──────────────┘    └──────────────────┘  │       │
│       │                                                      ▼       │
│       │          ┌──────────────┐    ┌──────────────────┐  ┌────┐  │
│       └─────────▶│ Fetch        │───▶│ Normalise        │─▶│    │  │
│                  │ Facebook     │    │ Facebook Reviews  │  │Mrg │  │
│                  │ Reviews      │    └──────────────────┘  └─┬──┘  │
│                  └──────────────┘                            │       │
│                                                              ▼       │
│  ┌────────────────────┐   ┌──────────────┐   ┌─────────────────┐  │
│  │ Check If Already   │◀──│ Merge All    │   │ Filter: New     │  │
│  │ Processed          │   │ Reviews      │──▶│ Reviews Only    │  │
│  └────────────────────┘   └──────────────┘   └────────┬────────┘  │
│                                                         ▼           │
│  ┌──────────────────┐   ┌──────────────────────┐  ┌──────────────┐│
│  │ Notify Owner via │◀──│ AI Draft Reply        │◀─│ (NEW reviews)││
│  │ Telegram         │   │ (GPT-4o)              │  └──────────────┘│
│  └──────────────────┘   └──────────────────────┘                   │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────────────┐                                           │
│  │ Log to Google Sheets │                                           │
│  │ (Status: PENDING)    │                                           │
│  └──────────────────────┘                                           │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  CHAIN 2 — OWNER APPROVAL HANDLER (Telegram Callback Trigger)       │
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────┐  │
│  │ Telegram Callback│───▶│ Parse Telegram   │───▶│ Route:       │  │
│  │ Trigger          │    │ Callback         │    │ APPROVE /    │  │
│  └──────────────────┘    └──────────────────┘    │ EDIT / SKIP  │  │
│                                                    └──────┬───────┘  │
│                            ┌──────────────┐              │           │
│                            │ Acknowledge  │        ┌─────┴─────┐    │
│                            │ Telegram     │        │           │    │
│                            │ Button       │     APPROVE      EDIT   │
│                            └──────────────┘        │           │    │
│                                                     ▼           ▼    │
│  ┌───────────────────┐   ┌──────────────────┐  ┌─────────────────┐ │
│  │ Get Review Data   │◀──│ Platform Router  │  │ Ask for Edited  │ │
│  │ from Sheet        │   │ Google/Facebook? │  │ Reply (Telegram)│ │
│  └───────────────────┘   └────────┬─────────┘  └─────────────────┘ │
│                                    │                                  │
│             ┌──────────────────────┤                                  │
│             ▼                      ▼                                  │
│  ┌──────────────────┐   ┌──────────────────┐                        │
│  │ Post Reply to    │   │ Post Reply to    │                        │
│  │ Google           │   │ Facebook         │                        │
│  └──────────────────┘   └──────────────────┘                        │
│             │                      │                                  │
│             └───────────┬──────────┘                                 │
│                         ▼                                             │
│  ┌──────────────────────────┐   ┌──────────────────────┐            │
│  │ Update Ledger: REPLIED   │──▶│ Confirm to Owner     │            │
│  │ (Google Sheets)          │   │ (Telegram message)   │            │
│  └──────────────────────────┘   └──────────────────────┘            │
│                                                                     │
│  ─── SKIP path ──▶  Update Ledger: SKIPPED                         │
└─────────────────────────────────────────────────────────────────────┘
```

***

## 🔬 Workflow Deep Dive

### Chain 1 — Review Poller

**Trigger:** `Schedule Trigger` — fires every **60 minutes**, 24/7.

***

#### Step 1 & 2: Fetch Reviews (Parallel)

Two HTTP Request nodes fire **simultaneously** — one for each platform:

**Google Business Profile:**
```
GET https://mybusiness.googleapis.com/v4/accounts/{{GOOGLE_ACCOUNT_ID}}/locations/{{GOOGLE_LOCATION_ID}}/reviews
  Auth: OAuth2
  pageSize: 10
```

**Facebook Graph API:**
```
GET https://graph.facebook.com/v19.0/{{FB_PAGE_ID}}/ratings
  Auth: Header (Page Access Token)
  fields: reviewer, rating, review_text, created_time, open_graph_story
  limit: 10
```

***

#### Step 3: Normalise Reviews

Both APIs return data in completely different formats. Two **Code nodes** (JavaScript) transform each platform's raw response into a **unified, flat schema**:

```javascript
// Normalised Review Object
{
  platform:          "Google" | "Facebook",
  reviewId:          string,   // unique identifier
  reviewerName:      string,   // display name or "A Guest"
  starRating:        string,   // e.g. "FIVE" (Google) or "5 STARS" (Facebook)
  starRatingNumeric: number,   // 1 – 5 (numeric for AI prompt logic)
  reviewText:        string,   // review content or "(No written review)"
  createTime:        string,   // ISO timestamp
  reviewUrl:         string    // direct link to the review
}
```

***

#### Step 4: Merge All Reviews

A **Merge node** (SQL mode) combines both normalised arrays into a single stream using `UNION ALL`:
```sql
SELECT * FROM t1 UNION ALL SELECT * FROM t2
```

***

#### Step 5: Check If Already Processed

A **Google Sheets node** reads the master Ledger sheet and cross-references `reviewId` values to prevent duplicate processing and double-replies.

***

#### Step 6: Filter — New Reviews Only

A **Filter node** passes only reviews with a `reviewId` not already in the Ledger. Duplicates are silently discarded.

***

#### Step 7: AI Draft Reply (GPT-4.1-mini / GPT-4o)

The **OpenAI node** processes each new review with a carefully engineered system prompt (see [AI Prompt Engineering](#-ai-prompt-engineering-rice-framework) below).

- Model: `gpt-4.1-mini` (configurable to `gpt-4o`)
- Max Tokens: 300
- Temperature: 0.7
- Output: A 2–3 sentence brand-consistent reply

***

#### Step 8: Notify Owner via Telegram

The bot sends the owner a formatted message:

```
🏨 New Google Review — 5⭐

👤 Reviewer: Maria Santos
📝 Review: "Absolutely stunning resort! The staff went above and beyond."

---
🤖 AI Draft Reply:
Thank you so much, Maria! We're overjoyed to hear that our team made your stay truly special...

---
⚠️ Action Required — Reply within 60 min for best SEO impact.

[✅ APPROVE]  [✏️ EDIT]  [⏭️ SKIP]
```

***

#### Step 9: Log to Google Sheets (Pending)

The review is immediately logged to the Ledger with status **PENDING** — creating an audit trail regardless of the final outcome.

***

### Chain 2 — Owner Approval Handler

**Trigger:** `Telegram Callback Trigger` — fires whenever the owner taps a button in the bot.

***

#### Step 1: Parse Telegram Callback

A **Code node** parses the callback data embedded in the button press. Button data is structured as:

```
ACTION::reviewId::platform
```

For example: `APPROVE::ChIJxxx123::Google`

The parser extracts: `action`, `reviewId`, `platform`, `chatId`, `messageId`, and the `aiDraft` text from the original message.

***

#### Step 2: Acknowledge Telegram Button

A `answerCallbackQuery` call instantly removes the "loading spinner" from the button in the owner's Telegram chat — confirming the tap was registered.

***

#### Step 3: Route — APPROVE / EDIT / SKIP

A **Switch node** routes the flow based on the action:

| Action | Route | What Happens |
|--------|-------|-------------|
| `APPROVE` | → Post Reply path | Proceeds to publish the AI draft as-is |
| `EDIT` | → Edit path | Sends a prompt to the owner asking for their custom text |
| `SKIP` | → Skip path | Marks review as SKIPPED in the Ledger, no reply posted |

***

#### Step 4: Get Review Data from Sheet

For APPROVE/EDIT routes, the full review data (reviewer name, AI draft, platform, etc.) is fetched from the Ledger using the `reviewId`.

***

#### Step 5: Platform Router — Google or Facebook?

A **Filter node** checks the `platform` field and routes to the correct API publisher.

***

#### Step 6: Post Reply

**Google:**
```
PUT https://mybusiness.googleapis.com/v4/{reviewId}/reply
Body: { "comment": "{approvedReplyText}" }
Auth: OAuth2
```

**Facebook:**
```
POST https://graph.facebook.com/v19.0/{reviewId}/comments
Body: { "message": "{approvedReplyText}", "access_token": "{FB_PAGE_ACCESS_TOKEN}" }
```

***

#### Step 7: Update Ledger — REPLIED / SKIPPED

The Google Sheets row is updated with the final status and timestamp, completing the audit trail.

***

#### Step 8: Confirm to Owner

A final Telegram message confirms the action:

```
✅ Reply Posted Successfully!

📍 Platform: Google
👤 Reviewer: Maria Santos
💬 Reply: "Thank you so much, Maria!..."

🎉 Your response rate stays at 100%!
```

***

## 🧠 AI Prompt Engineering (R.I.C.E. Framework)

The system prompt follows the **R.I.C.E.** framework for structured AI output:

| Component | Value |
|-----------|-------|
| **Role** | Professional Resort Manager for *Hacienda Verde Eco-Resort* |
| **Input** | Reviewer name, star rating (numeric), review text, platform source |
| **Constraint** | Warm eco-conscious tone. 2–3 sentences only. No "Draft:" prefix. |
| **Expectation** | Brand-consistent public reply OR Reputation Shield Protocol for < 3 stars |

**System Prompt (abbreviated):**
```
You are a professional Resort Manager for Hacienda Verde Eco-Resort — 
a boutique, eco-conscious paradise. Your job is to write warm, genuine, 
brand-consistent replies to guest reviews.

BRAND VOICE:
- Warm, personal, and grateful
- Eco-conscious and nature-inspired language
- Never defensive, always empathetic
- Address the reviewer by first name

REPUTATION SHIELD PROTOCOL (for reviews < 3 stars):
1. Acknowledge their specific concern sincerely
2. Offer a genuine apology for falling short
3. Move the conversation private: 'please reach us at guestcare@haciendaverde.com'
4. NEVER argue or make excuses

FORMAT: Reply must be 2-3 sentences only.
```

***

## ⚙️ Setup & Configuration

### Prerequisites

- n8n instance (self-hosted via Docker, or [n8n Cloud](https://n8n.io))
- Node.js 18+ (for self-hosted)
- A Google Cloud Project with the **My Business API** enabled
- A Facebook Developer App with **pages_read_engagement** and **pages_manage_posts** permissions
- An OpenAI account with API access
- A Telegram Bot (created via [@BotFather](https://t.me/BotFather))
- A Google Sheets file set up as the review ledger

***

### Environment Variables

All sensitive values are stored as **n8n Variables** (not hardcoded in the workflow). Configure these in your n8n instance under **Settings → Variables**:

| Variable Name | Description | Example |
|--------------|-------------|---------|
| `GOOGLE_ACCOUNT_ID` | Your Google Business Account ID | `accounts/1234567890` |
| `GOOGLE_LOCATION_ID` | Your Google Business Location ID | `locations/9876543210` |
| `GOOGLE_PLACE_ID` | Google Maps Place ID (for review URL) | `ChIJxxxxxxxxxxxxxxx` |
| `FB_PAGE_ID` | Your Facebook Page ID | `123456789012345` |
| `FB_PAGE_ACCESS_TOKEN` | Facebook Page Access Token | `EAABwzLixxx...` |
| `TELEGRAM_OWNER_CHAT_ID` | Your Telegram Chat ID | `987654321` |

***

### Google Business Profile API

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use an existing one)
3. Enable the **My Business Business Information API** and **My Business Reviews API**
4. Create **OAuth 2.0 credentials** (Desktop or Web App type)
5. In n8n, add a new **Google OAuth2 API** credential using your Client ID and Secret
6. Assign the credential to both Google-related nodes in Chain 1

***

### Facebook Graph API

1. Go to [Meta for Developers](https://developers.facebook.com)
2. Create a new App (Business type)
3. Add the **Facebook Login** product and request `pages_read_engagement`, `pages_manage_posts`, `pages_show_list` permissions
4. Generate a **Page Access Token** for your business page
5. In n8n, add an **HTTP Header Auth** credential with the token as the header value
6. Assign to the `Fetch Facebook Reviews` node

***

### OpenAI API

1. Get your API key from [platform.openai.com](https://platform.openai.com/api-keys)
2. In n8n, add a new **OpenAI API** credential
3. The workflow uses `gpt-4.1-mini` by default — change to `gpt-4o` in the AI node for higher quality drafts (higher cost)

***

### Telegram Bot Setup

1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. Send `/newbot` and follow the prompts to create your bot
3. Save the **Bot Token** provided
4. In n8n, add a new **Telegram API** credential with the bot token
5. Start a conversation with your bot and send any message
6. Find your Chat ID using: `https://api.telegram.org/bot<TOKEN>/getUpdates`
7. Set `TELEGRAM_OWNER_CHAT_ID` in n8n Variables

***

### Google Sheets Ledger

Create a Google Sheet with the following columns. This sheet serves as the deduplication database and audit trail:

| Column | Description |
|--------|-------------|
| `reviewId` | Unique review identifier from the platform |
| `platform` | `Google` or `Facebook` |
| `reviewerName` | Guest's name |
| `starRatingNumeric` | Rating (1–5) |
| `reviewText` | Full review text |
| `aiDraft` | AI-generated reply |
| `status` | `PENDING`, `REPLIED`, or `SKIPPED` |
| `repliedAt` | Timestamp when reply was posted |
| `createTime` | When the original review was written |
| `reviewUrl` | Direct link to the review |

> ⚠️ **Important:** The Google Sheet ID must be updated in the following nodes after creation:
> - `Check If Already Processed`
> - `Log to Google Sheets (Pending)`
> - `Get Review Data from Sheet`
> - `Update Ledger: REPLIED`
> - `Update Ledger: SKIPPED`

***

## 📥 Importing the Blueprint into n8n

1. Download the file: `RRE-AUTO-01-The-Review-Sniper.json`
2. Open your n8n instance
3. Click **"+"** to create a new workflow
4. Click the **"..."** menu (top right) → **Import from File**
5. Select the downloaded JSON file
6. The entire two-chain workflow will appear on your canvas
7. Update all **credentials** (Google OAuth2, Facebook Token, OpenAI, Telegram)
8. Update all **n8n Variables** (see Environment Variables table above)
9. Update the **Google Sheets Document IDs** in all sheet nodes
10. **Activate** the workflow — Chain 1 will begin polling immediately

> 💡 **Tip:** Test Chain 1 manually first by clicking **"Execute Workflow"** before activating the schedule. Check the Telegram notification arrives correctly before going live.

***

## 👀 How It Looks in Action

**Telegram Notification (Owner's Phone):**
```
🏨 New Google Review — 5⭐

👤 Reviewer: James Whitmore
📝 Review: "One of the best eco-resorts I've ever stayed at. 
The treehouse villa was magical."

---
🤖 AI Draft Reply:
Thank you for your kind words, James! We're so happy the 
treehouse villa captured your heart — it's one of our 
favourite spaces too. We hope to welcome you back to your 
little piece of the canopy soon! 🌿

---
⚠️ Action Required — Reply within 60 min for best SEO impact.
```

**Owner taps [✅ APPROVE]** → Reply is posted to Google in seconds.

**Confirmation message:**
```
✅ Reply Posted Successfully!

📍 Platform: Google
👤 Reviewer: James Whitmore
💬 Reply: "Thank you for your kind words, James!..."

🎉 Your response rate stays at 100%!
```

***

## 🛡 The Reputation Shield Protocol

For reviews rated **less than 3 stars**, the AI automatically switches to the **Reputation Shield Protocol** — a crisis management response strategy designed to de-escalate publicly while moving the conversation to a private channel.

**Protocol Rules:**
1. **Acknowledge** the guest's specific concern by name
2. **Apologise** sincerely — no excuses, no defensiveness
3. **Offer Resolution** privately: redirect to `guestcare@[resort].com`
4. **Never argue** or dispute the guest's experience publicly

**Example output for a 2-star review:**
> *"We're truly sorry to hear your stay didn't meet expectations, Elena. Your comfort and experience are our top priority, and we'd love the chance to understand what went wrong — please reach us directly at guestcare@haciendaverde.com so we can make this right."*

***

## 📊 Google Sheets Ledger Schema

The ledger serves dual purposes: **deduplication** (preventing duplicate replies) and **audit trail** (complete history of all review interactions).

```
+------------+----------+--------------+--------+------------------+----------+---------+
| reviewId   | platform | reviewerName | rating | status           | repliedAt| aiDraft |
+------------+----------+--------------+--------+------------------+----------+---------+
| ChIJxxx001 | Google   | Maria Santos |   5    | REPLIED          | 2026-... | Thank.. |
| fb_001xyz  | Facebook | John Doe     |   4    | REPLIED          | 2026-... | We're.. |
| ChIJxxx002 | Google   | A Guest      |   2    | SKIPPED          | 2026-... | We're.. |
| fb_002abc  | Facebook | Lisa Park    |   5    | PENDING          |          | Your s..|
+------------+----------+--------------+--------+------------------+----------+---------+
```

**Status Values:**

| Status | Meaning |
|--------|---------|
| `PENDING` | Review fetched, AI draft created, owner notified — awaiting decision |
| `REPLIED` | Owner approved, reply successfully posted to platform |
| `SKIPPED` | Owner skipped — no reply posted, review logged for record |

***

## 📈 Business Impact & Outcomes

| Metric | Before Automation | After Automation |
|--------|-----------------|-----------------|
| Average Response Time | 24–72 hours | < 60 minutes |
| Response Rate | ~40–60% | ~100% |
| Time Spent on Reviews | 2–3 hours/week | < 10 min/week |
| Google Maps Ranking Signal | Weak | Strong (active business) |
| Review Replies Tone | Inconsistent | Brand-consistent |

***

## 💼 Portfolio Highlights

This system demonstrates the following advanced skills:

1. **Multi-Platform API Integration** — Real-world OAuth2 authentication with Google's My Business API and header-auth with Facebook Graph API v19.0.

2. **Data Normalisation Pipeline** — JavaScript code nodes transform two differently-structured API responses into a single, consistent data schema using a SQL UNION pattern.

3. **AI Prompt Engineering** — Structured R.I.C.E. framework prompt with conditional logic (Reputation Shield for negative reviews), brand voice constraints, and output formatting rules.

4. **Human-in-the-Loop Design** — Telegram inline keyboard buttons with callback data encoding (`ACTION::reviewId::platform`) demonstrate advanced bot interaction design — keeping humans in control of AI output.

5. **Audit Trail & Deduplication** — Google Sheets used as a lightweight but effective operational database for status tracking, deduplication, and historical reporting.

6. **Dual-Chain Async Architecture** — Two independent n8n workflows that communicate via shared data state (Google Sheets), demonstrating event-driven, asynchronous automation design.

***

<img width="1530" height="635" alt="RRE-AUTO-01 – The Review Sniper Schema" src="https://github.com/user-attachments/assets/021d7c59-4d02-4b80-a87b-fe669947fa22" />


## 📄 License

This project is part of the **Reputation & Review Engine (RRE)** automation suite.

Built by **Suneel Pervez** — Automation Specialist & B2B Systems Designer.

- 🌐 [FirstlinkAI.com](https://firstlinkai.com)

***

> *"Speed of response is the new competitive advantage in hospitality. The Review Sniper makes 100% response rate the default, not the exception."*
