# Social Media Automation
### AI-Powered Research, Content Generation & Multi-Platform Publishing (n8n Workflow)

---

## Overview

This is a **two-system n8n automation workflow** combined into a single JSON file. It takes a topic submitted by a user, autonomously researches the web for credible sources, generates platform-tailored social media content using AI, and publishes approved posts across **LinkedIn, Instagram, Twitter (X), Medium, Facebook, and Slack** — all with human-in-the-loop approval gates at every critical step.

---

## The Two Systems

### System 1 — Research & Source Verification
Triggered by a web form. The user submits a topic and their email. The AI searches for up to 3 credible, recent articles on the topic, then emails the user a beautifully formatted HTML email asking them to **verify the sources** before any content is generated.

### System 2 — Content Generation & Publishing
Triggered by a webhook (the "Verify All Sources" button click in the email). Once the user approves the sources, the workflow generates an AI image, writes tailored posts for 6 platforms using Gemini 2.5 Flash, sends each post to the user for individual approval via Gmail's Send & Wait feature, and then publishes approved posts to the correct platforms.

---

## Full Workflow Diagram

```
[System 1 — Research & Source Verification]

Form Trigger (Email + Topic)
        │
        ▼
Basic LLM Chain  ◄──── OpenRouter (Qwen3-235B)
  + Structured Output Parser
        │
        ▼
Data Formatting (JavaScript)
   ┌────┴──────────────────┐
   ▼                       ▼
Keep Record            Build Verification
(Google Sheets)        Email (JavaScript)
                           │
                           ▼
                     Send Email (Gmail)
                     [User clicks "Verify All Sources"]


[System 2 — Content Generation & Publishing]

Webhook (Source Approval)
   ┌──────────────┴────────────────┐
   ▼                               ▼
Basic LLM Chain1               Image Generator
(Article Writer)               (HuggingFace FLUX)
◄── OpenRouter (Qwen3-235B)        │
   │                               ▼
   ▼                          Upload to Google Drive
Message a model1                   │
(Gemini 2.5 Flash)                 │
   │                               │
   ▼                               ▼
Make Output Appropriate ◄────── Merge
(JavaScript Parser)
        │
        ▼
Making Template for Post Approval
(JavaScript — per-platform HTML emails)
        │
        ▼
Loop Over Items
        │
        ▼
Send & Wait for Response (Gmail)
        │
   [User approves/rejects]
        │
        ▼
       If (approved = true?)
        │
        ▼
      Switch (by platform)
   ┌──────────┬────────┬──────┬───────┬──────┐
   ▼          ▼        ▼      ▼       ▼      ▼
LinkedIn Instagram Twitter Medium Facebook Slack
```

---

## Node Reference

### System 1 Nodes

| Node | Type | Purpose |
|---|---|---|
| Submission of Topic to post | Form Trigger | Collects user email and topic via a web form |
| Basic LLM Chain | LLM Chain | Prompts AI to find up to 3 credible, recent URLs on the topic |
| OpenRouter Chat Model | OpenRouter (Qwen3-235B) | Language model powering the URL research |
| Structured Output Parser | Output Parser | Enforces strict JSON schema on LLM URL output |
| Data Formatting | JavaScript Code | Unpacks the structured JSON into individual URL items |
| Keep record of the fetching URLs | Google Sheets | Appends topic, title, URL, source, and date to a tracking sheet |
| Code in JavaScript1 | JavaScript Code | Builds a rich HTML verification email with source cards and a "Verify All Sources" CTA button |
| Send a message | Gmail | Sends the verification email to the user |

### System 2 Nodes

| Node | Type | Purpose |
|---|---|---|
| Webhook | Webhook | Receives the approval click; captures `urls`, `topic`, and `email` from query params |
| Image generator HTTP Request | HTTP Request | Calls HuggingFace FLUX.1-schnell to generate a topic-relevant image |
| Upload file | Google Drive | Saves the generated image to the "Social Media Automation" Drive folder |
| Basic LLM Chain1 | LLM Chain | Visits each approved URL and writes a 400–500 word social-media-ready article |
| OpenRouter Chat Model1 | OpenRouter (Qwen3-235B) | Powers the article writing step |
| Message a model1 | Google Gemini 2.5 Flash | Reformats the article into tailored posts for 6 platforms as a JSON array |
| Make the chat model output appropriate | JavaScript Code | Cleans and parses Gemini's raw JSON output; appends user email as metadata |
| Merge | Merge | Combines the Drive image link with the parsed post content |
| Making Template for post Approval | JavaScript Code | Builds individual HTML approval emails per platform with image preview |
| Loop Over Items | Split in Batches | Iterates over each platform's post one at a time |
| Send message and wait for response | Gmail (Send & Wait) | Sends each platform's draft post to the user and waits for Approve/Reject |
| If | If Node | Checks if the user clicked Approve |
| Switch | Switch | Routes the approved post to the correct platform publisher |
| Create a post | LinkedIn | Publishes the approved post to LinkedIn |
| Publish | Instagram (community node) | Publishes a carousel post to Instagram |
| Create Tweet | Twitter (X) | Posts the approved tweet |
| Create a post1 | Medium | Publishes the article to Medium |
| Facebook Graph API | Facebook Graph API | Posts to Facebook |
| Message a model (disabled) | OpenAI GPT-4o-mini | Alternate post generator — currently disabled |

---

## AI Models Used

| Model | Provider | Used For |
|---|---|---|
| `qwen/qwen3-235b-a22b-thinking-2507` **or** `Perplexity` | OpenRouter | URL research (System 1) + article writing (System 2) — used alternately, only one active at a time in the OpenRouter node. **Perplexity is recommended** for best results due to its real-time web search capability |
| `models/gemini-2.5-flash` | Google Gemini | Multi-platform post generation (System 2) |
| `FLUX.1-schnell` | HuggingFace | AI image generation (System 2) |
| `gpt-4o-mini` | OpenAI | Alternate post generator (disabled) |

---

## Credentials Required

To import and run this workflow you will need to configure the following credentials in n8n:

- **OpenRouter** — for Qwen3-235B LLM calls and Perplexity-powered web search (`openRouterApi`)
- **Gmail OAuth2** — for sending verification and approval emails (`gmailOAuth2`)
- **Google Sheets OAuth2** — for URL tracking log (`googleSheetsOAuth2Api`)
- **Google Drive OAuth2** — for image storage (`googleDriveOAuth2Api`)
- **Google Gemini (PaLM) API** — for multi-platform post generation (`googlePalmApi`)
- **OpenAI API** — only needed if you re-enable the GPT-4o-mini node (`openAiApi`)
- **LinkedIn** — for auto-publishing to LinkedIn
- **Twitter (X)** — for auto-publishing tweets
- **Medium** — for auto-publishing articles
- **Facebook Graph API** — for auto-publishing to Facebook pages
- **HuggingFace** — API key embedded in the HTTP Request node header (replace with your own key before deploying)

> ⚠️ **Important:** The HuggingFace Bearer token is currently hardcoded in the "Image generator HTTP Request" node. Replace it with your own token or store it as an n8n credential before sharing or deploying this workflow.

---

## Google Sheets Tracking

A Google Sheet named **"Post Topic Meta data"** is used to log every URL fetched during research. Columns tracked:

- Topic
- Title
- URL
- Source
- Published Date

Sheet ID: *(configure your own Google Sheet and update the node)*

---

## Google Drive Image Storage

Generated images are stored in a Google Drive folder named **"Social Media Automation"**.

Folder ID: *(configure your own Google Drive folder and update the node)*

---

## How to Import & Run

1. Open your n8n instance.
2. Go to **Workflows → Import from File**.
3. Select `Social_Media_Automation_Swopnile.json`.
4. Set up all required credentials listed above.
5. Update the **ngrok URL** in the "Code in JavaScript1" node with your own webhook URL (or a production domain). Search for `ngrok-free.app` in the code.
6. Replace the HuggingFace API token in the "Image generator HTTP Request" node headers.
7. Activate the workflow.
8. Open the **Form Trigger URL** and submit a topic to start System 1.

---

## Human-in-the-Loop Approval Gates

This workflow has **two deliberate approval checkpoints** to keep humans in control:

1. **Source Verification Email** (end of System 1) — The user reviews all researched URLs and clicks "Verify All Sources" to proceed.
2. **Per-Platform Post Approval** (inside System 2 loop) — For each of the 6 platforms, the user receives a draft email and must click **Approve** before anything is published.

---

## Notes

The connection between the two systems is the **Webhook node** — its URL is embedded in the verification email's CTA button. When the user clicks "Verify All Sources," it fires the webhook and starts System 2.

---

## Demo / Screenshots

**Step 1 — Input Form**
*User submits their email and topic to kick off the workflow.*

![Input Form](./Input_Form.png)

---

**Step 2 — System 1: Research & Source Verification Workflow**
*n8n canvas showing the full research and email verification pipeline.*

![Research & Source Verification Workflow](./Research___Source_Verification.png)

---

**Step 3 — Source Verification Email**
*The user receives a formatted email listing the AI-researched sources to verify.*

![Verify Email](./Verify_Email.png)

---

**Step 4 — System 2: Content Generation & Automation Workflow**
*n8n canvas showing the full content generation, approval, and publishing pipeline.*

![Social Media Content Generation and Automation](./Social_Media_Content_Generation_and_Automation.png)

---

**Step 5 — LinkedIn Post Approval**
*Draft LinkedIn post sent to the user for approval before publishing.*

![LinkedIn Post](./LinekedIn_Post.png)

---

**Step 6 — Instagram Post Approval**
*Draft Instagram post with AI-generated image, ready for approval.*

![Instagram Post](./Instagram__Post.png)

---

**Step 7 — Twitter (X) Post Approval**
*Concise tweet draft with hashtags, sent for approval.*

![Twitter Post](./Twitter__Post.png)

---

**Step 8 — Medium Post Approval**
*Long-form Medium article draft sent for approval.*

![Medium Post](./Medium__Post.png)

---

**Step 9 — Facebook Post Approval**
*Facebook post draft with CTA and hashtags, sent for approval.*

![Facebook Post](./Facebook__Post.png)

---

**Step 10 — Slack Post Approval**
*Internal team Slack update draft, sent for approval.*

![Slack Post](./Slack_Post.png)

---

## Author

Built by **Aciful Islam Khan** — an AI-powered social media automation pipeline for research-backed, human-approved multi-platform content publishing.
