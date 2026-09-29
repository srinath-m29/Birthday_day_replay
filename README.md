# Instagram Birthday Auto Reply (n8n)

An n8n workflow that automatically handles birthday wishes on Instagram.

## What it does

| Trigger | Action |
|---|---|
| Birthday DM | Reacts with a heart and replies with a gender-based message |
| Story mention | Replies by DM with the same message, then reposts the mentioned story to your story |
| Birthday comment on a post | Replies to the comment with the same message |

**Reply text**
- Boy-named account: `Thank you da 😊`
- Girl-named account: `Thank you 😊`
- Unknown or low-confidence name: `Thank you 😊`

Birthday wishes are detected by keywords: birthday, bday, hbd, many more returns, 🎂 🎉 🥳 🎈 🎁.

## Requirements

- An n8n instance reachable from the internet (n8n Cloud or self-hosted with HTTPS)
- An Instagram **Business or Creator** account
- A Meta developer app with the Instagram API (Instagram Login) enabled
- An Instagram access token and your Instagram Business account ID

## Setup

1. **Import** `instagram-birthday-autoreply.json` in n8n: Workflows → Import from File.
2. Open the **Config** node and fill in:
   - `accessToken`: your Instagram access token
   - `igUserId`: your Instagram Business account ID
3. **Activate** the workflow so the production webhook URL is live.
4. In the Meta developer dashboard, go to Instagram → Webhooks:
   - Callback URL: `https://YOUR-N8N-DOMAIN/webhook/instagram-birthday`
   - Verify token: any value (the workflow answers the verification challenge automatically)
   - Subscribe to the **messages** and **comments** fields.
5. Make sure your app has these permissions: `instagram_business_basic`, `instagram_business_manage_messages`, `instagram_business_manage_comments`, `instagram_business_content_publish`.
6. Send a test birthday DM from another account and check the n8n Executions tab.

## How it works

1. Two webhook nodes share one path: GET answers Meta's verification, POST receives events.
2. **Parse Event** filters out your own messages and non-birthday text, and labels each event as `dm`, `story_mention` or `comment`.
3. **Get Profile** fetches the sender's name, **Extract First Name** cleans it, and **Genderize** (genderize.io) guesses gender. A guess needs at least 60% confidence to count as male.
4. **Build Reply** picks the reply text.
5. **Route by Type** sends each event down its own branch.

## Limitations

- The API only supports the standard ❤️ ("love") reaction. A white heart 🤍 cannot be sent.
- Gender is a guess from the first name. Nicknames and stylised usernames can be wrong.
- DM replies only work within 24 hours of the user's message.
- Instagram has no comments on stories, so story mentions get a DM reply instead.
- Comment replies use the commenter's username for the name guess, which is less accurate than DMs.
- The story repost reuses the mention's media link. Instagram may sometimes refuse to fetch it, and the workflow has not been tested against a live account.
- genderize.io's free tier is limited to about 1000 requests per day.
- Tokens expire (long-lived tokens last about 60 days), so refresh yours before then.

## Customising

- **Reply text:** edit the **Build Reply** code node.
- **Keywords:** edit the `bday` regex in the **Parse Event** code node.
- **Confidence threshold:** change `p >= 0.6` in **Build Reply**.
- **Fixed names for friends:** add an override object in **Build Reply** that maps a name to male or female before the genderize result is used.
