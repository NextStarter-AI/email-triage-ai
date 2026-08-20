# Installation Manual — AI Email Triage & Auto-Response

## What this template does

This workflow automatically reads incoming emails in your inbox, uses Artificial Intelligence to understand what each one is about (Support, Sales, Finance, or Other), and decides what to do next:

- If it's a simple support question and the AI has high confidence in the answer → it **replies automatically**.
- If it's a support question but the AI isn't confident → it **notifies your team on Slack** for manual review.
- If it's Sales or Finance → it **notifies the right Slack channel** with a summary of the email.
- If it can't classify the email → it falls into a safety category, notifying the general team.

This reduces response time and prevents important emails from sitting unanswered in the inbox.

---

## Step 1 — Prepare Gmail

1. Create two labels in Gmail: **"AI-Replied"** and **"Needs-Review"**.
2. Make sure the connected account has read and send permissions (this is standard with Gmail's OAuth authentication).

## Step 2 — Prepare Slack

1. Create (or use existing) channels to receive notifications — for example: `#support`, `#sales`, `#finance`, `#general`.
2. Add the Slack bot (which you'll connect in Step 4) to these channels.

---

## Step 3 — Import the workflow into n8n

1. In n8n, click **"+ Add workflow"**.
2. Click the three dots (**⋯**) → **"Import from File"**.
3. Select the `email-triage-ai.json` file.

---

## Step 4 — Connect the credentials

**Gmail:**
1. Double-click the **"New Email Received"** node.
2. Under Credential → "Create New" → sign in with the Gmail account you want to monitor.
3. Repeat for the **"Reply Automatically"** node — same credential.

**OpenAI:**
1. Double-click the **"Classify Email with AI"** node.
2. Under Credential → "Create New" → paste your OpenAI API Key (generated at platform.openai.com/api-keys).
3. The default model configured is `gpt-4o-mini` (good cost-to-performance ratio). You can switch to another model if you need higher accuracy.

**Slack:**
1. Double-click any **"Notify..."** node (there are four: Support, Sales, Finance, General Team).
2. Under Credential → "Create New" → connect via OAuth with your Slack workspace.
3. **Important:** in each of the 4 Slack nodes, replace the placeholder text `REPLACE_SUPPORT_CHANNEL`, `REPLACE_SALES_CHANNEL`, `REPLACE_FINANCE_CHANNEL`, and `REPLACE_GENERAL_CHANNEL` with the actual channel (select it from the field's dropdown list).

---

## Step 5 — Adjust the confidence threshold (optional)

By default, the workflow only auto-replies when the AI has **80% confidence or higher**. To change this:

1. Open the **"Validate AI Response"** node.
2. Find the line `confiancaAlta: confianca >= 0.8` in the code.
3. Replace `0.8` with the value you want (e.g., `0.9` to be more conservative).

---

## Step 6 — Test the workflow

1. Send a simple test email to the connected inbox (e.g., "What are your business hours?").
2. Run the workflow manually in n8n (the "Execute Workflow" button, or the play icon on the first node).
3. Check that:
   - The "Classify Email with AI" node returned a coherent category.
   - If confidence was high, the reply arrived in the test mailbox.
   - If confidence was low, the notification arrived in the correct Slack channel.
4. Send a second test email clearly related to "Sales" or "Finance" to confirm the routing works.
5. Everything working? Click **"Active"** to let it run automatically.

---

## Important precautions before going live

⚠️ **Recommendation:** for the first few weeks, keep the confidence threshold high (0.85+) and closely monitor the automatic replies. An incorrectly answered email can create problems with a client — it's better to escalate too much to a human at first and fine-tune the model over time.

## FAQ

**Can the AI reply with something wrong?**
Yes, that's why the confidence field exists. Adjust the threshold (Step 5) to match the safety level your operation requires.

**Can I add more categories (e.g., HR, Legal)?**
Yes. Edit the `CATEGORIAS_VALIDAS` list in the "Validate AI Response" node, the prompt in the "Classify Email with AI" node, and add a new output in the "Route by Category" node.

*Note: the internal category values in the code (`Suporte`, `Vendas`, `Financeiro`, `Outro`) are still in Portuguese — this is just internal logic and doesn't affect what the AI or the workflow does. If you want the code itself fully in English, rename those values consistently in the "Classify Email with AI" prompt and the "Validate AI Response" node together, so they still match each other.*

**Does the workflow reply to emails in any language?**
Yes, the AI model detects and replies in the same language as the received email, as long as the prompt isn't manually restricted to a specific language.
