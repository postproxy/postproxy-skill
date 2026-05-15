---
name: postproxy
description: Create, schedule, update, and manage social media posts and comments across Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, Threads, Pinterest, Bluesky, Telegram, and Google Business using the PostProxy API. Use when user wants to publish posts, schedule content, create drafts, upload media, manage posting queues, update existing posts, delete posts from social platforms, manage post comments, reply to Google Business reviews, or retrieve profile/follower stats.
version: 1.5.0
allowed-tools: Bash
---

# PostProxy API Skill

Call the [PostProxy](https://postproxy.dev/) API to manage social media posts across multiple platforms (Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, Threads, Pinterest, Bluesky, Telegram, Google Business).

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
Optional query parameter: `profile_group_id` to filter by profile group.

### Get Profile (with latest stats)
Returns the profile fields plus the latest stats snapshot per placement, and a `summary_stats` rollup for placement networks.
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Response shape:
- `latest_stats` — array of latest snapshots. One entry per placement for placement networks (Facebook/LinkedIn/Telegram); a single entry with `placement_id: null` for non-placement networks (Bluesky, Twitter, Instagram, Threads, YouTube, TikTok, Pinterest). Empty if no snapshots have been recorded yet.
- `latest_stats[].placement_id` — string or `null`.
- `latest_stats[].stats` — platform-specific metrics (see [Profile Stats Fields](#profile-stats-fields)).
- `latest_stats[].recorded_at` — ISO 8601 timestamp.
- `summary_stats` — for placement networks, numeric values summed across the latest snapshot of every placement (non-numeric values like `channel_title` are omitted). `null` for non-placement networks and when no snapshots exist.

Snapshots refresh roughly every 23 hours per profile. If `latest_stats` is empty, the profile is connected but hasn't been polled yet.

### Get Profile Stats (timeseries)
Retrieves the full stats timeseries for a profile. Use this to plot follower growth and engagement trends over time. Snapshots are captured roughly every 23 hours.
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/stats?placement_id={placement_id}&from=2026-04-01T00:00:00Z&to=2026-05-01T00:00:00Z" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Query parameters:
- `placement_id` (conditional): **Required** for `facebook`, `linkedin`, and `telegram` profiles. The platform-specific ID returned by [List Placements](#list-placements). Omit (or ignored) for other networks.
- `from` (optional): ISO 8601 — only include snapshots at or after this time.
- `to` (optional): ISO 8601 — only include snapshots at or before this time.

Response shape: `data.profile_id`, `data.platform`, `data.placement_id` (echo of the request, `null` for non-placement networks), and `data.records` — snapshots ordered by `recorded_at` ascending. Each record has `stats` (platform-specific metrics) and `recorded_at`.

Missing `placement_id` for a placement network returns `400`.

For a non-placement network like Bluesky, omit `placement_id`:
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/stats" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

#### Profile Stats Fields

The `stats` object's keys come straight from each platform's API — they are not normalized. A key only appears in a snapshot if the platform returned a value on that polling cycle, so fields can come and go between records.

| Network | Placement-scoped? | Typical fields |
|---------|-------------------|----------------|
| `facebook` | Yes (per page) | `fan_count`, `followers_count`, plus daily page insights (`page_impressions`, `page_views_total`, `page_fan_adds`, `page_fan_removes`, …) |
| `linkedin` | Yes (per organization) | `followerCount`, `shareCount`, `likeCount`, `commentCount`, `clickCount`, `engagement`, `allPageViews`, `overviewPageViews`, `aboutPageViews`, `careersPageViews`, `peoplePageViews`, `insightsPageViews` |
| `telegram` | Yes (per channel) | `followers_count`, `channel_title`, `channel_username` |
| `instagram` | No | `followers_count`, `follows_count`, `media_count`, `reach`, `profile_views`, `accounts_engaged`, `total_interactions`, `website_clicks`, `follower_count` |
| `threads` | No | `followers_count`, `views`, `likes`, `replies`, `reposts`, `quotes` |
| `youtube` | No | `subscriberCount`, `viewCount`, `videoCount` |
| `twitter` | No | `followers_count`, `following_count`, `tweet_count`, `listed_count`, `like_count` |
| `tiktok` | No | `follower_count`, `following_count`, `likes_count`, `video_count` |
| `pinterest` | No | `follower_count`, `following_count`, `pin_count`, `board_count`, `monthly_views`, `analytics_30d` |
| `bluesky` | No | `followersCount`, `followsCount`, `postsCount` |

Notes:
- Non-numeric fields (e.g. Telegram's `channel_title`) appear in `latest_stats[].stats` but are omitted from `summary_stats.stats`, which sums numeric values only.
- LinkedIn page-view metrics are filtered to rollups only (redundant mobile/desktop splits and dead sections like `productsPageViews` / `lifeAtPageViews` are dropped).

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

Profile options: `facebook`, `instagram`, `tiktok`, `linkedin`, `youtube`, `twitter`, `threads`, `pinterest`, `bluesky`, `telegram`, `google_business` (or use profile IDs)

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
Threads allow you to create a sequence of posts published as replies. Supported on X (Twitter), Threads, and Bluesky.
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
- Bluesky: `likes`, `reposts`, `comments`, `quotes`

Note: Instagram stories do not return stats. TikTok stats require a public ID. Bluesky does not expose impression/view counts. Telegram channel posts do not currently expose per-post stats.

### List Placements
Retrieves available placements for a profile. For Facebook: business pages. For LinkedIn: personal profile and organizations. For Pinterest: boards. For Telegram: channels the bot has been added to as administrator. For Google Business: locations (full resource path `accounts/X/locations/Y`). Available for `facebook`, `linkedin`, `pinterest`, `telegram`, and `google_business` profiles.

If no placement is specified when creating a post: LinkedIn defaults to personal profile, Facebook defaults to a random connected page, Pinterest fails, Telegram fails (`chat_id` is always required), Google Business fails (`location_id` is always required).
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/placements" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Response is a `data` array of placement objects with `id` (string, or `null` for the LinkedIn personal profile) and `name`. For Telegram, the placement `id` is the `chat_id` to pass in `platforms.telegram.chat_id` when creating a post. The Telegram list is empty until the user adds the bot as administrator to a channel — poll after connecting the bot.

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

Supported platforms: `facebook`, `threads`, `twitter`, `linkedin`, `pinterest`, `youtube`, `bluesky`, `telegram`, `google_business`.
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

### List Profile Comments
Retrieves reviews scoped to a profile (and optionally a placement/location). Currently only `google_business` profiles return data — Google reviews live on a **location**, not a post, so they're exposed here rather than under the post Comments API. Each top-level comment includes a flat `replies` array.
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/comments?placement_id={location_path}&page=0&per_page=20" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

Query parameters:
- `placement_id` (optional): Filter to reviews on a single location (the `accounts/X/locations/Y` path returned by [List Placements](#list-placements))
- `page` (optional): Page number, zero-indexed (default: `0`)
- `per_page` (optional): Top-level comments per page (default: `20`)

### Get Profile Comment
```bash
curl -X GET "https://api.postproxy.dev/api/profiles/{profile_id}/comments/{comment_id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

The `comment_id` can be a Postproxy hashid (e.g. `cmt_abc123`) or the platform's native external ID (e.g. `accounts/.../locations/.../reviews/...`).

### Reply to Profile Comment
Creates a reply to an existing review. **Top-level comments are not allowed** — Google Business reviews come from end users, so `parent_id` is required (returns `422` if missing).
```bash
curl -X POST "https://api.postproxy.dev/api/profiles/{profile_id}/comments" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "parent_id": "accounts/1234/locations/5678/reviews/AbFvOq",
    "text": "Thanks for the kind words!"
  }'
```

Parameters:
- `parent_id` (required): Hashid or external_id of the review being replied to
- `text` (required): Reply body

Processed asynchronously — returns the new comment in status `pending`, then transitions to `published` (or `failed` / `failed_waiting_for_retry` with `error_message`).

### Delete Profile Comment
Deletes **your reply only** — the Google Business API does not let businesses delete the underlying review, so the original review row stays. Returns `{ "accepted": true }` while the delete is dispatched.
```bash
curl -X DELETE "https://api.postproxy.dev/api/profiles/{profile_id}/comments/{comment_id}" \
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

## Profile Comments

The Profile Comments API (`/api/profiles/:profile_id/comments`) is **separate from the post Comments API**. It surfaces feedback scoped to a profile/location rather than to a single post. Today it exposes **Google Business reviews** — reviews live on a location, not a post, so they cannot be expressed through the post-level Comments API.

### Platform Support

| Action | Google Business |
|--------|-----------------|
| List | Yes (reviews on the location) |
| Reply | Yes (reply to an existing review only — top-level authoring not allowed) |
| Delete | Yes (removes your reply; review remains) |

Other networks return `405 Method Not Allowed`.

### Statuses
- `synced` — fetched from Google during sync (incoming reviews)
- `pending` — reply created via API, being published to Google
- `published` — reply successfully posted to Google
- `failed` / `failed_waiting_for_retry` — reply publish failed; `error_message` populated

### Comment ID Resolution
Both `:comment_id` and `parent_id` accept either:
- Postproxy hashid (e.g. `abc123xyz`)
- External ID — the platform's native resource path (e.g. `accounts/.../locations/.../reviews/...`)

### Webhook Event
Subscribe to `profile_comment.created` (or `*`) on a webhook endpoint to receive events for both newly-synced incoming reviews and outgoing replies once published. Payload contains the same fields as a single comment fetch (`id`, `profile_id`, `platform`, `placement_id`, `external_id`, `parent_external_id`, `body`, `status`, `author_username`, `author_avatar_url`, `platform_data`, `posted_at`, `created_at`).

### Errors
- `404` — profile/comment/parent not found, or profile group mismatch
- `405` — action not supported for this profile's network (e.g. reply on a non-`google_business` profile)
- `422` — missing `parent_id` (top-level authoring forbidden), or replying before the parent has been published

## Platform-Specific Parameters

For Instagram, TikTok, YouTube, Telegram, etc., add a `platforms` object keyed by network:
```json
{
  "platforms": {
    "instagram": { "format": "reel", "first_comment": "Link in bio!" },
    "youtube": { "title": "Video Title", "privacy_status": "public" },
    "tiktok": { "privacy_status": "PUBLIC_TO_EVERYONE" },
    "telegram": { "chat_id": "-1001234567890", "parse_mode": "HTML", "disable_link_preview": true }
  }
}
```

### Bluesky
- No custom parameters.
- **Character limit:** 300 graphemes (emoji + combining sequences count as one).
- **Rich text auto-detection** — Postproxy scans the body and converts mentions, hashtags, and URLs to AT Protocol facets. Write the post normally, no special markup:
  - `@handle.bsky.social` → clickable mention (custom-domain handles work too; resolution cached 24h; unresolvable handles fall through as plain text).
  - `#hashtag` → clickable tag feed link (letters/digits/underscores, up to 64 chars).
  - `https://...` URLs → clickable links (trailing punctuation stripped).
- **Link card previews** — when the post contains a URL **and no media attachments**, Postproxy auto-generates a Bluesky link card embed by reading OG meta (`og:title`, `og:description`, `og:image`). Image is skipped if > ~950 KB. Link cards and media attachments are mutually exclusive — if media is attached, the URL is still rendered as a clickable link but no preview card is generated. Place the URL at the end of the post for the best preview.
- **Media:** images ≤ 1 MB (jpg/png/webp/gif, max 4), video ≤ 100 MB (mp4/mov, max 1, 1–60s). Cannot mix video + image.

Example:
```json
{
  "post": {
    "body": "Hey @jay.bsky.team — new #release out: https://example.com/post"
  },
  "profiles": ["bluesky"]
}
```

### Telegram
Telegram is a **bring-your-own-bot** integration. Each connected profile is one bot (created via [@BotFather](https://t.me/BotFather)) and can publish to any channel where that bot has been added as administrator. List target channels via the [List Placements](#list-placements) endpoint.

Parameters (under `platforms.telegram`):
- `chat_id` (string, **required**) — ID of the destination channel/group. Get it from `/api/profiles/{profile_id}/placements`.
- `parse_mode` (string, optional) — `"HTML"`, `"MarkdownV2"`, or omit for plain text.
- `disable_link_preview` (boolean, optional) — suppress the URL preview card.
- `disable_notification` (boolean, optional) — send silently (no notification sound).

**Character limit:** 4,096 chars for text-only posts, 1,024 chars for captions when media is attached. Body content beyond the caption limit is truncated.

**Media:** images ≤ 10 MB (jpg/png/webp/gif, max 10), video ≤ 50 MB (mp4/mov, max 10), document ≤ 50 MB (pdf/doc/docx/zip/mp3/wav, max 1). Mixing video and image is allowed (sent as a media group).

The returned `external_id` is `"<chat_id>/<message_id>"`. For media groups, ids are comma-separated: `"<chat_id>/<id1>,<id2>,…"`.

Notes:
- Bot must be a member (preferably **administrator** with permission to post) of the target channel. Add it manually in Telegram.
- Channels appear in `/placements` after Telegram pushes a `my_chat_member` event. If a channel doesn't show up, ask the user to remove and re-add the bot to refire discovery.
- If the bot is kicked or its token is revoked in BotFather, the next publish fails with `inactive_profile_error` and the profile is deactivated.

Example:
```json
{
  "post": {
    "body": "<b>New release</b> — read more on our blog https://example.com/post"
  },
  "profiles": ["telegram"],
  "platforms": {
    "telegram": {
      "chat_id": "-1001234567890",
      "parse_mode": "HTML",
      "disable_link_preview": true,
      "disable_notification": false
    }
  }
}
```

### Google Business
Google Business publishes **local posts** to a Business Profile **location**. One profile may manage multiple accounts × multiple locations — the target location is selected via `location_id`. Use [List Placements](#list-placements) to enumerate locations.

#### Formats

| Format | Description |
|--------|-------------|
| `standard` | Plain local post (default) |
| `event` | Event with a title and date range |
| `offer` | Promotion with a validity window and optional coupon |

Each format accepts only its relevant parameters — fields are scoped per format.

#### Shared Parameters (all formats)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `location_id` | string | Yes | Full location resource path, e.g. `accounts/123456789/locations/987654321`. Get it from [List Placements](#list-placements). |
| `language_code` | string | No | BCP 47 code (e.g. `en`, `de`). Defaults to `en`. Metadata only — does not translate the body. |
| `cta_action_type` | string | No | Call-to-action button. One of `LEARN_MORE`, `BOOK`, `ORDER`, `SHOP`, `SIGN_UP`, `CALL`. |
| `cta_url` | string | No | HTTPS URL the CTA button opens. **Required for every CTA except `CALL`**. |

#### `event` Format — Additional Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_title` | string | Yes | Display title for the event. |
| `event_start_date` | string | Yes | `YYYY-MM-DD`. |
| `event_end_date` | string | Yes | `YYYY-MM-DD`. |
| `event_start_time` | string | No | `HH:MM` (24h). |
| `event_end_time` | string | No | `HH:MM` (24h). |

#### `offer` Format — Additional Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_start_date` | string | Yes | `YYYY-MM-DD`. Start of the offer validity window. |
| `event_end_date` | string | Yes | `YYYY-MM-DD`. End of the offer validity window. |
| `event_start_time` | string | No | `HH:MM` (24h). |
| `event_end_time` | string | No | `HH:MM` (24h). |
| `event_title` | string | No | Offer headline. Defaults to `"Special Offer"` if blank. |
| `offer_coupon_code` | string | No | Promo code displayed with the offer. |
| `offer_redeem_url` | string | No | URL where the offer can be redeemed. |
| `offer_terms` | string | No | Terms and conditions for the offer. |

**Character limit:** 1,500 characters (post summary).

**Media:** image only — jpg/png ≤ 5 MB, max 1, minimum 400×300 px (recommended 1200×900, 4:3). **Video is not supported.** Text-only posts are allowed.

#### Notes

- The `external_id` returned for a local post is the full Google resource path (e.g. `accounts/.../locations/.../localPosts/...`). Pass it back as-is for deletion.
- "Comments" on a Google Business profile map to **reviews on the location**, not per-post comments. They are exposed via the [Profile Comments API](#list-profile-comments), not the post Comments API.
- Google has sunset local posts for some verticals (e.g. lodging). Publishing on a location whose category is no longer eligible returns a permission/validation error.

#### Example — `standard`

```json
{
  "post": {
    "body": "We're now open on Sundays from 10am to 4pm — come visit!"
  },
  "profiles": ["google_business"],
  "platforms": {
    "google_business": {
      "format": "standard",
      "location_id": "accounts/123456789/locations/987654321",
      "cta_action_type": "LEARN_MORE",
      "cta_url": "https://acme.example.com/hours"
    }
  }
}
```

#### Example — `event`

```json
{
  "post": {
    "body": "Join us for our 5-year anniversary party — live music, free coffee, prizes."
  },
  "profiles": ["google_business"],
  "platforms": {
    "google_business": {
      "format": "event",
      "location_id": "accounts/123456789/locations/987654321",
      "event_title": "Acme Coffee 5-Year Anniversary",
      "event_start_date": "2026-06-15",
      "event_start_time": "18:00",
      "event_end_date": "2026-06-15",
      "event_end_time": "22:00",
      "cta_action_type": "LEARN_MORE",
      "cta_url": "https://acme.example.com/anniversary"
    }
  }
}
```

#### Example — `offer`

`offer` posts require a validity window (`event_start_date` and `event_end_date`). `event_title` is optional and defaults to `"Special Offer"` when blank.

```json
{
  "post": {
    "body": "20% off all whole-bean coffee through the end of the month."
  },
  "profiles": ["google_business"],
  "platforms": {
    "google_business": {
      "format": "offer",
      "location_id": "accounts/123456789/locations/987654321",
      "event_start_date": "2026-06-01",
      "event_end_date": "2026-06-30",
      "offer_coupon_code": "BEANS20",
      "offer_redeem_url": "https://acme.example.com/shop",
      "offer_terms": "One per customer. Cannot be combined with other offers."
    }
  }
}
```

## Threads

Threads allow you to create a sequence of posts that are published as replies to each other, forming a conversation thread.

### Supported Platforms
- **X (Twitter)** — each post is published as a reply to the previous tweet
- **Threads** — each post is published as a reply to the previous Threads post
- **Bluesky** — each post is published as a reply to the previous Bluesky post

Attempting to create a thread with other platforms (Instagram, Facebook, LinkedIn, etc.) will return a `422` error.

### How Threads Work
1. The parent post (`post[body]`) is published first on each platform
2. Each child post in the `thread` array is published sequentially as a reply to the previous post
3. Per-platform chains are independent — the X, Threads, and Bluesky chains run in parallel
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
