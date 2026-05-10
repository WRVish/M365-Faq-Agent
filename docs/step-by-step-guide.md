# M365 FAQ Agent - Complete Step-by-Step Build Guide

**Build an AI-powered FAQ agent for Microsoft 365 using Copilot Studio, SharePoint knowledge sources, and Power Automate escalation workflows.**

This guide walks you through creating a complete M365 FAQ Agent that:
- ✅ Answers questions about Microsoft Teams, SharePoint, Power Apps, Power Automate, Power Platform, and other tools
- ✅ Searches curated SharePoint knowledge bases using AI semantic search
- ✅ Escalates unanswered questions to IT ops team via Teams, email, and tracking number
- ✅ Logs all escalations to SharePoint for tracking and analytics
- ✅ Allows users to ask multiple questions and switch between products seamlessly

---

## Prerequisites

Before you start:
- Admin access to Microsoft 365 tenant
- Access to Copilot Studio (copilotstudio.microsoft.com)
- Access to Power Automate (make.powerautomate.com)
- SharePoint site created (we'll call it "FAQHub" or "DemoSite")
- Basic familiarity with SharePoint lists
- IT ops Teams channel created (e.g., #ops-faq-escalations)

---

## Architecture Overview

**Components:**
1. **Copilot Studio Agent** - The conversational AI interface
2. **SharePoint Lists** - 6 FAQ knowledge bases (one per product) + 1 analytics list
3. **Power Automate Flow** - EscalateToTeams flow for handling escalations
4. **Microsoft Teams** - Deployment channel and ops notifications
5. **Email** - Notifications to ops team and user confirmations

**User Flow:**
```
User opens chat 
→ Selects product (Teams/SharePoint/Power Apps/etc.)
→ Asks question
→ Agent searches that product's knowledge base
→ IF ANSWER FOUND: Shows answer → Ask another question or switch products
→ IF NO ANSWER: Escalates → Sends to Teams channel + Emails + Tracking number
```

---

## Section 1 — Prepare SharePoint Knowledge Sources

### 1.1 Create the SharePoint site

Go to your SharePoint admin center or use an existing site.

For this guide, we'll assume you have a site at:
```
https://yourtenant.sharepoint.com/sites/FAQHub
```

Or:
```
https://vishtechtalk.sharepoint.com/sites/DemoSite
```

Replace with your actual tenant name and site name throughout this guide.

---

### 1.2 Create the 7 SharePoint lists

You need to create 7 lists total:
- 6 FAQ lists (one for each product category)
- 1 Analytics list (for logging queries and escalations)

**Create these lists:**

1. **FAQ_Teams** - Microsoft Teams questions
2. **FAQ_SharePoint** - SharePoint questions
3. **FAQ_PowerApps** - Power Apps questions
4. **FAQ_PowerAutomate** - Power Automate questions
5. **FAQ_PowerPlatform** - Power Platform questions
6. **FAQ_NonM365** - Other tools (Zoom, VPN, etc.)
7. **FAQ_Analytics** - Query and escalation logging

---

### 1.3 Add columns to the FAQ lists

For EACH of the 6 FAQ lists (FAQ_Teams, FAQ_SharePoint, etc.), add these columns:

**Required columns for knowledge search:**

| Column Name | Type                   | Description                                       |
| ----------- | ---------------------- | ------------------------------------------------- |
| Title       | Single line of text    | The FAQ question (default column)                 |
| Answer      | Multiple lines of text | The complete answer with steps, links, details    |
| Keywords    | Single line of text    | Search keywords (e.g., "guest, external, invite") |

**Optional metadata columns** (for organization, not searched):

| Column Name  | Type                | Description                                                  |
| ------------ | ------------------- | ------------------------------------------------------------ |
| Product      | Single line of text | Product name (e.g., "Teams")                                 |
| Category     | Choice              | Topic category (Permissions, Sharing, Troubleshooting, etc.) |
| LastReviewed | Date                | When was this FAQ last reviewed                              |
| Owner        | Person              | Who maintains this FAQ                                       |

**Important:** Only **Title**, **Answer**, and **Keywords** will be indexed for AI search. The other columns are for your internal organization only.

---

### 1.4 Create the FAQ_Analytics list

Create one more list called **FAQ_Analytics** with these columns:

| Column Name         | Type                | Description                               |
| ------------------- | ------------------- | ----------------------------------------- |
| Title               | Single line of text | The user's question (default column)      |
| QueryDate           | Date and Time       | When the query happened                   |
| Product             | Single line of text | Which product (Teams, SharePoint, etc.)   |
| ResolutionStatus    | Choice              | Values: "escalated"                       |
| UserEmail           | Single line of text | Email of the person who asked             |
| EscalationTicketRef | Single line of text | Ticket reference (e.g., OPS-202605091545) |
| MatchedFAQTitle     | Single line of text | (Leave blank for escalations)             |
| MatchedFAQItemId    | Number              | (Leave blank for escalations)             |
| ConfidenceScore     | Number              | (Leave blank for escalations)             |

This list will track all escalated queries with their tracking numbers.

---

### 1.5 Load test data into FAQ lists

Add at least 3-5 FAQ items to each list so you have content to test with.

**Example for FAQ_Teams:**

| Title                                    | Answer                                                                                                                                                                                                                                    | Keywords                             |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| How do I add a guest to a Teams channel? | To add a guest to a Teams channel: 1. Go to the team and channel. 2. Click More options (...) > Manage team. 3. Click Members > Add member. 4. Enter the guest's email address. 5. Click Add. The guest will receive an invitation email. | guest, external, invite, add member  |
| How do I create a private channel?       | To create a private channel: 1. Go to the team. 2. Click More options (...) > Add channel. 3. Enter channel name. 4. Select Privacy: Private. 5. Click Add. Only members you add will see this channel.                                   | private, channel, create, restricted |
| How do I schedule a Teams meeting?       | To schedule a Teams meeting: 1. Click Calendar in Teams. 2. Click New meeting. 3. Add title, date, time, and attendees. 4. Click Save. Attendees will receive a calendar invite.                                                          | meeting, schedule, calendar, invite  |

Repeat for the other 5 FAQ lists with relevant questions for each product.

---

**Section 1 Complete! You now have:**

✅ SharePoint site ready  
✅ 6 FAQ lists created with proper columns  
✅ 1 Analytics list for logging  
✅ Test data loaded in each FAQ list  

---

## Section 2 — Configure the Copilot Studio Agent

Before you build any Power Automate flows, you need to create and configure the Copilot Studio agent. This section covers everything: creating the agent, setting its identity, writing instructions, connecting knowledge sources, and building the conversation topics.

Before you start: make sure you have completed Section 1 and all seven SharePoint lists exist on your site with the correct columns and at least some test data loaded.

---

### 2.1 Create the agent in Copilot Studio

Go to **copilotstudio.microsoft.com** and sign in with your admin account.

Click **Create** in the left navigation, then click **New agent**.

Give your agent a name: `M365 FAQ Agent`

Choose the language: **English**

Click **Create**. Copilot Studio creates a blank agent and opens the agent workspace.

---

### 2.2 Configure agent name, description, and icon

These settings control how the agent appears to users in Microsoft Teams. Configure these first because they define the agent's identity.

In the agent workspace, click **Settings** (gear icon in top right).

Click **Agent details** in the left menu.

**Agent name:**  
The name is already set to `M365 FAQ Agent` from when you created it. You can change it here if needed. This name appears:
- In the Teams app list
- At the top of the chat window
- In the Teams search results

Keep it short and descriptive. Good examples: "IT Help Agent", "M365 Support Bot", "FAQ Assistant"

**Description:**  
This appears in Teams when users search for your agent or browse the app catalog. Write a clear description that tells people what the agent does.

Click in the **Description** field and paste this:

```
Get instant answers to Microsoft 365 questions about Teams, SharePoint, Power Apps, Power Automate, and more. Ask a question and get help immediately, or get escalated to the IT ops team if needed.
```

You can customize this to match your organization's tone.

**Short description:**  
This appears in smaller spaces like the Teams sidebar. Keep it under 80 characters.

```
Instant help with Microsoft 365 products and tools
```

**Agent icon:**  
Click **Choose file** under Agent icon. Upload a logo PNG file (512x512 pixels).

If you don't have a logo yet, you can:
- Use the Microsoft 365 logo
- Create one at brandmark.io or canva.com
- Use a simple icon from icons8.com
- Leave the default Copilot Studio icon

**Agent color:**  
Pick a color that matches your organization's branding or Microsoft 365 (use `#0078D4` for Microsoft blue).

Click **Save** at the bottom.

---

### 2.3 Write the agent instructions

Agent instructions (also called the system prompt) tell the agent how to behave in every conversation. This is one of the most important configuration steps. Configure this BEFORE adding knowledge sources so the agent knows what to do with the knowledge.

In the agent workspace, click **Settings**, then click **Instructions** in the left menu.

You'll see a large text box. Delete any default text and paste this:

```
You are the M365 FAQ Agent, an IT support assistant that helps employees with Microsoft 365 questions.

## Your capabilities
- You can answer questions about Microsoft Teams, SharePoint, Power Apps, Power Automate, Power Platform, and other tools
- You search a curated knowledge base of FAQ content maintained by the IT team
- You escalate questions you cannot answer confidently to the IT operations team

## Conversation flow
1. Always greet users and show them the product selection buttons immediately
2. After they select a product, ask what their question is about that product
3. Search only the knowledge source for that specific product
4. If you find a confident answer, provide it with the full FAQ content
5. If you do not find a confident answer, call the EscalateToTeams action and tell the user their question has been escalated with a ticket reference

## Response guidelines
- Be helpful, professional, and concise
- Always provide the complete answer from the knowledge base, not a summary
- If the answer includes steps, present them clearly
- If the answer references links or resources, include them
- Never make up answers or guess if you are not confident
- Never say "I don't know" without escalating

## When to escalate
Escalate to the IT ops team when:
- No FAQ matches the user's question
- The best match has low confidence
- The user explicitly asks to speak with a human
- The user says the answer you provided did not help
- The question is about a specific error code, outage, or account-specific issue

## Tone
- Friendly but professional
- Clear and direct
- Empathetic when users are frustrated
- Encouraging when providing answers

Always remember: your goal is to help users quickly. If you can answer from the knowledge base, do it immediately. If you cannot, escalate immediately. Never leave users waiting or uncertain about next steps.
```

Click **Save**.

**Why these instructions matter:**

Without clear instructions, the agent might:
- Search all knowledge sources instead of the one the user selected
- Summarize answers instead of providing the full FAQ content
- Not know when to escalate
- Give uncertain responses like "I'm not sure" without escalating
- Skip the product selection step

The instructions you just pasted prevent all of these problems.

---

### 2.4 Add SharePoint lists as knowledge sources

Now that the agent knows WHO it is and HOW to behave, give it WHAT to search. You'll connect each FAQ list as a separate knowledge source so the agent can search them individually based on which product the user selected.

In **Settings**, click **Knowledge** in the left menu.

Click **+ Add knowledge**.

A panel opens on the right. Choose **SharePoint** from the list of source types.

**Add each product list as a separate knowledge source using its direct URL:**

You'll add 6 knowledge sources total, one for each product. Each list has a direct URL that points to it.

**Your list URLs will follow this pattern:**
```
https://[yourtenant].sharepoint.com/sites/[YourSiteName]/Lists/[ListName]
```

For example, if your site is `https://vishtechtalk.sharepoint.com/sites/DemoSite`, your list URLs are:
- `https://vishtechtalk.sharepoint.com/sites/DemoSite/Lists/FAQ_Teams`
- `https://vishtechtalk.sharepoint.com/sites/DemoSite/Lists/FAQ_SharePoint`
- And so on...

**How to get your exact list URLs:**

1. Open your site in SharePoint
2. Click on one of the FAQ lists (e.g., FAQ_Teams) in the left navigation
3. Copy the URL from your browser address bar
4. It should look like: `https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_Teams`

**Now add each knowledge source:**

**For FAQ_Teams:**

1. In the SharePoint URL field, paste the direct list URL (replace with YOUR actual site name):
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_Teams
   ```
2. Click **Next**
3. Copilot Studio recognizes this as a list and shows you the columns
4. Select these columns to index: **Title**, **Answer**, **Keywords**
5. Uncheck all other columns (Product, Category, etc.)
6. Click **Add**
7. Give this knowledge source a name: `Teams FAQs`
8. Wait for the status to show "Indexed"

**For FAQ_SharePoint:**

1. Click **+ Add knowledge** again
2. Choose **SharePoint**
3. Paste the list URL:
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_SharePoint
   ```
4. Click **Next**
5. Select columns to index: **Title**, **Answer**, **Keywords**
6. Click **Add**
7. Name it: `SharePoint FAQs`

**For FAQ_PowerApps:**

1. Click **+ Add knowledge**
2. Choose **SharePoint**
3. Paste the list URL:
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_PowerApps
   ```
4. Click **Next**
5. Select columns: **Title**, **Answer**, **Keywords**
6. Click **Add**
7. Name it: `Power Apps FAQs`

**For FAQ_PowerAutomate:**

1. Click **+ Add knowledge**
2. Choose **SharePoint**
3. Paste the list URL:
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_PowerAutomate
   ```
4. Click **Next**
5. Select columns: **Title**, **Answer**, **Keywords**
6. Click **Add**
7. Name it: `Power Automate FAQs`

**For FAQ_PowerPlatform:**

1. Click **+ Add knowledge**
2. Choose **SharePoint**
3. Paste the list URL:
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_PowerPlatform
   ```
4. Click **Next**
5. Select columns: **Title**, **Answer**, **Keywords**
6. Click **Add**
7. Name it: `Power Platform FAQs`

**For FAQ_NonM365:**

1. Click **+ Add knowledge**
2. Choose **SharePoint**
3. Paste the list URL:
   ```
   https://yourtenant.sharepoint.com/sites/YourSiteName/Lists/FAQ_NonM365
   ```
4. Click **Next**
5. Select columns: **Title**, **Answer**, **Keywords**
6. Click **Add**
7. Name it: `Other Tools FAQs`

Wait for all 6 knowledge sources to finish indexing. Status should show "Indexed" for each one. This usually takes 2-5 minutes per list.

**Important notes:**

- Replace `yourtenant` and `YourSiteName` with your actual SharePoint tenant and site names
- Always use the direct list URL (ending in `/Lists/FAQ_Teams`), not the site URL
- Copy the URL directly from your browser when viewing each list to ensure it's correct
- Do NOT add FAQ_Analytics as a knowledge source — that list is for logging queries only
- Make sure all 6 show "Indexed" status before proceeding to the next section

---

### 2.5 Configure Generative AI orchestration

Enable the AI features that let the agent use knowledge sources and call Power Automate flows automatically.

In **Settings**, click **Generative AI** in the left menu.

**Orchestration:**

You'll see the question "Use generative AI orchestration for your agent's responses?"

Make sure **Yes - Responses will be dynamic, using available tools and knowledge as appropriate** is selected (this should be selected by default).

This allows the agent to automatically use the knowledge sources you connected and call Power Automate flows when needed.

**Connected agents:**

Turn **ON** the toggle for "Let other agents connect to and use this one". This allows the agent to work with other Copilot Studio agents if needed in the future.

Click **Save** at the bottom.

**Note about Fallback behavior:**

In the current Copilot Studio interface, fallback behavior is handled automatically by the Generative AI orchestration. When the agent can't find a confident answer from knowledge sources:

1. It will automatically recognize low confidence
2. You'll configure it to call the EscalateToTeams flow in Section 4 (when we connect the flows to the agent)
3. The agent's instructions (Section 2.3) already tell it when to escalate

You don't need a separate "Fallback topic" — the AI orchestration handles this based on your agent instructions.

---

### 2.6 Configure the Conversation Start topic with product selection

Now you'll build the conversation flow that fires automatically when users open the chat. You'll use the **Conversation Start** system topic to show product selection buttons immediately.

In the agent workspace, click **Topics** in the left navigation.

Click on **System (9)** tab at the top.

Click on **Conversation Start** topic to open it.

**Step 1 — Edit the greeting message**

You'll see a **Message** node that says "Hello, I'm {x} Bot.Name string ..."

Click on this message node to edit it.

Replace the entire message with this text:

```
Hi! I'm the M365 FAQ Agent. I can help you with questions about Microsoft 365 products and tools.

Which product do you need help with?
```

**Step 2 — Add product selection buttons**

Below the message, click the **+** button → **Ask a question** → **Multiple choice**.

In the question text field, leave it empty (the message above already asks the question).

Click **+ New option** to add each product button. Add these 6 options:

- Microsoft Teams
- SharePoint
- Power Apps
- Power Automate
- Power Platform
- Other (Zoom, VPN, etc.)

**Save the user response to a variable:**

In the question node properties, find "Save response as".

Change the variable name to: `VarProductSelected`

This variable will store which product button the user clicked.

Click **Save** in the top right.

---

### 2.7 Create the 6 product topics

Each product needs its own topic. Each topic asks for the user's question, searches only that product's knowledge source, and returns the answer.

Go back to **Topics** in the left nav.

Click **+ New topic** → **From blank**.

**Creating the Teams Questions topic:**

1. Topic name: `TeamsQuestions`

2. The trigger is already added automatically. You'll see a **Trigger** node that says "The agent chooses".

3. Click **Edit** on the trigger node. In the "Describe what the topic does" field, paste this:
   ```
   This topic handles questions about Microsoft Teams. It is triggered when the user selects Teams from the product menu and asks their question about Teams features, channels, meetings, or guest access.
   ```
   This helps the agent know when to use this topic.

4. Below the trigger, click the **+** button → **Ask a question**

5. Question text: `What's your question about Microsoft Teams?`

6. Identify: select **User's entire response**

7. Save response to a variable. Click on the variable name and rename it to: `VarUserQuestion`

8. Click the **+** button below the question → **Advanced** → **Generative answers**

9. In the Generative answers node, click **Edit**

10. In the "Input" field, click the variable icon **{x}** and select **VarUserQuestion**

11. Under "Data sources", click **Edit** (the pencil icon)

12. In the Knowledge sources panel, toggle ON **Search only selected sources**, then check **Teams FAQs** (and uncheck all others)

13. Click **Save** on the Generative answers node

14. Click **Save** on the topic

**Repeat the exact same steps for the other 5 products:**

**SharePoint Questions topic:**
- Topic name: `SharePointQuestions`
- Trigger description: "This topic handles questions about SharePoint including sites, document libraries, permissions, and external sharing."
- Ask: "What's your question about SharePoint?"
- Save to: VarUserQuestion
- Generative answers source: **SharePoint FAQs** (only)

**Power Apps Questions topic:**
- Topic name: `PowerAppsQuestions`
- Trigger description: "This topic handles questions about Power Apps including canvas apps, model-driven apps, connectors, and formulas."
- Ask: "What's your question about Power Apps?"
- Save to: VarUserQuestion
- Generative answers source: **Power Apps FAQs** (only)

**Power Automate Questions topic:**
- Topic name: `PowerAutomateQuestions`
- Trigger description: "This topic handles questions about Power Automate including flows, triggers, approvals, and connectors."
- Ask: "What's your question about Power Automate?"
- Save to: VarUserQuestion
- Generative answers source: **Power Automate FAQs** (only)

**Power Platform Questions topic:**
- Topic name: `PowerPlatformQuestions`
- Trigger description: "This topic handles questions about Power Platform including environments, DLP policies, licensing, and governance."
- Ask: "What's your question about Power Platform?"
- Save to: VarUserQuestion
- Generative answers source: **Power Platform FAQs** (only)

**Other Tools Questions topic:**
- Topic name: `OtherToolsQuestions`
- Trigger description: "This topic handles questions about non-Microsoft 365 tools including Zoom, VPN, ITSM systems, and other IT services."
- Ask: "What's your question about other tools?"
- Save to: VarUserQuestion
- Generative answers source: **Other Tools FAQs** (only)

**Step 15 — Add continuation and product switching logic**

After the Generative answers node in EACH product topic, you need to add logic that lets users:
- Ask another question about the same product
- Switch to a different product
- End the conversation

For each topic, after the Generative answers node:

1. Click the **+** button → **Ask a question** → **Multiple choice**

2. Question text: `What would you like to do next?`

3. Add 3 options:
   - `Another Teams question` (change "Teams" to match the product)
   - `Different product`
   - `I'm done`

4. Save response to variable: `WhatNext`

5. Click **+** → **Add a condition**

**Create 3 condition branches:**

**Branch 1: Same product (loop back)**
- Condition: `WhatNext` is equal to `Another Teams question` (change to match product)
- Action: **Topic management** → **Repeat this topic**
- This loops back to ask another question about the same product

**Branch 2: Different product (show menu)**
- Condition: `WhatNext` is equal to `Different product`
- Action: **Topic management** → **Go to another topic** → Select **Greeting**
- This shows the product selection menu again

**Branch 3: Done (end conversation)**
- Condition: `WhatNext` is equal to `I'm done`
- Actions:
  - **Send a message**: `Thank you for using M365 FAQ Agent. Have a great day!`
  - **Topic management** → **End conversation**

Click **Save** on each topic.

**Important:** You must add this continuation logic to ALL 6 product topics. The only difference between them is the first option label:
- Teams Questions: "Another Teams question"
- SharePoint Questions: "Another SharePoint question"
- Power Apps Questions: "Another Power Apps question"
- And so on...

---

### 2.8 Connect the Conversation Start to the product topics

Now connect the product selection buttons in the Conversation Start topic to the correct product topics using condition logic.

Go back to the **Conversation Start** topic (Topics → System (9) → Conversation Start).

After the product selection question node, you need to add conditions that redirect to the right topic based on which button the user clicked.

Click the **+** button below the question node → **Add a condition**.

**Build 6 condition branches:**

**Branch 1 - If user selected "Microsoft Teams":**
- Condition: `VarProductSelected` is equal to `Microsoft Teams`
- Action: Click **+ Add node** → **Topic management** → **Go to another topic** → select `TeamsQuestions`

**Branch 2 - If user selected "SharePoint":**
- Click **Add condition** to add another branch
- Condition: `VarProductSelected` is equal to `SharePoint`
- Action: Go to another topic → select `SharePointQuestions`

**Branch 3 - If user selected "Power Apps":**
- Add condition
- Condition: `VarProductSelected` is equal to `Power Apps`
- Action: Go to another topic → select `PowerAppsQuestions`

**Branch 4 - If user selected "Power Automate":**
- Add condition
- Condition: `VarProductSelected` is equal to `Power Automate`
- Action: Go to another topic → select `PowerAutomateQuestions`

**Branch 5 - If user selected "Power Platform":**
- Add condition
- Condition: `VarProductSelected` is equal to `Power Platform`
- Action: Go to another topic → select `PowerPlatformQuestions`

**Branch 6 - If user selected "Other":**
- Add condition
- Condition: `VarProductSelected` is equal to `Other (Zoom, VPN, etc.)`
- Action: Go to another topic → select `OtherToolsQuestions`

Click **Save**.

---

### 2.9 Create a Greeting topic (for when users say "hi")

You also need a custom Greeting topic that fires when users say "hello" or "hi" (different from Conversation Start which fires automatically when chat opens).

Go to **Topics** → **Custom** tab.

Click **+ New topic** → **From blank**.

Topic name: `Greeting`

**Add trigger phrases:**

Click **Edit** on the Trigger node.

Add these trigger phrases:
- Good afternoon
- Good morning
- Hello
- Hey
- Hi

**Add the same product selection as Conversation Start:**

1. Click **+** → **Ask a question** → **Multiple choice**

2. Question: `Which product do you need help with?`

3. Add 6 options (same as before):
   - Microsoft Teams
   - SharePoint
   - Power Apps
   - Power Automate
   - Power Platform
   - Other (Zoom, VPN, etc.)

4. Save to variable: `VarProductSelected`

5. Add the same 6 condition branches that redirect to each product topic (same as Conversation Start)

Click **Save**.

---

### 2.10 Create a Start Over topic (for restarting conversation)

Create one more topic to let users restart the conversation.

Click **+ New topic** → **From blank**.

Topic name: `Start Over`

**Add trigger phrases:**

- let's begin again
- start over
- start again
- restart

**Add confirmation and redirect:**

1. Click **+** → **Ask a question** → **Boolean (Yes/No)**

2. Question: `Are you sure you want to restart the conversation?`

3. Save to variable: `Confirm`

4. Click **+** → **Add a condition**

**Branch 1 - If YES:**
- Condition: `Confirm` is equal to `true`
- Actions:
  - **Send a message**: `Okay, let's start fresh!`
  - **Go to another topic** → Select **Greeting**

**Branch 2 - If NO:**
- Condition: `Confirm` is equal to `false`
- Action:
  - **Send a message**: `Ok. Let's carry on.`

Click **Save**.

---

### 2.11 Test the complete flow

Before you build any Power Automate flows, test that the product selection and knowledge search are working end-to-end.

Click **Test** in the top right corner. The test chat panel opens.

**Test conversation 1 - Teams:**

1. The agent should immediately show the greeting and product buttons
2. Click **Microsoft Teams**
3. The agent asks "What's your question about Microsoft Teams?"
4. Type: `How do I add a guest to a channel?`
5. The agent searches the Teams FAQs knowledge source and returns the answer
6. Agent asks: "What would you like to do next?"
7. Click "Another Teams question" - should loop back
8. Click "Different product" - should show product menu again
9. Click "I'm done" - should say goodbye and end

**Test conversation 2 - SharePoint:**

1. Restart the conversation (click the reset icon)
2. Click **SharePoint**
3. Type a SharePoint question like `How do I share a file externally?`
4. Verify the agent searches only SharePoint FAQs and returns an answer

**Test conversation 3 - Greeting:**

1. Restart the conversation
2. Type "hello"
3. Greeting topic should fire and show product buttons

If all three tests work correctly, your agent foundation is complete.

If the search returns nothing:
- Check that your FAQ lists have content
- Check that all 6 knowledge sources show "Indexed" status in Settings → Knowledge
- Check that each product topic is configured to search the correct knowledge source
- Check that the Conversation Start condition branches redirect to the correct topics

---

**What you've completed in Section 2:**

✅ Agent created  
✅ Agent name, description, and icon configured  
✅ Agent instructions written (system prompt)  
✅ 6 SharePoint lists connected as separate knowledge sources  
✅ Generative AI orchestration enabled  
✅ Conversation Start topic with product selection buttons  
✅ 6 product topics that search specific knowledge sources with continuation logic  
✅ Greeting topic for when users say "hi"  
✅ Start Over topic for restarting conversations  
✅ Conversation Start connected to product topics with conditions  
✅ Complete flow tested end-to-end  

**Next:** Section 3 - Build the EscalateToTeams Power Automate flow

---

## Section 3 — Build the EscalateToTeams Power Automate Flow

Now that the agent can search for answers using Copilot Studio's knowledge sources, you need one Power Automate flow to handle escalations when the agent can't find a confident answer.

The EscalateToTeams flow will:
1. ✅ Generate a unique ticket reference number
2. ✅ Post an Adaptive Card to your IT ops Teams channel
3. ✅ Send an email to the IT ops team
4. ✅ Send an email to the user with their tracking number
5. ✅ Log the escalation to SharePoint FAQ_Analytics list
6. ✅ Return the ticket reference to the agent

Before you build: go to **make.powerautomate.com** and create a new solution. Call it **M365FAQAgent**. Build the flow inside this solution.

---

### 3.1 Create the EscalateToTeams flow

**Step 1 — Create the flow**

Inside your M365FAQAgent solution, click **New** → **Cloud flow** → **Instant**.

When asked for a trigger, search for "When Copilot Studio calls a flow" and select the **V2** version (important).

Click **Create**.

Name the flow: `EscalateToTeams`

---

**Step 2 — Add the input parameters**

The agent will pass information about the escalation to this flow. You need to tell the flow what inputs to expect.

In the trigger card, click **Add an input**. You'll add 4 text inputs:

**First input:**
- Input type: **Text**
- Input name: `userQuestion`
- Sample data: `How do I reset my password?`

**Second input:**
- Input type: **Text**
- Input name: `userEmail`
- Sample data: `john.doe@company.com`

**Third input:**
- Input type: **Text**
- Input name: `product`
- Sample data: `Teams`

**Fourth input:**
- Input type: **Text**
- Input name: `conversationId`
- Sample data: `a:1234abcd`

The conversationId is how the ops team's response will get back to the user's chat.

Rename the trigger to `Trigger-CopilotStudioCall` by clicking the three dots.

---

**Step 3 — Generate a unique ticket reference**

Click **New step** and search for **Initialize variable**.

Fill in the fields:

- Name: `ticketRef`
- Type: select **String** from the dropdown
- Value: click in the field, then click the **fx** (expression) button. Paste this expression:

```
concat('OPS-', formatDateTime(utcNow(), 'yyyyMMddHHmm'))
```

Click **OK**.

What this does: creates a ticket reference like `OPS-202605091430` using the current date and time.

Rename this action to `Var-ticketRef`.

---

**Step 4 — Post the Adaptive Card to Teams**

Click **New step** and search for **Post adaptive card in a chat or channel**.

Choose the one that says **Microsoft Teams** underneath it.

Fill in the fields:

**Post as:** select **Flow bot** from the dropdown

**Post in:** select **Channel** from the dropdown

**Team:** Click in this field. A list of your Teams appears. Select your IT ops team (the one where you created the #ops-faq-escalations channel).

**Channel:** Click in this field. Select **ops-faq-escalations** (or whatever you named your escalation channel).

**Adaptive Card:** This is where you paste the card JSON. Click in the large text box and paste this:

```json
{
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "🎫 FAQ Escalation",
      "size": "Large",
      "weight": "Bolder",
      "color": "Attention"
    },
    {
      "type": "FactSet",
      "facts": [
        {
          "title": "Ticket:",
          "value": "${ticketRef}"
        },
        {
          "title": "User:",
          "value": "${userEmail}"
        },
        {
          "title": "Product:",
          "value": "${product}"
        }
      ]
    },
    {
      "type": "TextBlock",
      "text": "Question:",
      "weight": "Bolder",
      "spacing": "Medium"
    },
    {
      "type": "TextBlock",
      "text": "${userQuestion}",
      "wrap": true,
      "color": "Dark"
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Claim and Respond",
      "data": {
        "action": "claim",
        "ticketRef": "${ticketRef}",
        "userEmail": "${userEmail}",
        "conversationId": "${conversationId}"
      }
    }
  ]
}
```

Now you need to replace the `${}` placeholders with actual dynamic content.

**To insert dynamic values in JSON:**

1. Find `"value": "${ticketRef}"` in the JSON
2. Delete `${ticketRef}` (keep the quotes)
3. Click between the quotes
4. Click **Add dynamic content**
5. Select **ticketRef** from the Var-ticketRef section
6. The JSON should now show `"value": "@{variables('ticketRef')}"`

Repeat for the other placeholders:
- Replace `${userEmail}` with the **userEmail** dynamic token
- Replace `${product}` with the **product** dynamic token  
- Replace `${userQuestion}` with the **userQuestion** dynamic token
- Replace `${conversationId}` in the actions section with the **conversationId** dynamic token

Rename this action to `Teams-PostEscalationCard`.

---

**Step 5 — Send email to IT ops team**

Click **New step** and search for **Send an email (V2)**.

Choose the one that says **Office 365 Outlook** underneath it.

Fill in the fields:

**To:** Enter your IT ops team email (e.g., `itops@yourcompany.com`)

You can also enter multiple emails separated by semicolons, like:
`itops@yourcompany.com;support@yourcompany.com`

**Subject:** Click in the field and type:

```
New FAQ Escalation - 
```

Then:
1. Put your cursor after the dash
2. Click **Add dynamic content**
3. Select **ticketRef** from the Var-ticketRef section

The subject should now show: `New FAQ Escalation - @{variables('ticketRef')}`

**Body:** Click in the field and paste this template:

```html
<p><strong>🎫 FAQ Escalation Alert</strong></p>

<p><strong>Ticket Reference:</strong> [ticketRef]</p>

<p><strong>User:</strong> [userEmail]</p>

<p><strong>Product:</strong> [product]</p>

<p><strong>Question:</strong></p>
<p>[userQuestion]</p>

<hr>

<p><strong>Next Steps:</strong></p>
<ul>
  <li>Review the question in the Teams channel: #ops-faq-escalations</li>
  <li>Respond to the user within 2 business hours</li>
  <li>If this is a recurring question, add it to the FAQ knowledge base</li>
</ul>

<p><em>This is an automated email from the M365 FAQ Agent.</em></p>
```

Now replace the placeholders with dynamic content:
1. Delete `[ticketRef]` and insert the **ticketRef** dynamic token
2. Delete `[userEmail]` and insert the **userEmail** dynamic token
3. Delete `[product]` and insert the **product** dynamic token
4. Delete `[userQuestion]` and insert the **userQuestion** dynamic token

Rename this action to `Email-NotifyOpsTeam`.

---

**Step 6 — Send email to user with tracking number**

Click **New step** and search for **Send an email (V2)** again.

Choose **Office 365 Outlook**.

Fill in the fields:

**To:** Click **Add dynamic content** → select **userEmail**

**Subject:** Type:

```
Your FAQ Support Request - 
```

Then insert the **ticketRef** dynamic token after the dash.

**Body:** Click in the field and paste this template:

```html
<p>Hello,</p>

<p>Thank you for contacting M365 FAQ Agent. We couldn't find an immediate answer to your question, so we've escalated it to our IT operations team.</p>

<p><strong>Your Question:</strong></p>
<p>[userQuestion]</p>

<p><strong>Your Tracking Number:</strong> <span style="color: #0078D4; font-weight: bold;">[ticketRef]</span></p>

<p><strong>What Happens Next:</strong></p>
<ul>
  <li>Our IT ops team has been notified</li>
  <li>You'll receive a response within 2 business hours</li>
  <li>Please reference your tracking number in any follow-up communications</li>
</ul>

<p>If you need urgent assistance, please contact the IT helpdesk directly at: <strong>itops@yourcompany.com</strong></p>

<hr>

<p><em>This is an automated email from the M365 FAQ Agent. Please do not reply to this email.</em></p>
```

Now replace the placeholders:
1. Delete `[userQuestion]` and insert the **userQuestion** dynamic token
2. Delete the first `[ticketRef]` and insert the **ticketRef** dynamic token (appears twice, do both)

**Important:** Update the IT helpdesk email address (`itops@yourcompany.com`) to your actual IT support email.

Rename this action to `Email-ConfirmToUser`.

---

**Step 7 — Log the escalation to SharePoint**

Click **New step** and search for **Create item**.

Choose the one that says **SharePoint** underneath it.

Fill in the fields:

**Site Address:** Type or paste your site URL (replace with your actual site):
```
https://yourtenant.sharepoint.com/sites/YourSiteName
```

**List Name:** Click the dropdown and select **FAQ_Analytics**

Now you need to fill in the column values. Click **Show all** to see all available columns.

**Title:** Click **Add dynamic content** → select **userQuestion**

**QueryDate:** Click the **fx** button and paste this expression:
```
utcNow()
```
Click **OK**.

**Product:** Click **Add dynamic content** → select **product**

**ResolutionStatus:** Type: `escalated`

**UserEmail:** Click **Add dynamic content** → select **userEmail**

**EscalationTicketRef:** Click **Add dynamic content** → select **ticketRef** from the Var-ticketRef section

Leave the other fields blank (ConfidenceScore, MatchedFAQItemId, MatchedFAQTitle) — they only apply to resolved queries, not escalations.

Rename this action to `SharePoint-LogEscalation`.

---

**Step 8 — Return the ticket reference to the agent**

This is critical. Without this step, the agent won't know the escalation succeeded and won't be able to tell the user the ticket number.

Click **New step** and search for **Response**.

Choose the one that says **Request** underneath it.

Fill in the fields:

**Status code:** Select **200** from the dropdown

**Body:** Click in the field, then click the **fx** button. Paste this expression:

```
json(concat('{"ticketRef":"', variables('ticketRef'), '","status":"escalated"}'))
```

Click **OK**.

**Response Headers:** Click **Add new item**. A row appears with Key and Value fields.

- Key: type `Content-Type`
- Value: type `application/json`

Rename this action to `Response-ReturnTicket`.

Click **Save** in the top right corner.

---

**Your EscalateToTeams flow is complete. It should have 8 actions:**

1. Trigger-CopilotStudioCall (with 4 inputs: userQuestion, userEmail, product, conversationId)
2. Var-ticketRef
3. Teams-PostEscalationCard
4. Email-NotifyOpsTeam
5. Email-ConfirmToUser
6. SharePoint-LogEscalation
7. Response-ReturnTicket

---

**Test the flow:**

Click **Test** in the top right. Choose **Manually**. Click **Test** again.

The test panel opens. Fill in test values:

- userQuestion: `How do I share a file externally?`
- userEmail: `test.user@yourdomain.com` (use YOUR actual email to receive the test email)
- product: `SharePoint`
- conversationId: `test-123`

Click **Run flow**.

**Check these 4 things:**

1. ✅ **Teams channel** (#ops-faq-escalations) - You should see the Adaptive Card with ticket number and question
2. ✅ **IT ops team inbox** - Check the email inbox for your IT ops email address
3. ✅ **User inbox** - Check your test email inbox for the confirmation email with tracking number
4. ✅ **SharePoint FAQ_Analytics list** - You should see a new item logged with ResolutionStatus = "escalated" and the ticket reference

If all 4 appear correctly, the flow works!

---

**What you've completed in Section 3:**

✅ EscalateToTeams flow created with 8 actions  
✅ Unique ticket reference generation  
✅ Teams channel notification with Adaptive Card  
✅ Email notification to IT ops team  
✅ Email confirmation to user with tracking number  
✅ Escalation logging to SharePoint  
✅ Response back to Copilot Studio with ticket reference  
✅ End-to-end testing completed  

**Next:** Section 4 - Connect the EscalateToTeams flow to Copilot Studio agent topics

---

## Section 4 — Connect EscalateToTeams Flow to Copilot Studio Agent

Now that you've built the EscalateToTeams Power Automate flow, you need to connect it to your Copilot Studio agent so the agent can call it when it can't find a confident answer.

This section covers:
1. Registering the flow as an Action in Copilot Studio
2. Adding escalation logic to all 6 product topics
3. Testing the complete end-to-end escalation flow

---

### 4.1 Make the EscalateToTeams flow available to your agent

Before you can call the Power Automate flow from your topics, you need to ensure it's in the correct environment and solution.

**CRITICAL:** The Power Automate flow MUST be in the SAME environment as your Copilot Studio agent. This is the most common reason flows don't appear.

---

**Step 1 — Check your Copilot Studio agent's environment**

Go to **copilotstudio.microsoft.com** and open your **M365 FAQ Agent**.

Look at the top right corner of the screen. You'll see the **environment name** (e.g., "Production", "Development", "Contoso (default)", etc.).

**Write down this environment name** — this is where your flow needs to be.

---

**Step 2 — Move your flow to the correct environment (if needed)**

Go to **make.powerautomate.com**.

Check the environment selector in the top right corner. Does it match your Copilot Studio environment?

**If NO (environments don't match):**

You need to recreate the flow in the correct environment:

1. In Power Automate, click the environment selector (top right)
2. Switch to the SAME environment as your Copilot Studio agent
3. Create a new solution in this environment: Click **Solutions** → **+ New solution**
   - Display name: `M365FAQAgent`
   - Publisher: Select your organization's publisher or create a new one
   - Click **Create**
4. Open your new solution
5. Recreate the EscalateToTeams flow following Section 3 instructions
6. Make sure to use the **"When an agent calls the flow V2"** trigger (NOT V1, NOT "When Copilot Studio calls a flow")

**If YES (environments match):**

Your flow is in the right place. Continue to Step 3.

---

**Step 3 — Ensure the flow uses the correct trigger**

Open your **EscalateToTeams** flow in Power Automate.

Look at the first step (the trigger). It MUST say one of these:
- ✅ **"When an agent calls the flow V2"** (CORRECT - this is the new trigger)
- ❌ "When Copilot Studio calls a flow" (OLD - won't work)
- ❌ "PowerApps (V2)" (WRONG trigger entirely)

If you have the wrong trigger:
1. Delete the flow
2. Create a new flow
3. When choosing the trigger, search for: **"When an agent calls the flow"**
4. Make sure you select the **V2** version
5. Rebuild the flow following Section 3

---

**Step 4 — Turn ON the flow**

In Power Automate, make sure your flow is:
- ✅ **Saved**
- ✅ **Turned ON** (not in draft mode)

To turn it on:
1. Open the flow
2. Click **Turn on** in the top right (if it says "Turn off", it's already on)

---

**Step 5 — Wait a few minutes and refresh**

After ensuring the flow is:
- In the correct environment
- Using the "When an agent calls the flow V2" trigger  
- Saved and turned ON

Wait 2-3 minutes, then refresh your Copilot Studio browser tab.

The flow should now be available to call from your topics.

---

**Troubleshooting: Flow still not appearing**

If the flow still doesn't appear in your topics:

1. **Check solution ownership:** In Power Automate → Solutions, make sure the solution containing your flow is in the same environment
2. **Check flow status:** The flow must show "On" status, not "Draft" or "Off"
3. **Check permissions:** Make sure you have edit access to both the Copilot Studio agent and the Power Automate flow
4. **Try the manual trigger test:** In Power Automate, open the flow and click Test → Manually → Test. If it asks for inputs, the trigger is correct.
5. **Clear browser cache:** Sometimes Copilot Studio caches the available flows. Try Ctrl+F5 to hard refresh.
6. **Contact support:** If nothing works, the environment may have a configuration issue

---

### 4.2 Add escalation logic to product topics

Each product topic handles escalation in TWO ways:

**Scenario 1 — Automatic:** Generative answers finds NO answer → condition detects blank → calls flow immediately
**Scenario 2 — Manual:** User gets an answer but clicks "Escalate to IT team" from the "What's next?" menu

Both are handled inside the product topic itself. We already know the product (e.g. "Microsoft Teams") so we never need to ask again.

We'll configure **TeamsQuestions** first. Repeat for the other 5 topics changing only the product name.

---

**Step 1 — Open the TeamsQuestions topic**

Go to **Topics** → **Custom** tab → Click **TeamsQuestions**.

---

**Step 2 — Add condition directly after Generative answers**

Click the **+** button directly below the **Generative answers** node → **Add a condition**.

Set the condition:
- Variable: **System** → **Activity** → **Text**
- Operator: **is equal to**
- Value: leave **empty/blank**

This detects when Generative answers returned nothing (no answer found).

---

**Step 3 — TRUE branch: Automatic escalation (no answer found)**

In the **TRUE branch**, click **+** → **Call an action** → select **EscalateToTeams**.

Fill in the 4 inputs:

| Input                   | Value                              |
| ----------------------- | ---------------------------------- |
| text (userQuestion)     | `=Topic.VarUserQuestion`           |
| text_1 (userEmail)      | `=System.User.Email`               |
| text_2 (product)        | `Microsoft Teams` (type literally) |
| text_3 (conversationId) | `=System.Conversation.Id`          |

After the flow action, click **+** → **Send a message**:

```
I couldn't find a confident answer to your question, so I've escalated it to our IT operations team.

Your tracking number is: {Topic.ticketRef}

You'll receive a confirmation email shortly. The IT ops team will respond within 2 business hours.

Thank you for using M365 FAQ Agent!
```

Then click **+** → **End dialog**.

---

**Step 4 — FALSE branch: Answer found → show What's next**

In the **FALSE (else) branch**, add:

Click **+** → **Ask a question** → **Multiple choice**

Question: `What would you like to do next?`

Add 4 options:
- `Another Teams question`
- `Different product`
- `Escalate to IT team`
- `I'm done`

Save to variable: `WhatNext`

---

**Step 5 — Add conditions for each What's next choice**

Click **+** → **Add a condition** with 4 branches:

**Branch 1 — Another Teams question:**
- Condition: `WhatNext` = `Another Teams question`
- Action: **Repeat this topic**

**Branch 2 — Different product:**
- Condition: `WhatNext` = `Different product`
- Action: **Go to another topic** → **Greeting**

**Branch 3 — Escalate to IT team (manual escalation):**
- Condition: `WhatNext` = `Escalate to IT team`
- Action: **Call an action** → **EscalateToTeams**

Fill in the same 4 inputs as Step 3, then **Send a message** with tracking number, then **End dialog**.

**Branch 4 — I'm done:**
- Condition: `WhatNext` = `I'm done`
- **Send a message**: `Thank you for using M365 FAQ Agent. Have a great day!`
- **End dialog**

Click **Save**.

---

**Your complete TeamsQuestions flow:**

```
Ask: "What's your question about Microsoft Teams?" → VarUserQuestion
  ↓
Generative answers (searches Teams FAQs only)
  ↓
CONDITION: System.Activity.Text = blank?
  │
  ├── TRUE (no answer found)
  │     Call EscalateToTeams
  │       text:  VarUserQuestion
  │       text_1: System.User.Email
  │       text_2: "Microsoft Teams"
  │       text_3: System.Conversation.Id
  │     Show message with {Topic.ticketRef}
  │     End dialog
  │
  └── FALSE (answer found)
        Ask: "What would you like to do next?"
          ├── Another Teams question → Repeat topic
          ├── Different product → Greeting
          ├── Escalate to IT team → Call flow → Show ticketRef → End
          └── I'm done → Goodbye → End
```

---

### 4.3 Sample YAML — TeamsQuestions topic

Paste this directly into the TeamsQuestions topic code view. Update `flowId` and `knowledgeSources` reference to match your environment.

```yaml
kind: AdaptiveDialog
modelDescription: This topic handles questions about Microsoft Teams. It is triggered when the user selects Teams from the product menu and asks their question about Teams features, channels, meetings, or guest access.
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent: {}
  actions:
    - kind: Question
      id: question_Eb49t1
      interruptionPolicy:
        allowInterruption: true
      variable: init:Topic.VarUserQuestion
      prompt: What's your question about Microsoft Teams?
      entity: StringPrebuiltEntity
    - kind: SearchAndSummarizeContent
      id: generativeAnswers_Teams
      userInput: =Topic.VarUserQuestion
      fileSearchDataSource:
        searchFilesMode:
          kind: DoNotSearchFiles
      knowledgeSources:
        kind: SearchSpecificKnowledgeSources
        knowledgeSources:
          - cra93_M365FAQAgent.topic.FAQ_Teams_qOrl7DNZzVjSYHZ6fBJy5
      inputType: {}
      outputType: {}
    - kind: ConditionGroup
      id: conditionGroup_AnswerCheck
      conditions:
        - id: condition_NoAnswer
          condition: =System.Activity.Text = Blank()
          actions:
            - kind: InvokeFlowAction
              id: invokeFlowAction_AutoEscalate
              input:
                binding:
                  text: =Topic.VarUserQuestion
                  text_1: =System.User.Email
                  text_2: Teams
                  text_3: =System.Conversation.Id
              output:
                binding:
                  text: Topic.Text
                  ticketref: Topic.ticketRef
              flowId: 1bcd942d-a94b-f111-bec5-70a8a59be063
            - kind: SendActivity
              id: sendAutoEscalationMessage
              activity: |-
                I couldn't find a confident answer to your question, so I've escalated it to our IT operations team.

                Your tracking number is: {Topic.ticketRef}

                You'll receive a confirmation email shortly. The IT ops team will respond within 2 business hours.

                Thank you for using M365 FAQ Agent!
            - kind: EndDialog
              id: endDialog_AutoEscalation
      elseActions:
        - kind: Question
          id: question_WhatNext
          displayName: WhatNext
          interruptionPolicy:
            allowInterruption: true
          variable: Topic.WhatNext
          prompt: What would you like to do next?
          entity:
            kind: EmbeddedEntity
            sensitivityLevel: None
            definition:
              kind: ClosedListEntity
              items:
                - id: Another Teams question
                  displayName: Another Teams question
                - id: Different product
                  displayName: Different product
                - id: Escalate to IT team
                  displayName: Escalate to IT team
                - id: I'm done
                  displayName: I'm done
        - kind: ConditionGroup
          id: conditionGroup_NextAction
          conditions:
            - id: condition_SameProduct
              condition: =Topic.WhatNext = 'cra93_M365FAQAgent.topic.TeamsTopic.main.question_WhatNext'.'Another Teams question'
              actions:
                - kind: RepeatDialog
                  id: repeatDialog_AskAgain
            - id: condition_DifferentProduct
              condition: =Topic.WhatNext = 'cra93_M365FAQAgent.topic.TeamsTopic.main.question_WhatNext'.'Different product'
              actions:
                - kind: BeginDialog
                  id: redirectToGreeting
                  dialog: cra93_M365FAQAgent.topic.Greeting
            - id: condition_Escalate
              condition: =Topic.WhatNext = 'cra93_M365FAQAgent.topic.TeamsTopic.main.question_WhatNext'.'Escalate to IT team'
              actions:
                - kind: InvokeFlowAction
                  id: invokeFlowAction_ManualEscalate
                  input:
                    binding:
                      text: =Topic.VarUserQuestion
                      text_1: =System.User.Email
                      text_2: Teams
                      text_3: =System.Conversation.Id
                  output:
                    binding:
                      text: Topic.Text
                      ticketref: Topic.ticketRef
                  flowId: 1bcd942d-a94b-f111-bec5-70a8a59be063
                - kind: SendActivity
                  id: sendManualEscalationMessage
                  activity: |-
                    I've escalated your question to our IT operations team.

                    Your tracking number is: {Topic.ticketRef}

                    You'll receive a confirmation email shortly. The IT ops team will respond within 2 business hours.

                    Thank you for using M365 FAQ Agent!
                - kind: EndDialog
                  id: endDialog_ManualEscalation
            - id: condition_Done
              condition: =Topic.WhatNext = 'cra93_M365FAQAgent.topic.TeamsTopic.main.question_WhatNext'.'I''m done'
              actions:
                - kind: SendActivity
                  id: sendGoodbye
                  activity: Thank you for using M365 FAQ Agent. Have a great day!
                - kind: EndDialog
                  id: endDialog_Goodbye
```

---

### 4.4 Repeat for the other 5 product topics

The **only things that change** per topic:

| Topic                  | text_2 (product) | "Another X question" label        | Knowledge source            |
| ---------------------- | ---------------- | --------------------------------- | --------------------------- |
| SharePointQuestions    | `SharePoint`     | `Another SharePoint question`     | FAQ_SharePoint reference    |
| PowerAppsQuestions     | `Power Apps`     | `Another Power Apps question`     | FAQ_PowerApps reference     |
| PowerAutomateQuestions | `Power Automate` | `Another Power Automate question` | FAQ_PowerAutomate reference |
| PowerPlatformQuestions | `Power Platform` | `Another Power Platform question` | FAQ_PowerPlatform reference |
| OtherToolsQuestions    | `Other`          | `Another question`                | FAQ_NonM365 reference       |

Everything else in the YAML is identical.

---

### 4.5 Test the complete escalation flow

**Test 1 — Automatic escalation (no answer found)**

1. Click **Test** in Copilot Studio
2. Click **Microsoft Teams**
3. Ask a nonsense question: `How do I configure quantum entanglement in Teams?`
4. Expected:
   - Generative answers finds nothing → condition TRUE
   - Flow called automatically — no extra questions asked
   - Shows: "Your tracking number is: OPS-202605091545"
   - Conversation ends
5. Verify: ✅ Teams card ✅ Ops email ✅ User email ✅ SharePoint log

---

**Test 2 — Manual escalation (user chooses to escalate)**

1. Restart test chat
2. Click **Microsoft Teams**
3. Ask a real question: `How do I add a guest to a channel?`
4. Agent returns the answer
5. Shows "What would you like to do next?" with 4 options
6. Click **Escalate to IT team**
7. Expected:
   - Flow called
   - Shows tracking number
   - Conversation ends
8. Verify: ✅ Teams card ✅ Ops email ✅ User email ✅ SharePoint log

---

**Test 3 — Normal flow (no escalation)**

1. Restart test chat
2. Click **Microsoft Teams**
3. Ask: `How do I add a guest to a channel?`
4. Click **Another Teams question**
5. Expected: topic repeats, no escalation triggered

---

**Troubleshooting:**

**Escalation doesn't trigger automatically:**
- Check condition is `System.Activity.Text = Blank()` placed directly after Generative answers node
- Confirm it's in the TRUE branch, not elseActions

**Flow not found in "Call an action":**
- Verify flow is in same environment as agent
- Verify flow is turned ON in Power Automate
- Refresh browser (Ctrl+F5), wait 2-3 minutes

**Tracking number shows blank:**
- Check output binding: `ticketref: Topic.ticketRef`
- Confirm flow output is named `ticketRef` in Power Automate
- Check flow run history in Power Automate to confirm it executed

**System.User.Email is empty in test:**
- Expected in test mode — will populate correctly when deployed to Teams
- For testing, temporarily hardcode a test email in text_1

---

**What you've completed in Section 4:**

✅ Automatic escalation when Generative answers finds no answer
✅ Manual escalation option in "What's next?" menu
✅ Both escalation paths call EscalateToTeams with correct inputs
✅ Product name passed correctly — user never asked twice
✅ Tracking number shown to user in both scenarios
✅ Teams notifications, emails, and SharePoint logging all working
✅ All 6 product topics configured with sample YAML
✅ Complete end-to-end testing successful

---

## 🎉 Congratulations!

**Your M365 FAQ Agent is now complete and ready to deploy to Microsoft Teams!**

**What you've built:**
- ✅ AI-powered FAQ agent with 6 product categories
- ✅ SharePoint knowledge bases with semantic search
- ✅ Automatic escalation when no answer is found
- ✅ Manual escalation option for users who want human help
- ✅ Teams channel notifications with Adaptive Cards
- ✅ Email notifications to IT ops and users
- ✅ Tracking numbers for all escalations
- ✅ SharePoint logging for analytics
- ✅ Seamless conversation flow with product switching

**Next steps (optional):**
- Deploy the agent to Microsoft Teams for your organization
- Set up Power BI dashboard for FAQ analytics
- Configure automated stale content detection
- Add multilingual support
- Implement sentiment analysis for user satisfaction tracking

---

**End of Complete Build Guide**