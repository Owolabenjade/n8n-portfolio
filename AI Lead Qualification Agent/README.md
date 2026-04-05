# AI Lead Qualification Agent

An autonomous n8n workflow that qualifies inbound leads using AI, generates personalized responses, and routes hot leads to your sales team in real-time.

## What It Does

When a new lead comes in, this agent automatically:

1. **Scores them against your ICP criteria** using Groq's Llama 3.1 70B model
2. **Routes qualified vs unqualified leads** based on score threshold
3. **Generates a personalized email response** tailored to each lead
4. **Sends the email** via Gmail
5. **Logs everything** to Google Sheets for tracking and reporting
6. **Alerts you on Slack** for hot leads (score 8+) requiring immediate follow-up

No manual review. No copy-paste. The agent handles Tier 1 lead qualification autonomously.

---

## Architecture

```
┌─────────────────┐
│  Webhook        │  ← Lead submits form / API call
│  (Lead Intake)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Groq LLM       │  ← Scores lead 1-10 against ICP
│  (Lead Scoring) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Parse Score    │  ← Extracts JSON, handles errors
│  (Code Node)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Is Qualified?  │  ← Score >= 7 = qualified
│  (IF Node)      │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│ YES   │ │ NO    │
└───┬───┘ └───┬───┘
    │         │
    ▼         ▼
┌───────────────┐ ┌───────────────┐
│ Generate      │ │ Generate      │
│ Qualified     │ │ Polite        │
│ Response      │ │ Decline       │
└───────┬───────┘ └───────┬───────┘
        │                 │
        └────────┬────────┘
                 │
                 ▼
        ┌─────────────────┐
        │  Merge Branches │
        └────────┬────────┘
                 │
                 ▼
        ┌─────────────────┐
        │  Log to         │  ← Records all data
        │  Google Sheets  │
        └────────┬────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
┌─────────────┐   ┌─────────────┐
│ Send Email  │   │ Is Hot Lead?│
│ via Gmail   │   │ (Score 8+)  │
└─────────────┘   └──────┬──────┘
                         │
                         ▼ (if yes)
                  ┌─────────────┐
                  │ Slack Alert │
                  └─────────────┘
```

---

## Tech Stack

| Component | Tool | Cost |
|-----------|------|------|
| Workflow Automation | n8n (self-hosted or cloud) | Free / $20+/mo |
| LLM Provider | Groq API (Llama 3.1 70B) | Free tier |
| Email | Gmail API | Free |
| Database | Google Sheets | Free |
| Notifications | Slack Webhooks | Free |

---

## Setup Instructions

### Prerequisites

- n8n instance (self-hosted via Docker or n8n Cloud)
- Groq API key (free at [console.groq.com](https://console.groq.com))
- Google Cloud project with Gmail and Sheets APIs enabled
- Slack workspace with incoming webhooks enabled

### Step 1: Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** → **Import from File**
3. Select `ai-lead-qualification-agent.json`
4. The workflow will load with all nodes pre-configured

### Step 2: Set Up Credentials

#### Groq API (for LLM scoring and response generation)

1. Go to [console.groq.com](https://console.groq.com) and create a free account
2. Generate an API key
3. In n8n, go to **Credentials** → **Add Credential** → **Header Auth**
4. Configure:
   - **Name:** `Groq API`
   - **Header Name:** `Authorization`
   - **Header Value:** `Bearer YOUR_GROQ_API_KEY`
5. Save and update the credential ID in these nodes:
   - `Score Lead (Groq LLM)`
   - `Generate Qualified Response`
   - `Generate Polite Decline`

#### Google Sheets (for logging)

1. In n8n, go to **Credentials** → **Add Credential** → **Google Sheets OAuth2 API**
2. Follow the OAuth flow to connect your Google account
3. Create a Google Sheet with these columns:
   - `Timestamp`
   - `Name`
   - `Email`
   - `Company`
   - `Message`
   - `Score`
   - `Qualified`
   - `Reasoning`
   - `Response Type`
   - `Email Sent`
4. Copy the spreadsheet ID from the URL (the long string between `/d/` and `/edit`)
5. Update the `Log to Google Sheets` node with your spreadsheet ID

#### Gmail (for sending responses)

1. In n8n, go to **Credentials** → **Add Credential** → **Gmail OAuth2 API**
2. Follow the OAuth flow to connect your Gmail account
3. Update the credential in the `Send Email via Gmail` node

#### Slack Webhook (for hot lead alerts)

1. Go to [api.slack.com/apps](https://api.slack.com/apps) and create a new app
2. Enable **Incoming Webhooks**
3. Add a webhook to your desired channel
4. Copy the webhook URL
5. Replace `YOUR_SLACK_WEBHOOK_URL` in the `Slack Hot Lead Alert` node

### Step 3: Customize ICP Criteria

Edit the system prompt in the `Score Lead (Groq LLM)` node to match your Ideal Customer Profile:

```
Score leads 1-10 based on these ICP criteria:
- Company size: 20-200 employees (ideal)
- Industry: SaaS, Agency, or E-commerce (ideal)
- Need: Automation, AI agents, or workflow optimization
- Budget signals: mentions growth, scaling, or specific tools
```

Modify the criteria, weights, and scoring logic to fit your business.

### Step 4: Update Email Templates

Edit the system prompts in:
- `Generate Qualified Response` — customize the tone, CTA, and Calendly link
- `Generate Polite Decline` — adjust the decline message and alternative suggestions

### Step 5: Activate the Workflow

1. Click **Save** in n8n
2. Toggle the workflow to **Active**
3. Copy the webhook URL from the `New Lead Webhook` node
4. Connect this URL to your lead capture forms, landing pages, or CRM

---

## Testing the Workflow

### Test Payload

Send a POST request to your webhook URL with this sample data:

```json
{
  "name": "Sarah Johnson",
  "email": "sarah@techstartup.io",
  "company": "TechStartup.io",
  "message": "Interested in automating our outbound sales process. We are a 50-person SaaS company looking to scale our SDR team without hiring."
}
```

### Expected Behavior

1. Lead is scored (this example should score 8-9/10)
2. Qualified response is generated with personalized content
3. Email is sent to the lead
4. Data is logged to Google Sheets
5. Slack alert fires (score >= 8)
6. Webhook returns success response

### Testing Unqualified Leads

```json
{
  "name": "John Doe",
  "email": "john@randomcompany.com",
  "company": "Random Company",
  "message": "Just browsing, not sure what I need"
}
```

This should score 3-4/10 and trigger the polite decline branch.

---

## Workflow Nodes Reference

| Node | Type | Purpose |
|------|------|---------|
| New Lead Webhook | Webhook | Receives incoming lead data via POST request |
| Score Lead (Groq LLM) | HTTP Request | Calls Groq API to score lead against ICP |
| Parse Score | Code | Extracts JSON from LLM response, handles errors |
| Is Qualified? | IF | Routes based on qualification status |
| Generate Qualified Response | HTTP Request | Creates personalized email for qualified leads |
| Generate Polite Decline | HTTP Request | Creates polite decline for unqualified leads |
| Parse Qualified Email | Code | Extracts email content from LLM response |
| Parse Decline Email | Code | Extracts email content from LLM response |
| Merge Branches | Merge | Combines both paths for unified logging |
| Log to Google Sheets | Google Sheets | Records all lead data and responses |
| Send Email via Gmail | Gmail | Delivers the AI-generated email |
| Is Hot Lead? | IF | Checks if score >= 8 for Slack alert |
| Slack Hot Lead Alert | HTTP Request | Posts formatted alert to Slack channel |
| Respond to Webhook | Respond to Webhook | Returns confirmation to lead source |

---

## Customization Options

### Adjust Qualification Threshold

In the `Is Qualified?` node, change the score threshold:
- Current: `score >= 7` = qualified
- Lower threshold: More leads qualify, higher volume
- Higher threshold: Fewer leads qualify, higher quality

### Add Lead Enrichment

Insert an HTTP Request node after the webhook to enrich lead data:
- Clearbit API for company info
- Hunter.io for email verification
- LinkedIn API for job title verification

### Add CRM Integration

After the `Log to Google Sheets` node, add:
- HubSpot node to create/update contacts
- Salesforce node to create leads
- Pipedrive node to add deals

### Add Calendar Booking

For qualified leads, add a Calendly or Cal.com integration to:
- Check availability
- Auto-book meetings
- Send calendar invites

---

## Error Handling

The workflow includes built-in error handling:

### LLM Response Parsing

The `Parse Score` node handles malformed LLM responses:

```javascript
let parsed;
try {
  parsed = JSON.parse(content);
} catch (e) {
  // Fallback if LLM does not return clean JSON
  parsed = { score: 5, reasoning: "Could not parse response", qualified: false };
}
```

Leads with parsing errors default to score 5 (borderline) for manual review.

### Recommended: Add Error Workflow

Create a separate error workflow that:
1. Catches failed executions
2. Logs errors to a dedicated sheet
3. Sends Slack alert for failed leads
4. Queues leads for manual processing

---

## Performance Metrics

Track these KPIs in your Google Sheet:

| Metric | How to Calculate |
|--------|------------------|
| Lead Volume | Count of rows per day/week |
| Qualification Rate | Qualified leads / Total leads |
| Average Score | Mean of Score column |
| Hot Lead Rate | Leads with score 8+ / Total leads |
| Response Time | Time from webhook to email sent |

---

## Cost Analysis

### Comparison: AI Agent vs Human SDR

| Solution | Annual Cost |
|----------|-------------|
| 1 Full-Time SDR | $80,000 - $120,000 |
| AI Lead Qualification Agent | $240 - $1,200/year |

### Agent Cost Breakdown

| Component | Monthly | Annual |
|-----------|---------|--------|
| n8n Cloud (Starter) | $20 | $240 |
| Groq API | $0 (free tier) | $0 |
| Google Workspace | $0 (existing) | $0 |
| Slack | $0 (free tier) | $0 |
| **Total** | **$20** | **$240** |

### ROI

- Replace 1 SDR with AI agent = **$79,000+ saved annually**
- Or: Free existing SDRs to focus only on qualified leads = **2-3x productivity**

---

## Future Enhancements

- [ ] Add voice agent for phone qualification (Vapi, Bland.ai)
- [ ] Integrate with LinkedIn for enrichment
- [ ] Add A/B testing for email templates
- [ ] Build dashboard for real-time metrics
- [ ] Add multi-language support

---