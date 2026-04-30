---
name: discord-forum-thread
description: Create Discord forum threads via the bot API. Uses curl to call Discord's REST API directly with the bot token from .env.
---

# Create Discord Forum Thread

Creates a forum thread in a Discord forum channel using the Discord REST API. 

**Alternative (preferred by user):** OpenClaw is installed at `/home/msns/.nvm/versions/node/v22.16.0/bin/openclaw` and can create forum threads via `openclaw message send` targeting a forum channel ID. First line = title, remainder = body. This is preferred over curl+python because it doesn't trigger terminal approval prompts.

## Prerequisites

- `DISCORD_BOT_TOKEN` must be set in `~/.hermes/.env`
- The bot must have permission to create threads in the target channel
- The target channel must be type 15 (GUILD_FORUM)

## Steps

### 1. Create the forum thread

```bash
BOT_TOKEN=$(grep DISCORD_BOT_TOKEN ~/.hermes/.env | cut -d= -f2)

curl -s -X POST \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/channels/{FORUM_CHANNEL_ID}/threads" \
  -d @/tmp/discord_msg.json
```

The JSON payload must be written to a temp file first (avoids bash escaping issues):

```json
{
  "name": "Thread Title Here",
  "type": 11,
  "message": {
    "content": "Thread body content here."
  }
}
```

- `type: 11` = GUILD_PUBLIC_THREAD (forum post)
- `name` = the forum thread title
- `message.content` = the initial post body
- Escape single quotes in content or use a JSON file to avoid bash issues

### 2. Post follow-up messages (optional)

```bash
curl -s -X POST \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d @/tmp/discord_reply.json \
  "https://discord.com/api/v10/channels/{THREAD_ID}/messages"
```

```json
{
  "content": "Follow-up message content"
}
```

### 3. Get the thread URL

Response includes `id` (thread ID) and `guild_id`. Construct:
`https://discord.com/channels/{guild_id}/{forum_channel_id}/{thread_id}`

## Renaming Forum Threads

Discord thread titles from bots are often meaningless (raw URLs, emojis, or generic text). Rename them into concise, human-readable sentence-case summaries.

### Rules
- Sentence case — only first word capitalized (except proper nouns like "Pi", "Hermes", "CDP")
- 5–10 words — focus on the topic, not the action taken
- No trailing punctuation
- Descriptive — a human scanning threads should instantly know what this one is about

### Example renames

| Before | After |
|---|---|
| https://lucumr.pocoo.org/2026/1/31/pi/ | Pi agent — CDP browser & Hermes integration |
| Error in cron job #4 | Cron job #4 failing on startup |
| test | Testing STT pipeline integration |

### API call

Use the built-in tool:

```
mcp_discord_mcp_edit_text_channel
```

Parameters:
- `guildId` — Discord server ID
- `channelId` — the thread ID
- `name` — new sentence-case title (5–10 words)

> **Note:** `mcp_discord_mcp_edit_text_channel` now works for threads because Discord threads are channel objects. Pass the thread ID as `channelId`.

## Tips

- **Always use a JSON file** (`-d @/tmp/file.json`) for the payload — never inline JSON in bash with curl, as escaping is fragile
- **Test first** with a simple message before posting complex formatted content
- The bot must be a member of the target server with `CREATE_THREADS` permission
- Forum channel type is 15; thread type is 11