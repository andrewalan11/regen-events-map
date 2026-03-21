# Instructions for Bonfires Bot
## Task: Forward event links from Telegram to our Regen Events Map database

### What you need to do

When someone shares a URL in our Telegram group that looks like a link to an event, gathering, festival, retreat, or conference, you should forward that link to our database by making an HTTP POST request.

### When to trigger

Forward a link when a message contains a URL that appears to be about an event, gathering, festival, conference, retreat, convergence, summit, or similar. Use your judgment — we'd rather have too many suggestions than miss one. Skip obvious non-event links (news articles, social media profiles, general websites, etc.).

### The API call to make

**Method:** POST

**URL:**
```
https://aurftdjjghtzeofqzezx.supabase.co/rest/v1/suggestions
```

**Headers:**
```
Content-Type: application/json
apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo
```

(Both apikey and Authorization use the same token value.)

**Body (JSON):**
```json
{
  "url": "<the event URL from the message>",
  "submitted_by": "<telegram username or display name of the person who shared it>",
  "source": "telegram",
  "notes": "<any surrounding text from the message that gives context, or null>"
}
```

Only `url` is required. The other fields are optional but helpful.

### How to handle responses

- **201** — success, the suggestion was saved. You can optionally reply in the group: "🗺️ Added to the Regen Events Map queue!"
- **409** — this URL was already submitted (duplicate). You can optionally reply: "This event was already suggested for the map." Or just silently ignore it.
- **400** — something was wrong with the request (missing URL field). Log it for debugging.
- **Any other error** — silently ignore, don't spam the group.

### Example

If someone posts: "Check out the European Ecovillage Gathering! https://ecovillagegathering.org It's in Bavaria this July"

You would POST:
```json
{
  "url": "https://ecovillagegathering.org",
  "submitted_by": "andrea_regen",
  "source": "telegram",
  "notes": "European Ecovillage Gathering, Bavaria, July"
}
```

### Important notes

- The database automatically rejects duplicate URLs, so don't worry about sending the same link twice — it's safe.
- A human reviews all suggestions before they appear on the map, so false positives are fine.
- The map is at: https://andrewalan11.github.io/regen-events-map
- This is a community tool for the regenerative economy ecosystem. Thank you for helping keep it alive!
