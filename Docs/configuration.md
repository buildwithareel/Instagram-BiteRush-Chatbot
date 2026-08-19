# 🔑 Configuration Guide

This document covers every credential, API key, and configurable field needed to run **Instagram BiteRush Chatbot**. Use it alongside [installation.md](./installation.md) when setting up the workflow for the first time, or whenever you need to rotate a key or migrate to a new environment.

> ℹ️ n8n stores credentials in its own encrypted **Credentials Manager** — this workflow does not use a `.env` file. Every credential below should be created inside n8n (**Credentials → Add Credential**) and then attached to the relevant node.

---

## 1. Credential Reference Table

| Credential | Type in n8n | Used By Node | Description |
|---|---|---|---|
| OpenAI API Key | `OpenAI API` | OpenAI Chat Model | Authenticates chat completion requests that power the Ai RAG BOT's reasoning. |
| Google Gemini API Key | `Google Gemini (PaLM) API` | Embeddings Google Gemini | Authenticates embedding generation for RAG lookups. |
| Pinecone API Key | `Pinecone API` | Pinecone Vector Store | Grants access to your Pinecone project. |
| Pinecone Index Name | *(node field, not a credential)* | Pinecone Vector Store | Name of the index holding your knowledge base (e.g. `bite-rush-rag`). |
| Google Sheets OAuth2 | `Google Sheets OAuth2 API` | Add data to google sheet | Grants write access to your lead-logging spreadsheet. |
| Google Sheet Document ID | *(node field, not a credential)* | Add data to google sheet | The specific spreadsheet where leads are appended. |
| Instagram Access Token | `HTTP Bearer Auth` | send message | Authenticates outbound calls to the Instagram Graph API `/messages` endpoint. |
| Meta Verify Token | *(node field, not a credential)* | Recieve message / Respond to Webhook | Shared secret used during Meta's webhook subscription handshake. |
| Instagram Business/Page ID | *(node field, not a credential)* | send message | Your Instagram professional account ID, used in the outgoing request URL. |

---

## 2. OpenAI Configuration

**Used by:** `OpenAI Chat Model`

1. Generate an API key at [platform.openai.com/api-keys](https://platform.openai.com/api-keys).
2. In n8n: **Credentials → Add Credential → OpenAI API**.
3. Paste the key and save.
4. Open the **OpenAI Chat Model** node and select the credential.
5. Set your preferred model (e.g. `gpt-4o` / `gpt-4o-mini`) and temperature depending on how deterministic you want customer replies to be.

---

## 3. Google Gemini Configuration

**Used by:** `Embeddings Google Gemini`

1. Generate an API key at [Google AI Studio](https://aistudio.google.com/app/apikey).
2. In n8n: **Credentials → Add Credential → Google Gemini (PaLM) API**.
3. Paste the key and save.
4. Open the **Embeddings Google Gemini** node and select the credential.

> ⚠️ The embedding model used here **must match** whatever model you used to embed your documents into Pinecone — mismatched embedding dimensions will cause vector search to fail.

---

## 4. Pinecone Configuration

**Used by:** `Pinecone Vector Store`

1. Create a Pinecone account and project at [pinecone.io](https://www.pinecone.io).
2. Create an index with a **dimension size matching your embedding model's output** (check Google Gemini embedding docs for the exact number).
3. In n8n: **Credentials → Add Credential → Pinecone API**, paste your API key.
4. Open the **Pinecone Vector Store** node:
   - Select the Pinecone credential.
   - Set **Index Name** to match your created index.
   - Confirm it's set to operate in **retrieve-as-tool** mode so the Ai RAG BOT can call it dynamically.
5. Populate the index with your knowledge base documents (see [installation.md](./installation.md#6-populate-your-pinecone-knowledge-base)).

---

## 5. Google Sheets Configuration

**Used by:** `Add data to google sheet`

1. In Google Cloud Console, enable the **Google Sheets API** for your project.
2. Create an **OAuth2 Client ID** (Web application type) and note the Client ID/Secret.
3. In n8n: **Credentials → Add Credential → Google Sheets OAuth2 API**, complete the OAuth consent flow.
4. Create a Google Sheet with a header row for the fields you want to capture, e.g.:

   | Name | Phone Number | Message | Timestamp |
   |---|---|---|---|

5. Open the **Add data to google sheet** node:
   - Select the credential.
   - Choose your spreadsheet by **Document ID** or from the file picker.
   - Select the correct sheet/tab.
   - Map each AI-extracted field (name, phone, etc.) to the matching column.

---

## 6. Meta / Instagram Configuration

**Used by:** `Recieve message`, `Respond to Webhook`, `send message`

### 6.1 Create the Meta App
1. Go to [developers.facebook.com](https://developers.facebook.com) → **My Apps → Create App**.
2. Choose the **Business** app type.
3. Add the **Instagram** product to the app.

### 6.2 Link Your Instagram Account
1. Connect your Instagram **Professional/Business account** to a Facebook Page.
2. Under **Instagram → API Setup**, link that Page to your app.

### 6.3 Generate an Access Token
1. Generate a **long-lived access token** with at least the `instagram_manage_messages` and `pages_messaging` permissions.
2. In n8n: **Credentials → Add Credential → HTTP Bearer Auth**, paste the token.
3. Attach this credential to the **send message** node.

### 6.4 Set the Verify Token
1. Choose any secret string (e.g. `biterush_verify_2026`) — this is your **Meta Verify Token**.
2. Set the same value inside the **Respond to Webhook** node's response logic.
3. You'll re-enter this exact value in the Meta dashboard in the next step.

### 6.5 Register the Webhook
1. Copy the **Production Webhook URL** from the **Recieve message** node in n8n.
2. In the Meta App dashboard: **Instagram → Configuration → Webhooks**.
3. Paste the URL, enter your Verify Token, and click **Verify and Save**.
4. Subscribe to the **`messages`** webhook field.

### 6.6 Set the Instagram Business/Page ID
1. Find your Instagram Business Account ID under **Instagram → API Setup** in the Meta dashboard.
2. Open the **send message** node and set this ID in the outgoing request URL (`/{ig-business-id}/messages`).

---

## 7. Node Field Reference

Quick reference for non-credential fields you'll need to fill in manually on the canvas:

| Node | Field | Example Value |
|---|---|---|
| Pinecone Vector Store | Index Name | `bite-rush-rag` |
| Add data to google sheet | Document ID | Your Google Sheet ID |
| Add data to google sheet | Sheet Name | `Leads` |
| send message | Instagram Business Account ID | `1789xxxxxxxxxx` |
| Recieve message / Respond to Webhook | Verify Token | `biterush_verify_2026` |
| Ai RAG BOT | System Prompt | Your brand tone, menu, and policies |

---

## 8. Rotating & Securing Credentials

- Rotate your **Instagram Access Token** before it expires — long-lived tokens typically last ~60 days unless refreshed.
- Never hard-code API keys directly into node parameters — always use n8n's Credentials Manager so secrets stay encrypted and out of the exported JSON.
- If you fork or share this workflow's JSON file, double-check that no credential values were accidentally exported (n8n excludes credentials by default, but always verify).

---

## Next Steps

- 🚀 [Installation Guide](./installation.md) — full setup walkthrough from import to activation.
- 🗺️ [Workflow Architecture](./workflow-architecture.md) — technical breakdown of every node and connection.