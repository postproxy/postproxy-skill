# PostProxy Skill

An Agent Skill for managing social media posts across multiple platforms using the [PostProxy API](https://postproxy.dev/).

## Overview

This skill enables AI agents to create, manage, and schedule social media posts across Facebook, Instagram, TikTok, LinkedIn, YouTube, X/Twitter, and Threads through the PostProxy API.

## Installation

Install this skill using the `skills` CLI:

```bash
npx skills add postproxy/postproxy-skill
```

Or install to specific agents:

```bash
npx skills add postproxy/postproxy-skill -a cursor -a claude-code
```

## Setup

Before using this skill, you need to:

1. Get your PostProxy API key from [https://app.postproxy.dev/api_keys](https://app.postproxy.dev/api_keys)
2. Set it as an environment variable:

```bash
export POSTPROXY_API_KEY=your_api_key_here
```

## Usage Examples

Once installed, your AI agent can use this skill to:

### Create a post across multiple platforms

```
Create a post saying "Check out our new feature!" and publish it to Twitter, LinkedIn, and Threads
```

### Create a thread (tweet chain)

```
Create a thread on Twitter about our product launch with 4 posts explaining the features
```

### Schedule a post

```
Schedule a post for tomorrow at 9 AM saying "Good morning! Here's today's update" to Twitter
```

### Create a draft with media

```
Create a draft post with the image from ./screenshot.png and the text "New product launch" for Instagram
```

### Get post stats

```
Show me the stats for my last 3 posts
```

```
How did my Instagram posts perform this week?
```

### List placements

```
What Facebook pages can I post to?
```

```
Show me my LinkedIn placements
```

### Queue management

```
Show me all my posting queues
```

```
Create a queue called "Weekday Mornings" for profile group pg123, timezone America/New_York, with timeslots Monday through Friday at 9am
```

```
Add a post to queue q1abc with high priority: "Check out our latest feature!" to Twitter and LinkedIn
```

```
Pause queue "morning updates"
```

```
What's the next available slot for queue "morning updates"?
```

### List and manage posts

```
Show me all my scheduled posts
```

```
Delete the post with ID 12345
```

## Supported Platforms

- Facebook
- Instagram
- TikTok
- LinkedIn
- YouTube
- X/Twitter
- Threads
- Pinterest

## Features

- ✅ Create posts with text and media
- ✅ Create thread posts (tweet chains) on Twitter and Threads
- ✅ Schedule posts for future publication
- ✅ Create drafts for review before publishing
- ✅ Upload local files as media attachments
- ✅ Publish drafts when ready
- ✅ Delete posts
- ✅ List profiles and posts
- ✅ Get post stats and performance metrics over time
- ✅ List placements (Facebook pages, LinkedIn orgs, Pinterest boards)
- ✅ Platform-specific parameters (Instagram Reels, YouTube titles, etc.)
- ✅ Queue management (create, update, pause, delete queues)
- ✅ Add posts to queues with priority-based scheduling
- ✅ Weekly timeslot configuration with timezone and jitter support

## API Documentation

For detailed API documentation, visit [https://postproxy.dev/](https://postproxy.dev/)

## Requirements

- PostProxy API key (get one at [https://app.postproxy.dev/api_keys](https://app.postproxy.dev/api_keys))
- Environment variable `POSTPROXY_API_KEY` must be set

## License

See the repository for license information.

## Related Links

- [PostProxy Website](https://postproxy.dev/)
- [PostProxy API Documentation](https://postproxy.dev/)
- [Skills.sh Directory](https://skills.sh/)
