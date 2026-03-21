# Make.com Setup: Telegram → Supabase Event Pipeline

## What this does
Automatically watches your Telegram group for messages containing URLs. When someone shares a link, it extracts the URL and sends it to your Supabase `suggestions` table. A human reviews suggestions and promotes the good ones to the events map.

## Prerequisites
- A Make.com account (free tier works — 1,000 operations/month)
- A Telegram Bot Token (you'll create one in 2 minutes)
- Your Supabase details (you already have these)

---

## Step 1: Create a Telegram Bot (2 min)

1. Open Telegram and message **@BotFather**
2. Send: `/newbot`
3. Give it a name: `Regen Events Collector` (or whatever you want)
4. Give it a username: `regen_events_collector_bot` (must end in `bot`)
5. BotFather gives you a **Bot Token** — save it, you'll need it in Make.com
6. **Add this bot to your Telegram group** (the one where people share event links)
7. Make the bot an admin of the group (so it can read messages)

## Step 2: Create the Make.com Scenario (3 min)

1. Go to **make.com** and sign up / log in
2. Click **"Create a new scenario"**

### Module 1: Telegram — Watch Updates

1. Click the **+** button and search for **"Telegram Bot"**
2. Select **"Watch Updates"**
3. Connect your bot: paste the Bot Token from Step 1
4. Set **Allowed Updates** to: `message`
5. Set **Limit** to: `10`
6. Click OK

### Module 2: Filter — Only messages with URLs

1. Click the **wrench icon** between Module 1 and the next module
2. Add a **Filter** with the condition:
   - **Label:** "Has URL"
   - **Condition:** `Message.Text` **matches pattern** (regex): `https?://[^\s]+`
3. This ensures only messages containing links get forwarded

### Module 3: HTTP — POST to Supabase

1. Click **+** and search for **"HTTP"**
2. Select **"Make a request"**
3. Configure:

**URL:**
```
https://aurftdjjghtzeofqzezx.supabase.co/rest/v1/suggestions
```

**Method:** POST

**Headers:**
| Key | Value |
|-----|-------|
| Content-Type | application/json |
| apikey | eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo |
| Authorization | Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo |
| Prefer | return=minimal |

**Body type:** Raw

**Content type:** JSON (application/json)

**Request content:** (use this template — Make.com lets you map fields from the Telegram module)
```json
{
  "url": "{{extract the URL from Message.Text using regex}}",
  "submitted_by": "{{Message.From.Username}}",
  "source": "telegram",
  "notes": "{{Message.Text}}"
}
```

**To extract just the URL from the message text**, use Make.com's `matchAll` function:
- For the `url` field, use: `{{first(matchAll(1.message.text; "/https?://[^\s]+/"))}}`
- Or simpler: just send the full message text as `notes` and put a placeholder for URL — you can refine later

**Important:** Under "Error handling," add a **Resume** directive. This way if Supabase returns 409 (duplicate URL), Make.com won't stop the scenario — it just skips that one.

## Step 3: Activate

1. Click **"Run once"** to test with a real message in your group
2. Verify the suggestion appeared in Supabase (check the `suggestions` table)
3. If it works, toggle the scenario **ON**
4. Set the schedule to **"Every 15 minutes"** (or however often you want)

---

## How duplicates are handled
The Supabase `suggestions` table has a unique constraint on the `url` column. If the same link is shared twice, the second POST gets a 409 Conflict response. Make.com's error handler (Resume) just skips it. No duplicates in your database.

## What happens next
Suggestions sit in the `suggestions` table with status="pending". In your Cowork sessions with Claude, you can review pending suggestions and Claude will add the good ones to the events map with full details (location, dates, tags, coordinates, etc.).

## Optional: Bot reply in group
If you want the bot to reply in the group confirming the suggestion was received, add a 4th module:
- **Telegram Bot → Send a message**
- Chat ID: `{{Message.Chat.ID}}`
- Text: `🗺️ Link added to the Regen Events Map suggestion queue!`
- Reply to Message ID: `{{Message.MessageID}}`
