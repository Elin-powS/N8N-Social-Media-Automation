# N8N-Social-Media-Automation
AI-powered n8n social media automation system that generates, verifies, scrapes, designs, and publishes multi-platform content using OpenRouter, Gemini, HuggingFace FLUX, and email-based approval workflows.
# 🚀 AI Social Media Automation with n8n

An end-to-end AI-powered social media automation workflow built using **n8n**, integrating LLM research, AI scraping, image generation, email verification loops, and multi-platform publishing.

This system automates the full content pipeline — from topic submission to AI-generated posts and final publishing across platforms like Facebook, Instagram, LinkedIn, Medium, and X.

---

## 🧠 Overview

This project builds a fully automated AI content production system that includes:

- Topic intake via form
- AI-powered research
- URL verification through email
- AI scraping & summarization
- AI post generation
- AI image generation
- Admin approval loop
- Multi-platform publishing

---

## 🔄 Workflow Architecture

### 1️⃣ Topic Submission

User submits:
- 📧 Email address
- 📝 Topic for content

---

### 2️⃣ AI Research (OpenRouter LLM)

- Searches:
  - Articles
  - Blogs
  - Vlogs
  - Web resources
- Returns high-quality relevant URLs

---

### 3️⃣ Data Formatting (Code Node)

- JavaScript code node:
  - Cleans and formats URLs
  - Structures metadata
  - Prepares data for verification

---

### 4️⃣ Email Verification (Gmail + Webhook)

- Custom Gmail template generated
- Includes:
  - URL list
  - Verification button
- When clicked:
  - Webhook triggers
  - Workflow continues
  - Metadata is passed forward

---

### 5️⃣ AI Scraping + Post Generation

After verification:

- OpenRouter model scrapes and summarizes link content
- Gemini model generates:
  - Engaging social media post
  - Optimized captions
  - Structured formatting

---

### 6️⃣ AI Image Generation (Parallel Process)

- HTTPS request sent to HuggingFace:
  
  `black-forest-labs/FLUX.1-schnell`

- Generates high-quality AI image
- Image stored in storage system

---

### 7️⃣ Final Admin Approval Loop

- Post + Image merged
- Final email sent to admin
- Includes:
  - Post preview
  - Image preview
  - Approve / Decline buttons

Logic:
- If approved → Publish
- If declined → Loop back for regeneration

---

### 8️⃣ Multi-Platform Publishing

Upon approval, content is automatically published to:

- Facebook
- Instagram
- LinkedIn
- Medium
- X (Twitter)

---

## 🏗️ Tech Stack

- n8n (Automation Engine)
- OpenRouter (LLM Research & Scraping)
- Google Gemini (Post Generation)
- HuggingFace FLUX.1-schnell (AI Image Generation)
- Gmail API (Email Automation)
- Webhooks (Trigger-Based Workflow)
- JavaScript (Custom Code Nodes)

---

## 🔐 Security Notice

All credentials (API keys, tokens, OAuth credentials) are stored in environment variables and are **not included in this repository**.

---

## 📊 High-Level Workflow


```text
Form Submission
        ↓
OpenRouter Research
        ↓
Format URLs
        ↓
Email Verification
        ↓
Webhook Trigger
        ↓
Scrape + Summarize
        ↓
Gemini Post Generation
        ↓
Image Generation (FLUX)
        ↓
Merge Content
        ↓
Admin Approval Loop
        ↓
Multi-Platform Publishing
```
---

## 📌 Key Features

✔ AI-powered research  
✔ Human verification checkpoint  
✔ Automated scraping  
✔ AI-generated posts  
✔ AI-generated images  
✔ Admin approval system  
✔ Multi-platform publishing  
✔ Modular & scalable architecture  

---

## 🚀 Use Cases

- Marketing agencies
- AI automation systems
- Personal branding automation
- SaaS growth systems
- Content distribution pipelines

---

## 📈 Future Improvements

- Hashtag optimization
- Analytics feedback loop
- A/B testing system
- Post scheduling system
- CRM integration
- Performance tracking dashboard

---

## 👨‍💻 Author

Aciful Islam Khan  
AI/ML Engineer | Automation Architect | Computer Vision Specialist  

---

## ⭐ Support

If you find this project useful, consider giving it a ⭐ on GitHub.

