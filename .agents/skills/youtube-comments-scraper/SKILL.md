---
name: youtube-comments-scraper
description: Expertise in scraping YouTube comments and performing sentiment analysis using the official YouTube Data API v3 and Bun. Use when asked to collect, analyze, or fetch YouTube comments.
---

# YouTube Comments Scraper Skill

This skill provides expertise and exact instructions for building and running a local YouTube comment scraper and sentiment analyzer using Bun and the official YouTube Data API v3.

## When to use this skill

- When the user asks to scrape or fetch comments from a YouTube video.
- When the user wants to analyze sentiment of YouTube comments.
- When working inside the "YouTube Comments Scraper" project.

## Technical Constraints & Environment

1. **Local WSL & Bun Only**: All development, scripts, and executions must use Bun (`bun init`, `bun install`, `bun run`). No Node.js, npm, or yarn. No cloud deployment. No Docker.
2. **Official API Only**: Do not use browser automation (Puppeteer/Selenium) or HTML DOM scraping. Only use the YouTube Data API v3.
3. **Security**: Ensure the `YOUTUBE_API_KEY` is always loaded from a `.env` file. Never hardcode the API key in the scripts or output logs.
4. **Sentiment Analysis**: Use a local lexicon-based JavaScript library (compatible with Bun) for sentiment analysis. Do not call external cloud APIs for sentiment.

## Core API Endpoints

You will rely primarily on two endpoints from `https://www.googleapis.com/youtube/v3/`:

1. **`commentThreads.list`**: Fetches top-level comments for a specific `videoId`.
   - **Parameters required**: `part=snippet`, `videoId`, `key`, `textFormat=plainText`, `maxResults=100`.
   - **Pagination**: Use `pageToken` to fetch next pages until limits are reached.
2. **`comments.list`**: Fetches replies to a specific top-level comment.
   - **Parameters required**: `part=snippet`, `parentId`, `key`, `textFormat=plainText`, `maxResults=100`.

## Implementation Steps

When tasked with building or running the scraper:

1. **Check Environment**: Ensure `.env` exists with `YOUTUBE_API_KEY`.
2. **Initialize**: Use `bun init` if `package.json` does not exist.
3. **Install Dependencies**: `bun add` required local sentiment libraries and CSV exporters if needed. (Bun has built-in fetch, so no `axios` or `requests` is needed).
4. **Fetch Flow**: 
   - Start with `commentThreads.list`.
   - Collect top-level comments.
   - Loop through `nextPageToken` if pagination is required.
   - Fetch replies using `comments.list` with `parentId` from top-level comments.
5. **Process Data**: Pass the `textDisplay` or `textOriginal` of each comment into the local sentiment analyzer to yield a numeric `score` and categorical `label` (e.g., POSITIVE, NEGATIVE, NEUTRAL).
6. **Export**: Map the data to objects containing `comment_id`, `author`, `text`, `like_count`, `published_at`, `sentiment_score`, `sentiment_label` and save it to a local CSV or JSON file using Bun's file system (`Bun.write`).

## Error Handling

- **Disabled Comments**: If the API returns a 403 error indicating comments are disabled, catch it gracefully and inform the user.
- **Quota Exceeded**: Catch 403 quota errors and exit cleanly. Do not loop infinitely.
- **Rate Limiting**: Implement exponential backoff if handling massive volumes of comments.
