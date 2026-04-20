---
name: postproxy
description: Create, schedule, update, and manage social media posts and comments across Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, and Threads using the PostProxy API. Use when user wants to publish posts, schedule content, create drafts, upload media, manage posting queues, update existing posts, delete posts from social platforms, or manage comments on social media platforms.
version: 1.3.0
allowed-tools: Bash
---

# PostProxy API Skill

Call the [PostProxy](https://postproxy.dev/) API to manage social media posts across multiple platforms (Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, Threads).

## Setup

API key must be set in environment variable `POSTPROXY_API_KEY`.
Get your API key at: https://app.postproxy.dev/api_keys

## Base URL

```
https://api.postproxy.dev
```

## Authentication

All requests require Bearer token:
```bash
-H "Authorization: Bearer $POSTPROXY_API_KEY"
```

## Endpoints

### List Profiles
```bash
curl -X GET "https://api.postproxy.dev/api/profiles" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### List Posts
```bash
curl -X GET "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Get Post
```bash
curl -X GET "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Create Post (JSON with media URLs)
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "Post content here"
    },
    "profiles": ["twitter", "linkedin", "threads"],
    "media": ["https://example.com/image.jpg"]
  }'
```

### Create Post (File Upload)
Use multipart form data to upload local files:
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -F "post[body]=Check out this image!" \
  -F "profiles[]=instagram" \
  -F "profiles[]=twitter" \
  -F "media[]=@/path/to/image.jpg" \
  -F "media[]=@/path/to/image2.png"
```

### Create Draft
Add `post[draft]=true` to create without publishing:
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -F "post[body]=Draft post content" \
  -F "profiles[]=twitter" \
  -F "media[]=@/path/to/image.jpg" \
  -F "post[draft]=true"
```

### Publish Draft
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{id}/publish" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Profile options: `facebook`, `instagram`, `tiktok`, `linkedin`, `youtube`, `twitter`, `threads` (or use profile IDs)

### Schedule Post
Add `scheduled_at` to post object:
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "Scheduled post",
      "scheduled_at": "2024-01-16T09:00:00Z"
    },
    "profiles": ["twitter"]
  }'
```

### Create Thread (Tweet Chain / Thread Post)
Threads allow you to create a sequence of posts published as replies. Only supported on X (Twitter) and Threads platforms.
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "1/ Here is a thread about our product launch"
    },
    "profiles": ["twitter", "threads"],
    "thread": [
      { "body": "2/ First, we built the foundation..." },
      { "body": "3/ Then we added the key features..." },
      { "body": "4/ And finally, we launched! Check it out at example.com" }
    ]
  }'
```

With media in thread posts:
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "1/ Here is a thread with screenshots"
    },
    "profiles": ["twitter"],
    "thread": [
      {
        "body": "2/ First feature",
        "media": ["https://example.com/screenshot1.jpg"]
      },
      {
        "body": "3/ Second feature",
        "media": ["https://example.com/screenshot2.jpg"]
      }
    ]
  }'
```

### Get Post Stats
Retrieves stats snapshots for one or more posts. Returns all matching snapshots so you can see trends over time.
```bash
curl -X GET "https://api.postproxy.dev/api/posts/stats?post_ids=abc123,def456&profiles=instagram,twitter&from=2026-02-01T00:00:00Z&to=2026-02-24T00:00:00Z" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Query parameters:
- `post_ids` (required): Comma-separated list of post hashids (max 50)
- `profiles` (optional): Comma-separated list of profile hashids or network names (e.g. `instagram,twitter`)
- `from` (optional): ISO 8601 timestamp — only include snapshots at or after this time
- `to` (optional): ISO 8601 timestamp — only include snapshots at or before this time

Response is keyed by post hashid, each containing a `platforms` array with `profile_id`, `platform`, and `records` (snapshots ordered by `recorded_at` ascending). Each record has a `stats` object (platform-specific metrics) and `recorded_at` timestamp.

Stats fields by platform:
- Instagram: `impressions`, `likes`, `comments`, `saved`, `profile_visits`, `follows`
- Facebook: `impressions`, `clicks`, `likes`
- Threads: `impressions`, `likes`, `replies`, `reposts`, `quotes`, `shares`
- Twitter: `impressions`, `likes`, `retweets`, `comments`, `quotes`, `saved`
- YouTube: `impressions`, `likes`, `comments`, `saved`
- LinkedIn: `impressions`
- TikTok: `impressions`, `likes`, `comments`, `shares`
- Pinterest: `impressions`, `likes`, `comments`, `saved`, `outbound_clicks`

Note: Instagram stories do not return stats. TikTok stats require a public ID.

### List Placements
Retrieves available placements for a profile. For Facebook: business pages. For LinkedIn: personal profile and organizations. For Pinterest: boards. Available for `facebook`, `linkedin`, and `pinterest` profiles.

If no placement is specified when creating a post: LinkedIn defaults to personal profile, Facebook defaults to a random connected page, Pinterest fails.
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/placements" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Response is a `data` array of placement objects with `id` (string or null for personal profile) and `name`.

### Update Post
Updates an existing post. Only drafts and scheduled posts more than 5 minutes before publish time can be updated. All body fields are optional — only send what you want to change.
```bash
curl -X PATCH "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "Updated post content!"
    }
  }'
```

Body parameters (all optional):
- `post[body]`: Updated text content
- `post[scheduled_at]`: Updated ISO 8601 schedule timestamp
- `post[draft]`: Set/unset draft status
- `profiles`: **Full replace** — array of profile IDs or network names
- `platforms`: **Merged** with existing platform params (per network)
- `media`: **Full replace** — array of URLs (send `[]` to remove all)
- `thread`: **Full replace** — array of thread children (send `[]` to remove all)
- `queue_id`: Assign post to a queue
- `queue_priority`: `high`, `medium`, or `low`

Update behavior:
- `post` fields are merged with existing values
- `profiles`, `media`, `thread` are full-replace — omitted = unchanged, `[]` = clear
- `platforms` is merged into existing params per network. Sending `{"platforms": {"youtube": {"privacy_status": "unlisted"}}}` updates only that field
- Thread children inherit parent's profiles, scheduling, draft status

Example: replace profiles and media:
```bash
curl -X PATCH "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profiles": ["twitter", "threads"],
    "media": ["https://example.com/new-image.jpg"]
  }'
```

Example: update platform params only:
```bash
curl -X PATCH "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platforms": {
      "youtube": { "privacy_status": "unlisted" }
    }
  }'
```

Errors:
- `422` "Post cannot be edited" — post is published or within 5min of scheduled publish
- `422` "Profile not found for {id}"
- `422` "Invalid platform params for {network}: {key}"
- `404` "Not found"

### Delete Post
By default removes only from DB. Pass `delete_on_platform=true` to also remove from social platforms first.
```bash
curl -X DELETE "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Delete from DB AND from all published platforms:
```bash
curl -X DELETE "https://api.postproxy.dev/api/posts/{id}?delete_on_platform=true" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Query parameter:
- `delete_on_platform` (optional, default `false`): If `true`, deletes from all published platforms before removing from DB

### Delete on Platform
Async deletes a published post from social platforms WITHOUT removing it from the DB. Optionally scope to a specific platform/profile. If no scope is given, deletes from all published platforms.
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{id}/delete_on_platform" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Scope by network:
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{id}/delete_on_platform" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"network": "twitter"}'
```

Scope by profile ID:
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{id}/delete_on_platform" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"profile_id": "prof_abc123"}'
```

Scope by post profile ID (covers entire thread for that profile):
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{id}/delete_on_platform" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"post_profile_id": "pp_abc123"}'
```

Body parameters (all optional, omit all = delete from every published platform):
- `post_profile_id`: ID of a specific post profile. Resolves to the underlying profile, deletes across the entire thread
- `profile_id`: ID of a profile. Deletes all post profiles for this profile on the post
- `network`: Network name. Deletes all post profiles for this network on the post

Supported platforms: `facebook`, `threads`, `twitter`, `linkedin`, `pinterest`, `youtube`.
NOT supported: `instagram`, `tiktok` — request returns `422` if scoped to one of these.

Response (200):
```json
{
  "success": true,
  "deleting": [
    { "post_profile_id": "pp_abc123", "platform": "twitter" }
  ]
}
```

After triggering, platform status transitions: `published` → `pending_deletion` → `deleted`.

### List Queues
```bash
curl -X GET "https://api.postproxy.dev/api/post_queues" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```
Optional query parameter: `profile_group_id` to filter by profile group.

### Get Queue
```bash
curl -X GET "https://api.postproxy.dev/api/post_queues/{queue_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Get Next Available Slot
```bash
curl -X GET "https://api.postproxy.dev/api/post_queues/{queue_id}/next_slot" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Create Queue
```bash
curl -X POST "https://api.postproxy.dev/api/post_queues" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profile_group_id": "pg123",
    "post_queue": {
      "name": "Morning Posts",
      "description": "Weekday morning content",
      "timezone": "America/New_York",
      "jitter": 10,
      "queue_timeslots_attributes": [
        { "day": 1, "time": "09:00" },
        { "day": 2, "time": "09:00" },
        { "day": 3, "time": "09:00" },
        { "day": 4, "time": "09:00" },
        { "day": 5, "time": "09:00" }
      ]
    }
  }'
```

Parameters:
- `profile_group_id` (required): Profile group ID to connect the queue to
- `post_queue[name]` (required): Queue name
- `post_queue[description]` (optional): Description
- `post_queue[timezone]` (optional): IANA timezone name (default: `UTC`)
- `post_queue[jitter]` (optional): Random offset in minutes 0–60 applied to scheduled times for natural posting patterns (default: `0`)
- `post_queue[queue_timeslots_attributes]` (optional): Array of timeslots, each with `day` (0=Sunday–6=Saturday) and `time` (24-hour `HH:MM`)

### Update Queue
```bash
curl -X PATCH "https://api.postproxy.dev/api/post_queues/{queue_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post_queue": {
      "name": "New Name",
      "timezone": "America/Los_Angeles",
      "enabled": false,
      "jitter": 5,
      "queue_timeslots_attributes": [
        { "day": 2, "time": "10:00" },
        { "id": 42, "_destroy": true }
      ]
    }
  }'
```

Parameters (all optional):
- `post_queue[name]`: New queue name
- `post_queue[description]`: New description
- `post_queue[timezone]`: IANA timezone
- `post_queue[enabled]`: `false` to pause, `true` to unpause
- `post_queue[jitter]`: Random offset in minutes (0–60)
- `post_queue[queue_timeslots_attributes]`: Add timeslots with `{ "day": N, "time": "HH:MM" }`, remove with `{ "id": N, "_destroy": true }`

### Delete Queue
```bash
curl -X DELETE "https://api.postproxy.dev/api/post_queues/{queue_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Add Post to Queue
Instead of `scheduled_at`, pass `queue_id` and optionally `queue_priority` when creating a post:
```bash
curl -X POST "https://api.postproxy.dev/api/posts" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "body": "Queued post content"
    },
    "profiles": ["twitter", "linkedin"],
    "queue_id": "q1abc",
    "queue_priority": "high"
  }'
```

Queue parameters on post creation:
- `queue_id` (required): Queue ID
- `queue_priority` (optional): `high`, `medium` (default), or `low`. Higher priority posts get earlier timeslots.

Do not pass `scheduled_at` together with `queue_id` — the queue determines the scheduled time automatically.

## Queue Concepts

- **Timeslots**: Weekly recurring schedules (day + time in queue's timezone)
- **Priority**: `high` → scheduled first, `medium` → default, `low` → fills remaining slots
- **Jitter**: Random +/- offset (0–60 min) for natural posting patterns
- **Pausing**: Set `enabled: false` to pause; posts won't publish while paused. Unpausing rearranges all posts into future slots.
- **Dynamic Rearrangement**: Queue auto-rearranges all posts when posts are added/removed, timeslots change, timezone changes, or queue is unpaused.

### List Comments
Retrieves paginated top-level comments for a published post. Each top-level comment includes a flat `replies` array. All comment endpoints require the `profile_id` query parameter.
```bash
curl -X GET "https://api.postproxy.dev/api/posts/{post_id}/comments?profile_id={profile_id}&page=0&per_page=20" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Query parameters:
- `profile_id` (required): Profile ID
- `page` (optional): Page number, zero-indexed (default: `0`)
- `per_page` (optional): Top-level comments per page (default: `20`)

Pagination applies to top-level comments only. All replies are flattened into the `replies` array of their root comment, sorted by `created_at` ascending. Each reply retains `parent_external_id` so the client can reconstruct the tree.

### Get Comment
```bash
curl -X GET "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

The `comment_id` can be a Postproxy ID (e.g. `cmt_abc123`) or the platform's native external ID.

### Create Comment
Creates a comment or reply on a published post. Processed asynchronously — returns `status: "pending"` initially.
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Thanks for the feedback everyone!"
  }'
```

To reply to an existing comment, add `parent_id`:
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Glad you liked it!",
    "parent_id": "cmt_abc123"
  }'
```

Parameters:
- `text` (required): Comment text content
- `parent_id` (optional): Postproxy ID or external ID of comment to reply to. Omit to comment on the post itself.

### Delete Comment
```bash
curl -X DELETE "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Hide Comment
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}/hide?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Unhide Comment
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}/unhide?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Like Comment
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}/like?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

### Unlike Comment
```bash
curl -X POST "https://api.postproxy.dev/api/posts/{post_id}/comments/{comment_id}/unlike?profile_id={profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

## Comments

### Platform Support for Comment Actions

| Action | Instagram | Facebook | Threads | YouTube | LinkedIn |
|--------|-----------|----------|---------|---------|----------|
| List | Yes | Yes | Yes | Yes | Yes |
| Reply | Yes | Yes | Yes | Yes | Yes |
| Delete | Yes | Yes | No | Yes | Yes |
| Hide/Unhide | Yes | Yes | Yes | No | No |
| Like/Unlike | No | Yes | No | No | No |

Attempting an unsupported action returns `405 Method Not Allowed`.

### Comment Statuses
- `synced` — fetched from the platform during sync
- `pending` — created via API, being published to the platform
- `published` — successfully published to the platform
- `failed` — failed to publish to the platform

### Async Behavior
All write operations (create, delete, hide, unhide, like, unlike) are processed asynchronously. Create returns the comment with `status: "pending"` and `external_id: null`. Once published, status updates to `"published"` and `external_id` is populated.

## Platform-Specific Parameters

For Instagram, TikTok, YouTube, add `platforms` object:
```json
{
  "platforms": {
    "instagram": { "format": "reel", "first_comment": "Link in bio!" },
    "youtube": { "title": "Video Title", "privacy_status": "public" },
    "tiktok": { "privacy_status": "PUBLIC_TO_EVERYONE" }
  }
}
```

## Threads

Threads allow you to create a sequence of posts that are published as replies to each other, forming a conversation thread.

### Supported Platforms
- **X (Twitter)** — each post is published as a reply to the previous tweet
- **Threads** — each post is published as a reply to the previous Threads post

Attempting to create a thread with other platforms (Instagram, Facebook, LinkedIn, etc.) will return a `422` error.

### How Threads Work
1. The parent post (`post[body]`) is published first on each platform
2. Each child post in the `thread` array is published sequentially as a reply to the previous post
3. Per-platform chains are independent — the X chain and Threads chain run in parallel
4. Position is determined by the array order (first item = first reply, etc.)

### Thread Parameter Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `body` | string | Yes | Text content for this thread post |
| `media` | array | No | Array of media URLs (same format as top-level `media`) |

### Error Handling
- If a post in the thread chain fails, subsequent posts in that chain will **wait** (they are not published)
- Each platform chain is independent — a failure on X does not block the Threads chain

## User Request

$ARGUMENTS
