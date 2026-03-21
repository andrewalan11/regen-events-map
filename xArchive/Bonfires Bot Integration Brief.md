# Bonfires Bot Integration Brief
## Regen Events Map — Telegram Event Pipeline

### What we need

When someone shares a link to an event in the Telegram group, the bot should forward that link to our database so it can be reviewed and added to our community events map.

### How it works

The bot makes a single HTTP POST request to our Supabase database whenever it detects an event link being shared.

### The API call

**Endpoint:**
```
POST https://aurftdjjghtzeofqzezx.supabase.co/rest/v1/suggestions
```

**Headers (required):**
```
Content-Type: application/json
apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo
```

**Body (JSON):**
```json
{
  "url": "https://example.com/some-event-page",
  "submitted_by": "username_of_person_who_shared",
  "source": "telegram",
  "notes": "any additional context from the message"
}
```

**Required fields:** `url` (the link)
**Optional fields:** `submitted_by`, `source`, `notes`

### Example with curl

```bash
curl -X POST 'https://aurftdjjghtzeofqzezx.supabase.co/rest/v1/suggestions' \
  -H 'Content-Type: application/json' \
  -H 'apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImF1cmZ0ZGpqZ2h0emVvZnF6ZXp4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM5OTA4MzUsImV4cCI6MjA4OTU2NjgzNX0.PeOisRNczOCYOgyiKL8lNDA0ZNiwhzBhqQWDYx6DtMo' \
  -d '{"url":"https://ecovillagegathering.org","submitted_by":"andrew","source":"telegram","notes":"European Ecovillage Gathering"}'
```

### What happens with duplicates

The database has a unique constraint on the `url` field. If the same URL is submitted twice, the second submission will be rejected with a `409 Conflict` response. The bot can safely ignore this — it just means the event was already suggested.

### What happens after submission

Submissions go into a "pending" queue. A human reviews them and either approves (adds the event to the map with full details) or rejects them. The map at https://andrewalan11.github.io/regen-events-map shows only approved events.

### Responses

- **201 Created** — suggestion submitted successfully
- **409 Conflict** — duplicate URL (already in the database)
- **400 Bad Request** — missing required `url` field

### Bot behavior suggestions

1. **Trigger:** When a message contains a URL that looks like an event page (not general chat links)
2. **What to send:** The URL + the Telegram username of the person who shared it + any text from the message as notes
3. **Confirmation:** Optionally reply in the group: "Added to the Regen Events Map suggestion queue"
4. **Duplicate handling:** If the API returns 409, optionally reply: "This event was already suggested"
