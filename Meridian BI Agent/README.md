# Meridian BI Agent

![alt text](image.png)

![alt text](image-1.png)

## 📌 Overview

**Meridian BI Agent v2** is an intelligent Slack-integrated Business Intelligence assistant built using **n8n**. It listens for Slack mentions, processes user queries, and responds with either:

* Direct knowledge-based answers, or
* Live financial data via integrated tools (stock prices, company profiles, news)

The workflow is designed to be **context-aware**, **tool-efficient**, and **thread-sensitive**, making it suitable for real-time business intelligence interactions.
---

## ⚙️ Architecture Summary

### 🔁 High-Level Flow

1. Slack Trigger (app mention)
2. Thread detection (If node)
3. Context formatting (thread vs non-thread)
4. AI Agent processing (LangChain)
5. Optional tool usage (Stock / Profile / News)
6. Response sent back to Slack thread

---

## 🧩 Workflow Components

### 1. 🔔 Slack Trigger

* **Node:** `Slack Trigger`
* **Event:** `app_mention`
* **Purpose:** Listens for when the bot is mentioned in Slack
* **Output:** Message payload including:

  * `text`
  * `channel`
  * `user`
  * `thread_ts`

---

### 2. 🔀 Thread Detection Logic

* **Node:** `If`
* **Condition:** Checks if `thread_ts` exists

| Condition | Path                          |
| --------- | ----------------------------- |
| TRUE      | Fetch thread messages         |
| FALSE     | Process as standalone message |

---

### 3. 🧵 Thread Context Handling

#### A. Thread Path

* **Node:** `Fetch Thread Messages`

  * Retrieves full conversation thread from Slack
* **Node:** `Format Thread Context`

  * Removes bot messages
  * Cleans mentions (`<@USER>`)
  * Builds conversation history string

**Output Example:**

```json
{
  "conversationHistory": "User: ...\nUser: ...",
  "currentMessage": "...",
  "isThread": true
}
```

---

#### B. Non-Thread Path

* **Node:** `Format Without Thread`
* Produces minimal context:

```json
{
  "conversationHistory": "",
  "currentMessage": "...",
  "isThread": false
}
```

---

### 4. 🧠 Meridian AI Agent

* **Node:** `Meridian Agent`
* **Type:** LangChain Agent
* **Model:** Groq (`llama-3.3-70b-versatile`)
* **Memory:** Redis-backed chat memory

#### 🧾 Prompt Strategy

The agent uses structured prompting:

**Input Logic:**

```text
IF thread:
  Previous conversation + new message
ELSE:
  Only current message
```

---

### 🧠 System Rules (Core Intelligence)

#### ✅ Direct Answer Rules

* General knowledge → answer directly
* No tool usage

#### 🛠 Tool Usage Rules

| User Intent    | Tool Used       |
| -------------- | --------------- |
| Stock price    | Stock Request   |
| Company info   | Company Profile |
| News / updates | Company News    |

⚠️ Strict enforcement:

* No unnecessary tool calls
* News must use **Company News tool ONLY**

---

## 🔌 Integrated Tools

### 📈 1. Stock Request

* **API:** Finnhub `/quote`
* **Use Case:** Current stock price
* **Key Field:** `"c"` = current price

---

### 🏢 2. Company Profile

* **API:** Finnhub `/profile2`
* **Returns:**

  * Industry
  * Market cap
  * Sector
  * Company details

---

### 📰 3. Company News

* **Type:** Sub-workflow (`News Fetcher`)
* **Returns:** 5 formatted headlines with URLs
* ⚠️ Output must be returned **exactly as received**

---

## 🧠 Memory System

* **Node:** `Redis Chat Memory`
* **Session Key:**

```js
thread_ts || channel
```

### Benefits:

* Maintains context within Slack threads
* Supports conversational continuity
* TTL: 1 hour

---

## 💬 Response Delivery

* **Node:** `Send Response`
* Sends output back to Slack:

  * Same channel
  * Same thread (if exists)

---

## 🧪 Example Interactions

### 📌 General Question

> “What is EBITDA?”

✅ Agent responds directly (no tools)

---

### 📌 Stock Query

> “What’s Apple’s stock price?”

✅ Uses Stock Request tool
→ Returns current price

---

### 📌 Company Info

> “Tell me about Tesla”

✅ Uses Company Profile tool

---

### 📌 News Query

> “What’s happening with Microsoft?”

✅ Uses Company News tool
→ Returns formatted headlines

---

## 🚀 Deployment Notes

### Requirements:

* Slack API credentials
* Groq API key
* Redis instance
* Finnhub API key

---

### 🔐 Environment Variables (Recommended)

* `FINNHUB_API_KEY`
* `GROQ_API_KEY`
* `REDIS_URL`
* `SLACK_API_TOKEN`

---

## ⚠️ Known Constraints

* Tool selection relies heavily on prompt accuracy
* News output must not be modified
* Thread detection depends on Slack payload integrity

---

## 🔧 Potential Improvements

* Add fallback handling for failed API calls
* Introduce rate limiting for external APIs
* Add logging/monitoring layer
* Improve entity extraction (ticker symbol detection)

---

## 🏁 Conclusion

This workflow is a **robust, production-ready Slack BI assistant** that:

* Maintains conversation context
* Uses tools intelligently
* Avoids unnecessary API calls
* Delivers concise, relevant insights

---

If you plan to scale this:

* Add more tools (earnings, analyst ratings)
* Introduce caching layer
* Add user-specific personalization

---
