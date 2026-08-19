# 🚀 Installation Guide

This guide walks you through setting up **Instagram BiteRush Chatbot** from a blank n8n instance to a fully running Instagram DM automation.

> 📖 New to the project? Start with the [main README](../README.md) for an architecture overview before following the steps below.

---

## 1. Prerequisites Checklist

Before you begin, make sure you have the following ready. Each of these is required — the workflow will not run correctly with anything missing.

| # | Requirement | Where to get it |
|---|---|---|
| 1 | An **n8n instance** (Cloud or self-hosted) with LangChain community nodes enabled | [n8n.io](https://n8n.io) |
| 2 | A **Meta Developer App** with the Instagram Messaging API product added | [developers.facebook.com](https://developers.facebook.com) |
| 3 | An **Instagram Professional/Business account** linked to a Facebook Page | Meta Business Suite |
| 4 | An **OpenAI API key** | [platform.openai.com](https://platform.openai.com) |
| 5 | A **Google Gemini API key** | [Google AI Studio](https://aistudio.google.com) |
| 6 | A **Pinecone account** with a vector index created and populated | [pinecone.io](https://www.pinecone.io) |
| 7 | A **Google Sheets** spreadsheet + Google Cloud OAuth2 credentials | Google Cloud Console |
| 8 | A **long-lived Instagram access token** with `instagram_manage_messages` permission | Meta Developer App dashboard |

📎 For full configuration details on each of the above, see [configuration.md](./configuration.md).

---

## 2. Get the Workflow File

Clone this repository, or simply download the workflow JSON directly:

```bash
git clone https://github.com/<your-username>/instagram-biterush-chatbot.git
cd instagram-biterush-chatbot
```

The importable workflow file is:

```
Instagram_BIteRush_Chatbot.json
```

---

## 3. Import the Workflow into n8n

1. Open your n8n instance.
2. Go to **Workflows → Import from File** (or **⋯ menu → Import from File** on n8n Cloud).
3. Select `Instagram_BIteRush_Chatbot.json` from the cloned repo.
4. The full canvas — **Recieve message → Switch → echo check → Ai RAG BOT → Edit required data → send message** — will load, along with the connected model/tool/memory nodes.

> ⚠️ At this point the workflow is **inactive** and none of the credentials are connected yet — that's expected. Continue to the next step.

---

## 4. Connect Credentials to Each Node

Open each of the following nodes and attach (or create) the matching credential:

| Node | Credential to attach |
|---|---|
| OpenAI Chat Model | OpenAI API Key |
| Embeddings Google Gemini | Google Gemini API Key |
| Pinecone Vector Store | Pinecone API Key |
| Add data to google sheet | Google Sheets OAuth2 |
| send message | Instagram Access Token (Bearer Auth) |

Detailed field-by-field setup for each credential is covered in [configuration.md](./configuration.md).

---

## 5. Update Business-Specific Values

Before activating, edit these node fields to match your own business:

- **Pinecone Vector Store** → set your Pinecone **Index Name**.
- **Add data to google sheet** → set your target **Spreadsheet ID** and sheet/tab name.
- **send message** → set your Instagram **Page/Business Account ID** in the request URL.
- **Ai RAG BOT** → update the system prompt with your brand's tone, menu, FAQs, and policies.
- **Recieve message / Respond to Webhook** → set your **Meta Verify Token** to match what you'll enter in the Meta App dashboard.

---

## 6. Populate Your Pinecone Knowledge Base

The Ai RAG BOT can only answer questions grounded in what's stored in Pinecone. Before going live:

1. Prepare your knowledge documents (menu, FAQs, hours, delivery policy, etc.) as plain text or structured chunks.
2. Generate embeddings using the **same embedding model** referenced in the workflow (Google Gemini Embeddings).
3. Upsert those embeddings into your Pinecone index under the index name configured in step 5.

---

## 7. Register the Webhook with Meta

1. Copy the **Production Webhook URL** shown on the **Recieve message** node.
2. In your Meta Developer App dashboard, go to **Instagram → Configuration → Webhooks**.
3. Paste the webhook URL and enter the same **Verify Token** you configured in step 5.
4. Subscribe to the **`messages`** field.
5. Meta will send a `GET` verification request — the **Respond to Webhook** node handles this automatically and completes the handshake.

---

## 8. Activate the Workflow

1. In n8n, toggle the workflow to **Active** (top-right switch).
2. Send a test Direct Message to your connected Instagram account.
3. Confirm:
   - The message reaches the **Recieve message** node (check the **Executions** tab).
   - The **Ai RAG BOT** produces a grounded reply.
   - The reply is delivered back into the Instagram DM thread via **send message**.
   - If you shared contact details in your test message, confirm a new row appears via **Add data to google sheet**.

---

## 9. Verify End-to-End

Run through this quick smoke test after activation:

- [ ] Webhook verification succeeded (no errors in Meta App dashboard).
- [ ] A real Instagram DM triggers an execution in n8n.
- [ ] The AI reply is relevant and grounded in your Pinecone knowledge base (not generic/hallucinated).
- [ ] Conversation context is retained across multiple messages (memory working).
- [ ] Lead data appears correctly in Google Sheets when shared.
- [ ] The reply appears in the customer's actual Instagram inbox.

If any step fails, double check the credential mapping in [configuration.md](./configuration.md) and review execution logs in n8n's **Executions** tab.

---

## Next Steps

- 🔧 [Configuration Guide](./configuration.md) — detailed credential and environment setup.
- 🗺️ [Workflow Architecture](./workflow-architecture.md) — full technical breakdown of every node.