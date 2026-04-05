# UrbanNest CX Automation

AI-Powered Customer Support Triage System using n8n, Grok, Freshdesk, and Google Sheets.

## Overview

This workflow automatically processes incoming support tickets by:
1. **Classifying tickets** into 6 categories using AI (Llama 3.3 70B)
2. **Enriching tickets** with customer order history from a database
3. **Generating draft responses** using templates or AI based on ticket type
4. **Routing tickets** intelligently to the appropriate team
5. **Logging analytics** for operational visibility
6. **Notifying teams** via email with full context

## Architecture

```
Freshdesk Ticket Created
        ↓
    Webhook (n8n receives ticket)
        ↓
    Google Sheets Lookup (customer data)
        ↓
    Enrich Ticket Data (combine context)
        ↓
    Groq AI Classification (category + urgency)
        ↓
    Update Freshdesk Tags
        ↓
    Route by Category (Switch)
        ├── ESCALATE → Handle Escalation (human required)
        ├── COMPLAINT → Handle Complaint (human required)
        ├── SHIPPING → Handle Shipping (template response)
        ├── RETURN → Handle Return (template response)
        ├── PRODUCT_QUESTION → Handle Product Question (AI response)
        └── Fallback → Handle Other (manual review)
                    ↓
            Prepare Analytics Data
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
Log to Analytics     Send Team Notification
(Google Sheets)           (Gmail)
```

## Categories

| Category | Description | Response Type |
|----------|-------------|---------------|
| SHIPPING | Order tracking, delivery status | Template-based |
| RETURN | Return/exchange requests | Template-based |
| PRODUCT_QUESTION | Pre-sale inquiries | AI-generated |
| ORDER_CHANGE | Order modifications | Manual review |
| COMPLAINT | Customer dissatisfaction | Human required |
| ESCALATE | Manager requests, threats | Human required |

## Smart Routing

| Category | Condition | Assigned To |
|----------|-----------|-------------|
| ESCALATE | Any | Senior Agent |
| COMPLAINT | Urgency ≥ 4 | Senior Agent |
| COMPLAINT | Urgency < 4 | Support Team |
| SHIPPING | Any | Shipping Team |
| RETURN | Any | Returns Team |
| PRODUCT_QUESTION | Any | Sales Team |
| Other | Any | Support Team |

## Features

### AI Classification
- Uses Groq's Llama 3.3 70B model
- 95%+ accuracy on ticket categorization
- Urgency scoring (1-5 scale)
- Fallback to human escalation if AI fails

### Context-Aware Responses
- Template-based for SHIPPING and RETURN (predictable, accurate)
- AI-generated for PRODUCT_QUESTION (flexible, informative)
- Human required for ESCALATE and high-urgency COMPLAINT

### Error Handling

| Failure Point | Fallback Strategy |
|---------------|-------------------|
| AI Classification fails | Auto-escalate with urgency 5 |
| Customer not in database | Ask for order number + email |
| Freshdesk API fails | Continue workflow, log error |
| Response generation fails | Post system notice for manual handling |
| Email notification fails | Continue without blocking |

### Edge Cases Handled
- Unknown customer → Asks for order details
- Pre-sale inquiry → Answers directly, no order number request
- Delayed order → Acknowledges delay, provides tracking
- Delivered but not received → Suggests carrier investigation
- Angry customer with multiple contacts → Escalates to senior agent

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or cloud)
- Groq API key
- Freshdesk account with API access
- Google account with Sheets API enabled
- Gmail API enabled

### Step 1: Import Workflow
1. Open n8n
2. Go to Workflows → Import
3. Paste the JSON from `UrbanNest-CX-Automation-Workflow.json`

### Step 2: Configure Credentials

Replace these placeholders in the workflow:

| Placeholder | Description |
|-------------|-------------|
| `YOUR_GROQ_API_KEY` | Get from [console.groq.com](https://console.groq.com) |
| `YOUR_SUBDOMAIN` | Your Freshdesk subdomain (e.g., `company.freshdesk.com`) |
| `YOUR_FRESHDESK_API_KEY` | Get from Freshdesk → Profile Settings → API Key |

### Step 3: Set Up Google Sheets

Create a Google Sheet named "UrbanNest Database" with two tabs:

**Tab 1: Order Database**
| Column | Type |
|--------|------|
| customer_email | Email |
| customer_segment | VIP / Standard / New |
| order_id | Text (e.g., ORD-2024-1001) |
| order_date | Date |
| order_total | Currency |
| items | Text |
| shipping_status | delivered / in_transit / delayed / shipped / processing |
| tracking_number | Text |

**Tab 2: Ticket Analytics**
| Column | Type |
|--------|------|
| Timestamp | DateTime |
| ticket_id | Number |
| customer_email | Email |
| customer_segment | Text |
| category | Text |
| urgency | Number |
| reasoning | Text |
| draft_generated | TRUE/FALSE |
| assigned_to | Email |
| freshdesk_link | URL |

### Step 4: Configure Freshdesk Webhook

1. Go to Freshdesk → Admin → Automations → Ticket Creation
2. Create a rule that triggers on ticket creation
3. Add action: "Trigger Webhook"
4. URL: `https://your-n8n-instance.com/webhook/freshdesk-ticket`
5. Method: POST
6. Content: Include ticket fields

### Step 5: Configure Google Sheets Node

1. Open the "Get row(s) in sheet" node
2. Connect your Google account (OAuth2)
3. Select your "UrbanNest Database" spreadsheet
4. Select "Order Database" sheet
5. **Important:** Go to Settings tab → Enable "Always Output Data"

### Step 6: Configure Gmail Node

1. Open the "Send Team Notification" node
2. Connect your Gmail account (OAuth2)
3. Enable Gmail API at [Google Cloud Console](https://console.developers.google.com/apis/api/gmail.googleapis.com/overview)

### Step 7: Activate Workflow

1. Save the workflow
2. Toggle "Active" to enable production mode
3. Test with a sample Freshdesk ticket

## Testing

### Test Cases to Verify

| Test | Expected Result |
|------|-----------------|
| Known customer, shipping inquiry | Template response with tracking |
| Unknown customer | Asks for order number + email |
| Pre-sale question ("Do you ship to X?") | AI answers directly, no order request |
| Angry customer demanding manager | ESCALATE, urgency 5, human required |
| Return request | Template with RMA process |
| Order status: delayed | Acknowledges delay, provides tracking |

## File Structure

```
├── UrbanNest-CX-Automation-Workflow.json    # n8n workflow (import this)
├── README.md                                  # This file
├── docs/
│   ├── UrbanNest-CX-Automation-Case-Study-Final.docx
│   └── UrbanNest-Presentation-Script.docx
```

## Technology Stack

| Component | Tool | Alternatives |
|-----------|------|--------------|
| Workflow Engine | n8n | Make.com, Zapier |
| AI/LLM | Groq (Llama 3.3 70B) | OpenAI GPT-4, Claude |
| Helpdesk | Freshdesk | Zendesk, Intercom |
| Database | Google Sheets | PostgreSQL, Airtable |
| Notifications | Gmail | Slack, MS Teams |