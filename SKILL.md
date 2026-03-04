---
name: postproxy
description: Create, schedule, and manage social media posts across Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, and Threads using the PostProxy API. Use when user wants to publish posts, schedule content, create drafts, upload media, or manage existing posts on social media platforms.
version: 1.0.0
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

### Delete Post
```bash
curl -X DELETE "https://api.postproxy.dev/api/posts/{id}" \
  -H "Authorization: Bearer $POSTPROXY_API_KEY"
```

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
