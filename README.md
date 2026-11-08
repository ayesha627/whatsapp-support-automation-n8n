# WhatsApp Customer Support Automation (n8n)

An n8n workflow that answers customer messages on WhatsApp automatically, logs every conversation, and alerts a human agent when needed.

**Author:** Ayesha Imran  
**Organization:** Logitrix Solutions  
**Supervisor:** Mohsin

## What it does

1. Receives customer messages from the Meta WhatsApp Business Cloud API (webhook trigger).
2. Ignores delivery/read receipts and asks for text if the customer sends a non-text message.
3. Routes each message by keyword:

| Category | Keywords | Priority | Emails human |
|---|---|---|---|
| Human support | human, agent, urgent, representative, person | High | Yes |
| Refund / billing | refund, billing, return, cancel | High | Yes |
| Complaint | complaint, problem, issue, broken, not working, disappointed | High | Yes |
| Working hours | hours, open, close, timing, working time | Low | No |
| Order / delivery | order, delivery, shipping, track | Medium | No |
| Other | anything else | Low / Medium | No |

4. For "other" questions, Google Gemini answers using **only** the company information. If it cannot answer (`NO_ANSWER`), the customer gets a "ticket created" reply.
5. Sends the reply through WhatsApp.
6. Logs customer name, message, detected intent, reply and timestamp to Google Sheets.
7. Emails the support team for high-priority cases.

## Tech stack

n8n, Meta WhatsApp Business Cloud API, Google Gemini, Google Sheets, Gmail.

## Setup

1. Import `Final WhatsApp Chat Support Automation.json` into n8n.
2. Create credentials and select them in the nodes:
   - **WhatsApp Trigger:** WhatsApp OAuth (Meta App ID and App Secret).
   - **Send WhatsApp Reply:** WhatsApp Business Cloud (Access Token and Business Account ID). Use a System User token so it does not expire after 24 hours.
   - **Answer From Company Info:** Google Gemini API key.
   - **Log Conversation:** Google Sheets OAuth, with a sheet that has the columns `customer_name`, `customer_message`, `detected_intent`, `automatic_reply`, `timestamp`.
   - **Notify Support Team:** Gmail OAuth, and set the recipient email.
3. Edit `company_info` in the **Extract Message Fields** node with your real FAQ content.
4. In Meta for Developers, publish the app (Live mode) and subscribe the webhook to the `messages` field.
5. Publish the n8n workflow, then send a WhatsApp message to the business number to test (do not use "Execute workflow" while the workflow is active; a Meta app allows only one trigger).

## Notes

- Free-text replies work only within 24 hours of the customer's last message (WhatsApp rule).
- Never commit access tokens or app secrets. The exported JSON contains only credential IDs and names, not secrets.
