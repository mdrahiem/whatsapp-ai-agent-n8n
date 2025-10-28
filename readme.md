# Complete Documentation: WhatsApp AI Assistant with Google Drive \& Chat History Integration

> [!TIP]
> **🚀 You can download the workflow directly from above and import it into your n8n instance to get started quickly! 🚀**


## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Part 1: Install Node.js and n8n](#part-1-install-nodejs-and-n8n)
5. [Part 2: Set Up Google Drive Integration](#part-2-set-up-google-drive-integration)
6. [Part 3: Install Evolution API](#part-3-install-evolution-api)
7. [Part 4: Connect WhatsApp to Evolution API](#part-4-connect-whatsapp-to-evolution-api)
8. [Part 5: Create Main AI Agent Workflow](#part-5-create-main-ai-agent-workflow)
9. [Part 6: Create WhatsApp Message Fetcher Sub-Workflow](#part-6-create-whatsapp-message-fetcher-sub-workflow)
10. [Part 7: Configure AI Agent Tools](#part-7-configure-ai-agent-tools)
11. [Part 8: Configure Evolution API Webhook](#part-8-configure-evolution-api-webhook)
12. [Part 9: Testing \& Troubleshooting](#part-9-testing-troubleshooting)
13. [Common Issues \& Solutions](#common-issues-solutions)
14. [Best Practices](#best-practices)
15. [Advanced Customizations](#advanced-customizations)
16. [Quick Reference Commands](#quick-reference-commands)

***

## Overview

This comprehensive guide walks you through building an intelligent WhatsApp AI Assistant that can:

✅ Receive WhatsApp messages automatically
✅ Process them using an AI Agent (Gemini/ChatGPT)
✅ Access your Google Drive files and documents
✅ Retrieve WhatsApp chat history and past conversations
✅ Search for specific contacts and messages
✅ Send intelligent, context-aware responses back to WhatsApp

### Technology Stack

- **n8n**: Workflow automation platform (self-hosted)
- **Evolution API**: Open-source WhatsApp integration (Node.js)
- **Google Gemini/ChatGPT**: AI/LLM for intelligent processing
- **Google Drive API**: Access and search files
- **Evolution API Chat API**: Access WhatsApp message history

***

## Architecture

```
WhatsApp Message
      ↓
Evolution API (receives message)
      ↓
n8n Webhook (triggers main workflow)
      ↓
Extract phone number & message text
      ↓
AI Agent (Gemini/ChatGPT LLM)
  ├─→ Google Drive Tool (search files)
  ├─→ Get WhatsApp Chats Tool (list contacts)
  └─→ Get Messages from Contact Tool (sub-workflow)
      ↓
Extract AI Response
      ↓
HTTP Request to Evolution API
      ↓
WhatsApp Response (sent back to user)
```

**Sub-Workflow Architecture (Message Fetcher):**

```
n8n Webhook → Call Evolution API → Format Messages → Respond to Webhook
```


***

## Prerequisites

### Required Software

- **Operating System**: Linux, macOS, or WSL on Windows
- **Node.js**: v18+ or v20+ (installed via NVM)
- **npm**: Comes with Node.js (v10+)
- **Git**: For cloning repositories
- **curl**: For API testing


### Required Accounts

- **Google Account**: For Drive API access
- **Google Cloud Project**: For OAuth credentials
- **WhatsApp**: Mobile number for connection
- **Google API Key**: For AI model (Gemini recommended)


### System Requirements

- **RAM**: Minimum 2GB
- **Disk Space**: 500MB for n8n + Evolution API
- **Network**: Stable internet connection

***

## Part 1: Install Node.js and n8n

### Step 1.1: Install Node.js via NVM

```bash
# Download and install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash

# Reload shell configuration
source ~/.bashrc
# For zsh users: source ~/.zshrc

# Install Node.js 20 LTS
nvm install 20
nvm use 20

# Verify installation
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
```


### Step 1.2: Install n8n Globally

```bash
# Install n8n globally
npm install -g n8n

# Verify installation
n8n --version
```


### Step 1.3: Start n8n

```bash
# Start n8n server
n8n start

# n8n will start on http://localhost:5678
```

**Expected Output:**

```
n8n ready on http://localhost:5678
```


### Step 1.4: Access n8n Dashboard

1. Open browser: `http://localhost:5678`
2. Create your owner account (first-time setup)
3. You'll see the n8n dashboard

***

## Part 2: Set Up Google Drive Integration

### Step 2.1: Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click "Select a project" → "New Project"
3. **Project Name**: `n8n-whatsapp-assistant`
4. Click "Create"

### Step 2.2: Enable Google Drive API

1. In Google Cloud Console, go to "APIs \& Services" → "Library"
2. Search for "Google Drive API"
3. Click on it → Click "Enable"

### Step 2.3: Create OAuth Credentials

1. Go to "APIs \& Services" → "Credentials"
2. Click "Create Credentials" → "OAuth client ID"
3. **If prompted**, configure OAuth consent screen first:
    - User Type: **External**
    - App name: `n8n WhatsApp Assistant`
    - User support email: Your email
    - Developer contact: Your email
    - Scopes: Add `../auth/drive.readonly`
    - Test users: Add your Gmail address
    - Click "Save and Continue"
4. Back to "Create OAuth client ID":
    - Application type: **Web application**
    - Name: `n8n Google Drive`
    - Authorized JavaScript origins: `http://localhost:5678`
    - Authorized redirect URIs: `http://localhost:5678/rest/oauth2-credential/callback`
    - Click "Create"
5. **Copy and save:**
    - Client ID: `123456789-abc...googleusercontent.com`
    - Client Secret: `GOCSPX-xxx...`

### Step 2.4: Configure Google Drive in n8n

1. In n8n, go to "Credentials" (left sidebar)
2. Click "+ Add Credential"
3. Search for "Google Drive OAuth2 API"
4. Fill in:
    - **Credential Name**: `My Google Drive`
    - **Client ID**: (paste from Step 2.3)
    - **Client Secret**: (paste from Step 2.3)
5. Click "Connect my account"
6. Google OAuth popup will appear
7. Sign in with your Google account
8. Allow access to Google Drive
9. You'll be redirected back to n8n
10. Click "Save"

✅ **Google Drive is now connected!**

***

## Part 3: Install Evolution API

### Step 3.1: Verify Node.js Installation

```bash
# Should already have Node.js from Part 1
node --version  # v20.x.x
npm --version   # 10.x.x
```


### Step 3.2: Clone Evolution API Repository

```bash
# Navigate to home directory
cd ~

# Clone repository
git clone https://github.com/EvolutionAPI/evolution-api.git

# Enter directory
cd evolution-api

# Checkout main branch
git checkout main
```


### Step 3.3: Install Dependencies

```bash
# Inside evolution-api directory
npm install

# This will take 2-5 minutes
```


### Step 3.4: Configure Environment Variables

```bash
# Copy example env file
cp .env.example .env

# Open .env for editing
nano .env
# Or: vim .env
# Or: code .env (VS Code)
```

**Minimal .env configuration:**

```bash
# Server Configuration
SERVER_NAME=evolution
SERVER_TYPE=http
SERVER_PORT=8080
SERVER_URL=http://localhost:8080

# CORS Settings
CORS_ORIGIN=*
CORS_METHODS=POST,GET,PUT,DELETE
CORS_CREDENTIALS=true

# Authentication (IMPORTANT: Change this!)
AUTHENTICATION_API_KEY=d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202
# Generate your own: openssl rand -hex 32

# Database (disabled for simplicity)
DATABASE_ENABLED=false
DATABASE_PROVIDER=

# Storage
STORE_MESSAGES=true
STORE_MESSAGE_UP=true
STORE_CONTACTS=true

# Cache (disabled for simplicity)
CACHE_REDIS_ENABLED=false
CACHE_LOCAL_ENABLED=true

# Instance deletion (false = never delete)
DEL_INSTANCE=false

# Logging
LOG_LEVEL=ERROR,WARN,INFO,DEBUG
LOG_COLOR=true
LOG_BAILEYS=error

# Webhook
WEBHOOK_GLOBAL_ENABLED=true
WEBHOOK_GLOBAL_URL=
WEBHOOK_EVENTS_MESSAGES_UPSERT=true

# QR Code
QRCODE_LIMIT=60
QRCODE_COLOR=#175197

# Session
CONFIG_SESSION_PHONE_CLIENT=Evolution API
CONFIG_SESSION_PHONE_NAME=Chrome
```

**Save the file** (Ctrl+X → Y → Enter in nano)

### Step 3.5: Generate Prisma Client

```bash
# Still in evolution-api directory
npm run db:generate
```

**Expected output:**

```
✔ Generated Prisma Client
```

⚠️ **CAVEAT**: If you get an error about Prisma, make sure your `.env` has `DATABASE_ENABLED=false` or set up PostgreSQL properly.

### Step 3.6: Start Evolution API

```bash
# Start in development mode
npm run dev:server
```

**Expected output:**

```
[Evolution API] v2.3.6 - Server is running on port 8080
[Evolution API] Documentation: http://localhost:8080/docs
```

✅ **Evolution API is running!**

**Test it:**

```bash
# In a new terminal
curl http://localhost:8080

# Should return API information
```


### Step 3.7: Make Evolution API Run Permanently (Optional but Recommended)

```bash
# Install PM2 process manager
npm install -g pm2

# Start Evolution API with PM2
pm2 start npm --name "evolution-api" -- run start:prod

# Save PM2 configuration
pm2 save

# Setup auto-start on boot
pm2 startup
# Follow the command it outputs
```

**PM2 Commands:**

```bash
pm2 status                # Check status
pm2 logs evolution-api    # View logs
pm2 restart evolution-api # Restart
pm2 stop evolution-api    # Stop
pm2 delete evolution-api  # Remove from PM2
```


***

## Part 4: Connect WhatsApp to Evolution API

### Step 4.1: Create WhatsApp Instance

```bash
curl -X POST http://localhost:8080/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202" \
  -d '{
    "instanceName": "my-whatsapp",
    "qrcode": true,
    "integration": "WHATSAPP-BAILEYS"
  }'
```

**Response (save this):**

```json
{
  "instance": {
    "instanceName": "my-whatsapp",
    "instanceId": "0297ee50-...",
    "status": "connecting"
  },
  "hash": "4DDDB711-...",
  "qrcode": {
    "code": "2@5TuabPSqUvoEL...",
    "base64": "data:image/png;base64,iVBORw0KGgo..."
  }
}
```


### Step 4.2: Get QR Code

```bash
# Get fresh QR code
curl -X GET http://localhost:8080/instance/connect/my-whatsapp \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202" \
  | jq -r '.base64'
```

Copy the entire output (starts with `data:image/png;base64,...`)

### Step 4.3: View QR Code

**Method 1 (Easiest - Browser):**

1. Copy the base64 string from Step 4.2
2. Open a new browser tab
3. Paste the entire string in the address bar
4. Press Enter
5. QR code will display as an image

**Method 2 (Terminal with qrencode):**

```bash
# If you have qrencode installed
curl -s -X GET http://localhost:8080/instance/connect/my-whatsapp \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202" \
  | jq -r '.code' | qrencode -t ANSIUTF8
```

**Method 3 (Evolution API Manager Dashboard):**

- Visit `http://localhost:8080/manager`
- View the QR code directly

⚠️ **CAVEAT**: QR codes expire after 60 seconds (configurable in `.env`). If it expires, just run Step 4.2 again.

### Step 4.4: Scan QR Code with WhatsApp

1. Open WhatsApp on your phone
2. Go to **Settings** → **Linked Devices**
3. Tap "Link a Device"
4. Scan the QR code displayed
5. Wait 10 seconds for connection

### Step 4.5: Verify Connection

```bash
curl -X GET http://localhost:8080/instance/connectionState/my-whatsapp \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202"
```

**Expected response:**

```json
{
  "instance": {
    "instanceName": "my-whatsapp",
    "state": "open"
  }
}
```

✅ `state: "open"` means WhatsApp is connected!

⚠️ **CAVEAT**: If you see `state: "close"`, the QR scan failed or WhatsApp disconnected. Regenerate QR and try again.

### Step 4.6: Test Sending Message

```bash
# Send test message (replace with your phone number)
curl -X POST http://localhost:8080/message/sendText/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202" \
  -d '{
    "number": "31612345678",
    "text": "Test message from Evolution API! 🎉"
  }'
```

**Phone number format:**

- ❌ Wrong: `+31 6 1234 5678`
- ❌ Wrong: `+31612345678`
- ✅ **Correct**: `31612345678` (country code + number, no + or spaces)

Check your WhatsApp - you should receive the message!

***

## Part 5: Create Main AI Agent Workflow

This is your primary workflow that handles incoming WhatsApp messages and responds using AI with access to Google Drive and chat history.

### Workflow Structure

```
Webhook → Extract Message → AI Agent → Extract Response → Send WhatsApp Response
                                ↓
                          [AI Agent Tools]
                        - Google Drive Search
                        - Get WhatsApp Chats
                        - Get Messages from Contact
```


### Step 5.1: Create New Workflow

1. In n8n, click "+ Add workflow"
2. **Name it**: `WhatsApp AI Chatbot`
3. Click "Save"

### Step 5.2: Add Webhook Trigger Node

1. Search for "Webhook" node
2. Drag it to canvas
3. Configure:
    - **HTTP Method**: POST
    - **Path**: `whatsapp-ai-bot` (or any name)
    - **Authentication**: None
    - **Respond**: Using 'Respond to Webhook' Node
4. Copy the **Production Webhook URL** (we'll use this in Part 8)
    - Example: `http://localhost:5678/webhook/whatsapp-ai-bot`

### Step 5.3: Add "Extract Message from WhatsApp" Node

Add a **Code** node after Webhook:

**Node name**: `Extract message from Whatsapp`

**Code:**

```javascript
// Extract relevant data from WhatsApp webhook
const webhookData = $input.first().json;

// Get message data
const messageData = webhookData.data;
const phoneNumber = messageData.key.remoteJid;
const messageText = messageData.message?.conversation || 
                   messageData.message?.extendedTextMessage?.text || 
                   "";

// Only process if it's not from us and has text
if (messageData.key.fromMe || !messageText) {
  return [];
}

return {
  json: {
    phoneNumber: phoneNumber,
    messageText: messageText,
    messageId: messageData.key.id
  }
};
```

**Alternative using Set Node** (simpler approach):

Add a **Set** node instead:

- **Keep Only Set**: Enable
- **Values to Set**:
    - Name: `receiver_number`
Value: `{{ $json.data.key.remoteJid.split('@')[^0] }}`
    - Name: `message_text`
Value: `{{ $json.data.message.conversation || $json.data.message.extendedTextMessage?.text }}`


### Step 5.4: Add AI Agent Node

1. Add **AI Agent** node after Extract node
2. Configure:

**Basic Settings:**

- **Agent Type**: Tools Agent / ReAct Agent
- **Prompt Type**: Define below
- **Chat Model**: Select **Google Gemini Chat Model** (or ChatGPT)
- **Credential**: Add your Gemini API key

**System Message:**

```text
You are a helpful AI assistant with access to:
1. Google Drive: Search and retrieve files from a specific folder
2. WhatsApp Chat History: Access past conversations with contacts

Tools Available:
- Search files and folders in Google Drive: Use when user asks about files, documents, or Drive content
- get_whatsapp_chats: Get list of ALL WhatsApp contacts and groups with their IDs
- get_messages_from_contact: Fetch messages from a specific contact (requires remoteJid from get_whatsapp_chats)

WhatsApp Message Retrieval Flow:
When user asks about messages or conversations:
1. First call get_whatsapp_chats to see all contacts
2. Identify the contact's "remoteJid" or "id" from the response (looks like "31654207351@s.whatsapp.net")
3. Call get_messages_from_contact with that EXACT remoteJid value

Example:
User: "What did I speak to Sai?"
Step 1: Call get_whatsapp_chats
Response shows: {"id": "31654207351@s.whatsapp.net", "pushName": "Sai Kiran"}
Step 2: Call get_messages_from_contact with remoteJid="31654207351@s.whatsapp.net"

IMPORTANT: 
- The remoteJid must include @s.whatsapp.net or @g.us suffix!
- Always check chat history first before saying you don't know
- Format responses for WhatsApp (short paragraphs, clear formatting)
- Be helpful, friendly, and concise

User Message: {{ $json.messageText }}
```

**Or use this simpler prompt if using Set node:**

```text
You are a helpful AI assistant with access to:
- Google Drive: Search and retrieve files from user's Google Drive
- WhatsApp Chat History: Access past conversations

When users ask about files:
1. Use the Google Drive tool to search
2. Provide clear, concise responses
3. Format responses for WhatsApp (short paragraphs)

When users ask about conversations:
1. First use get_whatsapp_chats to find the contact
2. Then use get_messages_from_contact with the exact remoteJid

Always be helpful and accurate.

User Message: {{ $('Extract message from Whatsapp').item.json.message_text }}
```


### Step 5.5: Add "Extract Data from Agent" Node

Add a **Code** node after AI Agent:

**Node name**: `Extract data from agent`

**Code:**

```javascript
const response = $input.first().json;

return {
  json: {
    aiResponse: response.output || response.text || "I couldn't process that request.",
    phoneNumber: $('Extract message from Whatsapp').first().json.phoneNumber
  }
};
```

**Alternative using Set Node:**

- **Values to Set**:
    - Name: `receiver_number`
Value: `{{ $('Extract message from Whatsapp').item.json.receiver_number }}`
    - Name: `agent_response`
Value: `{{ $('AI Agent').item.json.output }}`


### Step 5.6: Add "Send WhatsApp Response" HTTP Request Node

Add **HTTP Request** node:

**Node name**: `Send WhatsApp Response`

**Configuration:**

**Basic Settings:**

- **Method**: POST
- **URL**: `http://localhost:8080/message/sendText/my-whatsapp`

**Authentication:**

- **Authentication**: Generic Credential Type → **Header Auth**
- **Credential**: Create new → "Evolution API Auth"
    - **Name**: `apikey` ⚠️ **IMPORTANT**: all lowercase, no underscore
    - **Value**: `d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202`

**Body:**

- **Send Body**: ✅ Enable
- **Specify Body**: **"Using Fields Below"** (recommended)
- **Fields**:
    - Field 1:
        - **Name**: `number`
        - **Value**: `{{ $('Extract data from agent').item.json.phoneNumber }}` (if using Code node)
OR `{{ $('Extract data from agent').item.json.receiver_number }}` (if using Set node)
    - Field 2:
        - **Name**: `text`
        - **Value**: `{{ $('Extract data from agent').item.json.aiResponse }}` (if using Code node)
OR `{{ $('Extract data from agent').item.json.agent_response }}` (if using Set node)

⚠️ **CAVEAT**: Don't use "Using JSON" with n8n expressions - it often creates invalid JSON. Use "Using Fields Below" instead.

⚠️ **CAVEAT**: Header Auth credential MUST have header name as `apikey` (lowercase, no underscore). Not `api_key`, not `API_KEY`.

### Step 5.7: Save Workflow

Click "Save" (top-right). The workflow is saved but not active yet.

***

## Part 6: Create WhatsApp Message Fetcher Sub-Workflow

This sub-workflow fetches messages from a specific WhatsApp contact dynamically and is called by the AI Agent as a tool.

### Workflow Structure

```
Webhook → Call Evolution API → Format Response → Respond to Webhook
```


### Step 6.1: Create New Workflow

1. Create a new workflow
2. **Name it**: `WhatsApp Message Fetcher`
3. Click "Save"

### Step 6.2: Add Webhook Node

1. Add **Webhook** node
2. Configure:
    - **HTTP Method**: POST
    - **Path**: `fetch-whatsapp-messages`
    - **Respond**: Using 'Respond to Webhook' Node
3. Copy the **Production URL** (e.g., `http://localhost:5678/webhook/fetch-whatsapp-messages`)

### Step 6.3: Add "Call Evolution API" HTTP Request Node

Add **HTTP Request** node:

**Configuration:**

- **Method**: POST
- **URL**: `http://localhost:8080/chat/findMessages/my-whatsapp`
- **Authentication**: Evolution API Auth (Header Auth credential)
- **Send Body**: ✅ Enabled
- **Body Content Type**: JSON
- **Specify Body**: Using JSON

**JSON Body:**

```json
{
  "where": {
    "key": {
      "remoteJid": "{{ $json.body.remoteJid }}"
    }
  },
  "limit": 50
}
```

⚠️ **Note**: Use `{{ $json.body.remoteJid }}` WITHOUT the `=` prefix in JSON mode.

### Step 6.4: Add "Format Response" Code Node

Add **Code** node:

**Code:**

```javascript
const response = $input.first().json;

// The response is wrapped in "messages" object
const messagesObj = response.messages || response;
const records = messagesObj.records || [];

if (records.length === 0) {
  return [{ json: { response: "No messages found" } }];
}

// Format the messages
const messages = records.slice(0, 10).map(msg => {
  const from = msg.key.fromMe ? "You" : (msg.pushName || "Contact");
  
  // Extract text from different message types
  let text = "[Media]";
  if (msg.message?.conversation) {
    text = msg.message.conversation;
  } else if (msg.message?.extendedTextMessage?.text) {
    text = msg.message.extendedTextMessage.text;
  } else if (msg.messageType === "reactionMessage") {
    text = `[Reacted with ${msg.message.reactionMessage?.text || "emoji"}]`;
  } else if (msg.messageType === "audioMessage") {
    text = "[Voice message]";
  } else if (msg.messageType === "imageMessage") {
    text = "[Image]";
  }
  
  const timestamp = msg.messageTimestamp 
    ? new Date(msg.messageTimestamp * 1000).toLocaleString() 
    : "Unknown time";
    
  return `${from} (${timestamp}): ${text}`;
}).join('\n\n');

// Return formatted response
return [{ json: { response: messages } }];
```


### Step 6.5: Add "Respond to Webhook" Node

1. Add **Respond to Webhook** node
2. Configure:
    - **Respond With**: First Incoming Item

Done! This will return the formatted messages back to the AI Agent.

### Step 6.6: Activate Sub-Workflow

Toggle the workflow to **Active** (top-right corner).

### Step 6.7: Copy Production Webhook URL

From the Webhook node, copy the **Production URL**:

```
http://localhost:5678/webhook/fetch-whatsapp-messages
```

You'll need this URL for configuring AI Agent tools in the main workflow.

***

## Part 7: Configure AI Agent Tools

Now connect all three tools to your AI Agent in the main workflow.

### Tool 1: Google Drive Search

1. In the **AI Agent** node in your main workflow, click **Tools**
2. Click **"Add Tool"**
3. Select **"Search files and folders in Google Drive"** tool

**Configure:**

- **Resource**: File/Folder
- **Operation**: Search
- **Google Drive Credential**: Select your connected account
- **Drive ID**: Your Drive ID (usually your email or "My Drive")
- **Folder ID**: Specific folder ID (or leave empty for all files)
- **Search Query**: Let AI Agent decide dynamically (leave empty or use `{{ $json.query }}`)

**Tool Description:**

```text
Searches for files in Google Drive. Use when user asks about files, documents, or Drive content. Returns file names, IDs, and links.
```

⚠️ **CAVEAT**: n8n doesn't have a "List files" option in Google Drive tool. Use "Search" with empty query or specific folder ID instead.

### Tool 2: Get WhatsApp Chats

1. Click **"Add Tool"** again
2. Select **HTTP Request** tool

**Configure:**

**Basic Settings:**

- **Method**: POST
- **URL**: `http://localhost:8080/chat/findChats/my-whatsapp`

**Authentication:**

- **Authentication**: Evolution API Auth (Header Auth credential)

**Body:**

- **Send Body**: ✅ Enable
- **Body Content Type**: JSON
- **Specify Body**: Using JSON
- **JSON**: `{}`

**Tool Settings:**

- **Tool Name**: `get_whatsapp_chats`

**Tool Description:**

```text
Gets list of ALL WhatsApp contacts and groups with their IDs (remoteJid). 
Use this FIRST when user asks about messages or conversations to find the contact's remoteJid.

Returns: Array of contacts with fields like "id" or "remoteJid" (format: "31654207351@s.whatsapp.net") and "pushName" (contact name).

Example response:
[
  {"id": "31654207351@s.whatsapp.net", "pushName": "John Doe"},
  {"id": "31698765432@s.whatsapp.net", "pushName": "Jane Smith"}
]
```


### Tool 3: Get Messages from Contact (Sub-Workflow Call)

1. Click **"Add Tool"** again
2. Select **HTTP Request** tool

**Configure:**

**Basic Settings:**

- **Method**: POST
- **URL**: `http://localhost:5678/webhook/fetch-whatsapp-messages` (your sub-workflow URL from Part 6.7)

**Authentication:**

- **Authentication**: None

**Body:**

- **Send Body**: ✅ Enable
- **Body Content Type**: JSON
- **Specify Body**: Using Fields Below
- **Fields**:
    - Field 1:
        - **Name**: `remoteJid`
        - **Value**: (leave empty - AI will fill this dynamically)

**Tool Settings:**

- **Tool Name**: `get_messages_from_contact`

**Tool Description:**

```text
Fetches WhatsApp messages from a specific contact using their remoteJid.

IMPORTANT: You must call get_whatsapp_chats FIRST to get the contact's remoteJid!

Input format: {"remoteJid": "31654207351@s.whatsapp.net"}
The remoteJid MUST be the exact value from get_whatsapp_chats response, including @s.whatsapp.net or @g.us suffix.

Returns: Last 10 messages from that contact with sender names and timestamps.

Example flow:
1. User asks: "What did I speak to Sai Kiran?"
2. Call get_whatsapp_chats to get all contacts
3. Find Sai Kiran's remoteJid: "31654207351@s.whatsapp.net"
4. Call this tool with that exact remoteJid
5. Receive formatted message history
```

**Save the main workflow** after adding all tools.

***

## Part 8: Configure Evolution API Webhook

Now tell Evolution API to send incoming WhatsApp messages to your n8n main workflow.

### Step 8.1: Get n8n Production Webhook URL

1. In your main workflow (`WhatsApp AI Chatbot`), click on the **Webhook** node
2. Look at the "Webhook URLs" section
3. Copy the **Production URL** (not Test URL)
4. Example: `http://localhost:5678/webhook/whatsapp-ai-bot`

⚠️ **CAVEAT**: Always use Production URL for Evolution API webhook, not Test URL. Test URL only works when manually clicking "Execute workflow".

### Step 8.2: Configure Webhook in Evolution API

```bash
curl --request POST \
  --url http://localhost:8080/webhook/set/my-whatsapp \
  --header 'Content-Type: application/json' \
  --header 'apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202' \
  --data '{
    "webhook": {
      "enabled": true,
      "url": "http://localhost:5678/webhook/whatsapp-ai-bot",
      "webhookByEvents": false,
      "events": ["MESSAGES_UPSERT"]
    }
  }'
```

**Replace** `whatsapp-ai-bot` with your actual webhook path.

⚠️ **CAVEAT**: The JSON structure must have a root `"webhook"` object. Don't put fields directly at root level or you'll get a 400 error.

**Verify webhook is set:**

```bash
curl -X GET http://localhost:8080/webhook/find/my-whatsapp \
  -H "apikey: d2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202"
```

Should return your webhook configuration.

### Step 8.3: Activate Main Workflow

1. In n8n main workflow, look at top-right corner
2. Click the toggle switch to turn it **ON** (should show "Active")

✅ The workflow will now run automatically every time a WhatsApp message arrives!

***

## Part 9: Testing \& Troubleshooting

### Test 9.1: Basic Response Test

Send a WhatsApp message to your connected number:

```
Hello, are you there?
```

**Expected flow:**

1. Evolution API receives message
2. Webhook triggers n8n main workflow
3. AI Agent processes query
4. Response sent back to WhatsApp

Check response in WhatsApp (should arrive in 5-15 seconds)

### Test 9.2: Google Drive Test

Send:

```
What files are in my Google Drive?
```

**Expected:**

- AI Agent uses Google Drive tool
- Returns list of files from your Drive
- Response includes file names


### Test 9.3: WhatsApp Contacts Test

Send:

```
Who did I chat with recently?
```

**Expected:**

- AI Agent uses `get_whatsapp_chats` tool
- Returns list of contacts with names
- Shows recent chat partners


### Test 9.4: Message History Test

Send:

```
What did I speak to [Contact Name]?
```

**Expected:**

1. AI calls `get_whatsapp_chats` first
2. Finds the contact's remoteJid
3. Calls `get_messages_from_contact` with that ID
4. Returns last 10 messages from that contact

### Test 9.5: Check n8n Executions

1. In n8n, go to "Executions" (left sidebar)
2. You should see new executions appearing when messages arrive
3. Click on an execution to see detailed workflow execution
4. Check each node for data flow and errors

### Test 9.6: Check Evolution API Logs

```bash
# If using PM2
pm2 logs evolution-api --lines 50

# If running with npm
# Check the terminal where you ran npm run dev:server
```

Look for:

- "Webhook sent to..."
- "200 OK" responses
- Any error messages


### Test 9.7: Manual Webhook Test

Test the webhook manually with curl:

```bash
curl -X POST http://localhost:5678/webhook/whatsapp-ai-bot \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "key": {
        "remoteJid": "31234567890@s.whatsapp.net",
        "fromMe": false,
        "id": "TEST123"
      },
      "message": {
        "conversation": "Test message"
      }
    }
  }'
```

Check n8n Executions tab to see if it triggered.

***

## Common Issues \& Solutions

### Issue 1: Google Drive Returns Empty Results

**Symptom**: AI Agent says "No files found" but you have files in Drive.

**Solutions:**

1. Check folder ID in Google Drive tool configuration
2. Verify OAuth scopes include `drive.readonly`
3. Test with "Search" operation instead of expecting "List"
4. Try empty search query to return all files in folder
5. Re-authenticate Google Drive credential
6. Check the folder/file permissions (make sure files aren't restricted)

### Issue 2: "401 Unauthorized" from Evolution API

**Symptom**: HTTP Request node shows "Unauthorized" error.

**Solutions:**

1. Check API key matches `.env` file:

```bash
cat ~/evolution-api/.env | grep AUTHENTICATION_API_KEY
```

2. Verify Header Auth credential in n8n:
    - Header name: `apikey` (lowercase, no underscore) ✅
    - NOT `api_key` ❌
    - NOT `API_KEY` ❌
3. Restart Evolution API after changing `.env`:

```bash
pm2 restart evolution-api
```


### Issue 3: "JSON parameter needs to be valid JSON"

**Symptom**: HTTP Request node shows JSON parsing error.

**Solutions:**

1. Use "Using Fields Below" instead of "Using JSON" in HTTP Request
2. If using JSON, ensure expressions don't have extra quotes
3. Add a Set/Code node before HTTP Request to extract clean values
4. Check for special characters in values (quotes, newlines, etc.)
5. Test JSON in a JSON validator first

### Issue 4: WhatsApp Connection Lost

**Symptom**: connectionState shows `state: "close"`.

**Solutions:**

1. Regenerate QR code:

```bash
curl -X GET http://localhost:8080/instance/connect/my-whatsapp \
  -H "apikey: YOUR_API_KEY"
```

2. Scan QR again with WhatsApp
3. Check phone has internet connection
4. Check if WhatsApp on phone is running
5. Restart Evolution API:

```bash
pm2 restart evolution-api
```

6. Check Evolution API logs for disconnect reason

### Issue 5: "Couldn't link device" When Scanning QR

**Symptom**: WhatsApp shows "Couldn't link device, try again later".

**Solutions:**

1. Scan QR immediately (within 60 seconds of generation)
2. Don't save QR as image - scan directly from browser/screen
3. Check if you have 4 linked devices already (WhatsApp max limit)
4. Unlink old devices in WhatsApp settings before adding new one
5. Make sure phone and computer are on stable internet
6. Try regenerating QR code

### Issue 6: Workflow Doesn't Run Automatically

**Symptom**: Must click "Execute workflow" manually each time.

**Solutions:**

1. Activate workflow with toggle switch in top-right corner
2. Use Production Webhook URL in Evolution API, not Test URL
3. Verify webhook is enabled:

```bash
curl -X GET http://localhost:8080/webhook/find/my-whatsapp \
  -H "apikey: YOUR_API_KEY"
```

4. Check n8n Executions tab for incoming triggers
5. Restart n8n if needed
6. Check firewall isn't blocking webhook requests

### Issue 7: Phone Number Format Issues

**Symptom**: Message doesn't send or goes to wrong number.

**Solutions:**

- Use format: `31612345678` (country code + number)
- Remove `+` symbol: `+31612345678` → `31612345678`
- Remove spaces: `31 6 1234 5678` → `31612345678`
- Remove leading zero: `0612345678` → `31612345678`
- Remove `@s.whatsapp.net` if present in sending number (but keep it for remoteJid in chat queries)


### Issue 8: Prisma Client Not Initialized

**Symptom**: `@prisma/client did not initialize yet` error.

**Solutions:**

1. Run Prisma generate:

```bash
npm run db:generate
```

2. Disable database in `.env`:

```
DATABASE_ENABLED=false
```

3. Reinstall dependencies:

```bash
npm install --force
```


### Issue 9: n8n Can't Reach Evolution API

**Symptom**: "Connection refused" or "ECONNREFUSED" errors.

**Solutions:**

1. Check Evolution API is running:

```bash
curl http://localhost:8080
```

2. Verify port 8080 is correct in HTTP Request URL
3. Use `localhost` not `127.0.0.1` (or vice versa)
4. Check firewall isn't blocking port 8080
5. Check PM2 status:

```bash
pm2 status
```


### Issue 10: AI Agent Doesn't Use Google Drive Tool

**Symptom**: AI responds without checking Drive.

**Solutions:**

1. Update system prompt to explicitly mention tool usage
2. Ask specific questions like "Search my Drive for budget.pdf"
3. Check tool is properly connected to AI Agent node
4. Verify Google Drive credential is valid and not expired
5. Test Google Drive tool independently first

### Issue 11: "No messages found" for Existing Contact

**Symptom**: AI says no messages found but conversation exists.

**Solutions:**

1. Check the remoteJid format (must include `@s.whatsapp.net`)
2. Verify the contact has message history in Evolution API
3. Test Evolution API directly:

```bash
curl -X POST http://localhost:8080/chat/findMessages/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{
    "where": {"key": {"remoteJid": "31654207351@s.whatsapp.net"}},
    "limit": 10
  }'
```

4. Check if `STORE_MESSAGES=true` in Evolution API `.env`
5. Messages might be in local storage - restart Evolution API

### Issue 12: Sub-Workflow Not Responding

**Symptom**: AI Agent times out when calling message fetcher.

**Solutions:**

1. Ensure sub-workflow is **Active** (toggle on)
2. Check that remoteJid is being passed correctly (look at execution logs)
3. Verify the webhook URL is correct in Tool 3 configuration
4. Check "Call Evolution API" node is using `{{ $json.body.remoteJid }}` (without `=`)
5. Test sub-workflow directly with curl:

```bash
curl -X POST http://localhost:5678/webhook/fetch-whatsapp-messages \
  -H "Content-Type: application/json" \
  -d '{"remoteJid": "31654207351@s.whatsapp.net"}'
```


### Issue 13: Webhook Not Triggering

**Symptom**: No executions appear when sending WhatsApp messages.

**Solutions:**

1. Verify Evolution API webhook is configured correctly
2. Check that both workflows (main and sub) are Active
3. Test webhook manually with curl (see Test 9.7)
4. Check Evolution API logs for webhook calls
5. Verify Production URL is used, not Test URL
6. Check n8n is running: `http://localhost:5678`

***

## Best Practices

### Security

1. **Change Default API Key**: Generate a new API key in Evolution API `.env`

```bash
openssl rand -hex 32
```

2. **Don't Commit Secrets**: Never commit `.env` files to Git

```bash
echo ".env" >> .gitignore
```

3. **Use Environment Variables**: Store sensitive data in environment variables, not hardcoded
4. **Restrict CORS**: In production, change `CORS_ORIGIN=*` to your specific domain
5. **Enable Webhook Authentication**: Add authentication to webhooks in production
6. **Use HTTPS**: In production, use SSL certificates for all endpoints
7. **Limit Tool Access**: Only give AI Agent access to necessary folders in Google Drive
8. **Regular Updates**: Keep n8n, Evolution API, and Node.js updated

### Performance

1. **Use PM2**: Auto-restart Evolution API on crashes

```bash
pm2 start evolution-api
pm2 save
```

2. **Enable Redis Caching**: For better performance (optional)

```bash
CACHE_REDIS_ENABLED=true
```

3. **Set Up Database**: Use PostgreSQL for persistent message storage (optional)
4. **Monitor Logs Regularly**:

```bash
pm2 logs evolution-api
```

5. **Limit Message History**: Don't fetch more than 50 messages at once
6. **Optimize AI Prompts**: Keep system messages concise for faster responses

### Maintenance

1. **Backup n8n Workflows**: Regularly export workflow JSON files
2. **Backup Evolution API Data**: If using database, set up regular backups
3. **Update Evolution API**:

```bash
cd ~/evolution-api
git pull
npm install
pm2 restart evolution-api
```

4. **Update n8n**:

```bash
npm update -g n8n
```

5. **Monitor WhatsApp Connection**: Set up alerts for connection drops
6. **Test Webhook Periodically**: Ensure webhook is still active
7. **Review Executions**: Check n8n failed executions weekly
8. **Clean Up Old Data**: Archive old messages if storage is limited

***

## Advanced Customizations

### Add More AI Agent Tools

#### Tool 4: Send WhatsApp Message to Any Contact

Add this as a Code tool in AI Agent:

```javascript
const phoneNumber = $json.phoneNumber; // AI provides this
const message = $json.message; // AI provides this

await this.helpers.httpRequest({
  method: 'POST',
  url: 'http://localhost:8080/message/sendText/my-whatsapp',
  headers: {
    'apikey': 'd2b9f256bfebf3a66840e0d8711654a1576ea4aabe2cb43df942dfcea244e202'
  },
  body: {
    number: phoneNumber,
    text: message
  }
});

return { success: true, message: "Message sent successfully" };
```

**Tool Description:**

```text
Sends a WhatsApp message to any phone number. Requires phoneNumber (format: 31612345678) and message text.
```


#### Tool 5: Get File Content from Google Drive

Add Google Drive "Download" operation as a tool to extract file content and send to AI for analysis.

### Improve Message Formatting

Modify the "Format Response" code to include:

```javascript
// Add better date formatting
const formatDate = (timestamp) => {
  const date = new Date(timestamp * 1000);
  const today = new Date();
  const yesterday = new Date(today);
  yesterday.setDate(yesterday.getDate() - 1);
  
  if (date.toDateString() === today.toDateString()) {
    return `Today at ${date.toLocaleTimeString()}`;
  } else if (date.toDateString() === yesterday.toDateString()) {
    return `Yesterday at ${date.toLocaleTimeString()}`;
  } else {
    return date.toLocaleString();
  }
};

// Group messages by date
const groupedMessages = {};
records.forEach(msg => {
  const date = new Date(msg.messageTimestamp * 1000).toDateString();
  if (!groupedMessages[date]) {
    groupedMessages[date] = [];
  }
  groupedMessages[date].push(msg);
});

// Format grouped messages
let formattedText = "";
Object.keys(groupedMessages).forEach(date => {
  formattedText += `\n--- ${date} ---\n`;
  groupedMessages[date].forEach(msg => {
    // Add message formatting here
  });
});
```


### Add Rate Limiting

Prevent spam by adding a Code node that tracks message frequency per user:

```javascript
const phoneNumber = $input.first().json.phoneNumber;
const now = Date.now();

// Get or initialize rate limit storage
const rateLimits = $execution.customData.get('rateLimits') || {};

// Check if user has sent messages recently
if (rateLimits[phoneNumber]) {
  const timeSinceLastMessage = now - rateLimits[phoneNumber].lastMessage;
  const messageCount = rateLimits[phoneNumber].count;
  
  // If more than 5 messages in last 60 seconds, block
  if (timeSinceLastMessage < 60000 && messageCount > 5) {
    return [{
      json: {
        blocked: true,
        message: "Too many requests. Please wait a moment."
      }
    }];
  }
}

// Update rate limit
rateLimits[phoneNumber] = {
  lastMessage: now,
  count: (rateLimits[phoneNumber]?.count || 0) + 1
};

$execution.customData.set('rateLimits', rateLimits);

return [{ json: { blocked: false } }];
```


### Add Multi-Language Support

Update AI system prompt:

```text
Detect the language of the user's message and respond in the same language.

Supported languages:
- English
- Dutch
- Spanish
- French
- German

If user writes in Dutch, respond in Dutch. If user writes in English, respond in English.
```


***

## Quick Reference Commands

### Evolution API Commands

```bash
# Start Evolution API
cd ~/evolution-api
npm run dev:server

# Or with PM2
pm2 start evolution-api
pm2 logs evolution-api
pm2 restart evolution-api
pm2 stop evolution-api

# Check instance status
curl -X GET http://localhost:8080/instance/connectionState/my-whatsapp \
  -H "apikey: YOUR_API_KEY"

# Get QR code
curl -X GET http://localhost:8080/instance/connect/my-whatsapp \
  -H "apikey: YOUR_API_KEY" | jq -r '.base64'

# Send test message
curl -X POST http://localhost:8080/message/sendText/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{"number": "31612345678", "text": "Test"}'

# Get all chats
curl -X POST http://localhost:8080/chat/findChats/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{}'

# Get messages from contact
curl -X POST http://localhost:8080/chat/findMessages/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{
    "where": {"key": {"remoteJid": "31654207351@s.whatsapp.net"}},
    "limit": 10
  }'

# Configure webhook
curl -X POST http://localhost:8080/webhook/set/my-whatsapp \
  -H "Content-Type: application/json" \
  -H "apikey: YOUR_API_KEY" \
  -d '{
    "webhook": {
      "enabled": true,
      "url": "http://localhost:5678/webhook/whatsapp-ai-bot",
      "webhookByEvents": false,
      "events": ["MESSAGES_UPSERT"]
    }
  }'

# Check webhook status
curl -X GET http://localhost:8080/webhook/find/my-whatsapp \
  -H "apikey: YOUR_API_KEY"
```


### n8n Commands

```bash
# Start n8n
n8n start

# Or in background
nohup n8n start > /dev/null 2>&1 &

# Update n8n
npm update -g n8n

# Access n8n
open http://localhost:5678
```


### Useful Debugging Commands

```bash
# Check if ports are in use
lsof -i :5678  # n8n
lsof -i :8080  # Evolution API

# Check Node.js version
node --version

# Check PM2 processes
pm2 status
pm2 describe evolution-api

# View Evolution API logs
pm2 logs evolution-api --lines 100

# Restart all services
pm2 restart all

# Generate new API key
openssl rand -hex 32
```


***

## Conclusion

You now have a **fully functional AI-powered WhatsApp chatbot** that can:

✅ Respond to WhatsApp messages automatically
✅ Search and retrieve Google Drive files
✅ Access WhatsApp chat history
✅ Fetch messages from specific contacts
✅ Provide intelligent, context-aware responses using AI
✅ Handle multiple tools and sub-workflows

### What You've Built

1. **Main Workflow**: Receives messages, processes with AI, sends responses
2. **Sub-Workflow**: Fetches WhatsApp message history on demand
3. **Three AI Tools**: Google Drive, Contact List, Message History
4. **Production-Ready Setup**: PM2 process management, proper error handling

### Next Steps

1. **Deploy to Production**: Set up on a VPS with domain and SSL
2. **Add More Tools**: Calendar integration, Gmail, web scraping, etc.
3. **Improve Prompts**: Refine AI behavior for your specific use case
4. **Set Up Monitoring**: Add logging and alerts for failures
5. **Add Authentication**: Secure webhooks with API keys or tokens
6. **Database Integration**: Enable PostgreSQL for persistent storage
7. **Multi-Instance**: Support multiple WhatsApp accounts
8. **Analytics**: Track usage metrics and response quality

### Resources

- **n8n Documentation**: [https://docs.n8n.io/](https://docs.n8n.io/)
- **Evolution API GitHub**: [https://github.com/EvolutionAPI/evolution-api](https://github.com/EvolutionAPI/evolution-api)
- **Evolution API Docs**: [https://doc.evolution-api.com/](https://doc.evolution-api.com/)
- **Google Drive API**: [https://developers.google.com/drive](https://developers.google.com/drive)
- **Gemini API**: [https://ai.google.dev/](https://ai.google.dev/)


### Support

If you encounter issues:

1. Check the [Common Issues \& Solutions](#common-issues-solutions) section
2. Review n8n Executions tab for errors
3. Check Evolution API logs: `pm2 logs evolution-api`
4. Test each component independently
5. Review system prompts and tool descriptions

***

Happy automating! 🚀🤖