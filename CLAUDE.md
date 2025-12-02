# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zenn content repository for publishing technical articles and books on zenn.dev (Japanese technical publishing platform). Articles are synced with Zenn via GitHub integration - pushing to `main` branch publishes content.

## Commands

- `npx zenn preview` - Start local preview server at http://localhost:8000
- `npx zenn new:article` - Create new article with auto-generated slug
- `npx zenn new:book` - Create new book
- `npx zenn list:articles` - List all articles
- `npx zenn list:books` - List all books

## Project Structure

- `articles/` - Markdown articles with YAML frontmatter
- `books/` - Zenn books
- `images/` - Images referenced in articles (use relative paths)

## Article Format

```yaml
---
title: "Article Title"
emoji: "🔧"
type: "tech" # or "idea"
topics: ["topic1", "topic2"] # up to 5 topics
published: true # or false for drafts
publication_name: "gmomedia" # for organization publication
---
```

## Article Naming Convention

- Article file names should include the title/topic (e.g., `why-i-write-blogs.md`) instead of using auto-generated slugs (e.g., `368b4be339f110.md`)

## Writing Guidelines

- Write all articles in Japanese

## Article Review

When asked to review an article, check the following:

- **Structure**: Is the flow logical? (Introduction → Main points → Conclusion)
- **Concrete examples**: Are there specific examples to support abstract claims?
- **Logical connections**: Do sections flow naturally into each other?
- **Typos**: Check for spelling and grammatical errors

## Publishing Workflow

1. Create article: `npx zenn new:article`
2. Edit the generated markdown file in `articles/`
3. Preview locally: `npx zenn preview`
4. Set `published: true` in frontmatter
5. Commit and push to `main` branch to publish