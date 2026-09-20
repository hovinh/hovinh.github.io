# Blog Style Guide

Conventions for posts in `blog/_posts/`, distilled from the 2026 posts. Follow these so new posts read as one blog.

## File and front matter

- Filename: `YYYY-MM-DD-slug.md`. The slug should match the title (title "Web Dev for Data People" gives `web-dev-for-data-people`).
- Front matter:

```yaml
---
layout: post
title: "Short Title: Optional Subtitle That Says What the Reader Gets"
description: >
  Two to four sentences. What the post covers and the angle it takes.
author: author1
comments: true
---
```

- Assets live under `assets/blog/YYYY-MM-DD/` (the post's date) and are referenced as `/assets/blog/YYYY-MM-DD/name.ext`.

## Structure

1. **Intro, 1 to 3 short paragraphs, before the table of contents.** Cover, in this order:
   - Why the post exists (a personal trigger or a problem I ran into).
   - What the reader gets (the main ideas or artifacts).
   - How the post is organised, in a sentence or two.
   - Prerequisites, as one plain sentence inside the intro (no separate `**Prerequisite**` line).
2. **Table of contents**: a bullet list of anchor links to the `##` sections, placed right after the intro. No "Table of Contents" heading is required. Sub-bullets are allowed for long posts.
3. **Body**: one `##` per major idea, `###` for sub-parts. Headings are short phrases with a hook ("Two Gaps That Bite Later", "One Table, Three Questions"), not generic labels.
4. **Ending**: finish on `## Summary` or `## Closing Thought`. The last paragraph is the takeaway. Nothing follows it, no trailing separators or link lists.

## Tone

- First person, practitioner voice: "I have worked on", "the rule I keep repeating to myself".
- Concrete before abstract. Show the example, then name the idea.
- Be honest about limits: say what was and was not verified or run.
- Plain words, short paragraphs. Bold key terms on first use; use italics for light emphasis.
- Use `>` blockquotes for the one rule or definition worth remembering.

## Punctuation

- **No em-dashes.** Do not use `—` (or ` — `). Rewrite with a comma, colon, period, or parentheses instead.
- Prefer commas and periods over long chained clauses.

## Figures

Use either form, with a caption and the figure class:

```markdown
![Fig01](/assets/blog/YYYY-MM-DD/name.svg){:data-width="900" data-height="500"}
Fig. 1. What the figure shows and what to notice.
{:.figure}
```

or `<img src="/assets/blog/YYYY-MM-DD/name.png" alt="Describe the image" />` for images with no caption. Number figures in order and always give alt text.

## Math, code, tables

- Math: `$$...$$` inline or block.
- Code: fenced blocks with a language tag. Prefer real, runnable code over pseudocode.
- Tables for comparisons and lookups, with a short lead-in sentence explaining how to read them.

## References

- Cite inline: `text<sup><a href="URL">(1)</a></sup>` or a normal markdown link `[text](URL)`.
- Do not add a trailing list of bare URLs; the inline links are the references.

## Pre-publish checklist

- [ ] Slug matches the title; date matches the assets folder.
- [ ] Intro covers why, what, how, prerequisites; sits before the ToC.
- [ ] Every ToC anchor matches a heading.
- [ ] No em-dashes (search for `—`).
- [ ] Figures numbered, captioned, with alt text.
- [ ] No trailing references list; post ends on the Summary or Closing Thought.
