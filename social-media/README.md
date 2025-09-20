# Omni-Social Auto-Reply + Scheduler (IG/FB/TikTok/YouTube)

**By [Hazem Ali](https://github.com/DrHazemAli)**
Part of the [`DrHazemAli/n8n-workflows`](https://github.com/DrHazemAli/n8n-workflows) collection.

---

## Overview

This workflow is a full **social media automation system** built for **n8n**.
It lets you:

* Auto-reply to **comments and direct messages** across:

  * Instagram
  * Facebook
  * TikTok
  * YouTube
* Manage **multiple accounts and pages** with account-specific reply logic.
* Train unique reply styles for each account (support, friendly, sales, short, or custom persona).
* Schedule content publishing from **Google Drive + Google Sheets**:

  * Upload media with date/time, captions, and target account.
  * Workflow automatically publishes and marks tasks as `DONE`.

---

## Features

* **Multi-Account Reply Engine**
  Each event (comment/DM) is normalized and routed based on `platform:account_id`.
  Reply style and persona are customizable via Google Sheets.

* **Hybrid Reply Generation**

  * Use your own LLM endpoint (configurable per account).
  * Or rely on built-in fallback templates (price, booking, general reply).

* **Publishing Scheduler**

  * Cron checks Google Sheet every minute.
  * Finds due posts (`PENDING` status).
  * Downloads asset from Google Drive.
  * Publishes to the right platform.
  * Updates row to `DONE`.

* **Extensible**
  All nodes are modular. Easy to add other platforms, analytics, or advanced AI pipelines.

---

## Setup

### 1. Import Workflow

* Import `omni-social-n8n-workflow.json` into your n8n instance.

### 2. Create Google Sheets File

Create a Sheet named **SocialConfig** with two tabs:

#### **Accounts**

| A: key (`platform:account_id`) | B: access\_token | C: reply\_style | D: persona\_prompt | E: keywords\_csv | F: llm\_endpoint | G: llm\_api\_key |
| ------------------------------ | ---------------- | --------------- | ------------------ | ---------------- | ---------------- | ---------------- |
| instagram:1789xxxx             | …                | friendly        | رد ودي وسريع…      | price,booking    | (optional)       | (optional)       |

#### **Schedule**

| A: datetime\_iso (UTC) | B: platform | C: account\_key | D: drive\_file\_id | E: status | F: caption | G: extra\_json | H: target\_id |
| ---------------------- | ----------- | --------------- | ------------------ | --------- | ---------- | -------------- | ------------- |
| 2025-09-21T22:30:00Z   | instagram   | 1789xxxx        | 1AbCdEfGh…         | PENDING   | Caption…   | {}             | …             |

### 3. Credentials in n8n

* **Google API (Service Account)** – for Sheets.
* **Google Drive OAuth2** – for media download.

### 4. Environment Variables

| Variable                 | Description                         |
| ------------------------ | ----------------------------------- |
| `GSHEET_CONFIG_ID`       | Google Sheet ID for SocialConfig    |
| `GOOGLE_CREDENTIAL_ID`   | Credential name for Sheets          |
| `GOOGLE_DRIVE_CRED_ID`   | Credential name for Drive           |
| `TIKTOK_REPLY_ENDPOINT`  | (optional) TikTok reply API         |
| `TIKTOK_UPLOAD_ENDPOINT` | (optional) TikTok upload API        |
| `YT_UPLOAD_PROXY`        | (optional) Proxy for YouTube upload |

### 5. Webhook Setup

* **Instagram/Facebook**: configure Meta App webhook → `…/webhook/meta-webhook`
* **TikTok**: configure Business API webhook → `…/webhook/tiktok-webhook`
* **YouTube**: use Pub/Sub push or proxy to → `…/webhook/youtube-webhook`

---

## How It Works

1. **Webhook Receives Event**
   → Normalizer unifies schema.
   → Google Sheets lookup pulls account config.

2. **Reply Generation**

   * If LLM endpoint set → call HTTP request with persona + style.
   * Else → use fallback template.

3. **Reply Sent**

   * Routed to correct API (Meta Graph, TikTok, YouTube).

4. **Scheduler**

   * Cron runs every 1 min.
   * Fetches due posts (`PENDING`).
   * Downloads from Drive.
   * Publishes.
   * Marks row `DONE`.

---

## Notes

* Meta webhook challenge/verification step should be handled via proxy or function before hitting n8n.
* YouTube uploads typically require a proxy for resumable upload.
* Respect each platform’s automation and messaging policies.
* Extendable for analytics, CRM integration, or advanced AI workflows.

---

## License

MIT – use freely, adapt, and extend.
