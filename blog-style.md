# Blog Style Guide

Conventions for posts in `blog/_posts/`, distilled from the 2026 posts ("Web Dev for Data People" is the cleanest reference). Follow these so new posts read as one blog. A copy-paste skeleton is at the bottom.

## File and front matter

- Filename: `YYYY-MM-DD-slug.md`. The slug should match the title (title "Web Dev for Data People" gives `web-dev-for-data-people`).
- Front matter:

```yaml
---
layout: post
title: "Short Hook: Subtitle That Says What the Reader Gets"
description: >
  Two to four sentences. What the post covers, the angle it takes, and the
  concrete thing the reader leaves with (a framework, a repo, a skill file).
author: author1
comments: true
---
```

- Title: Title Case, usually `Hook: Subtitle`. The hook is a phrase with a twist ("One DP Engine, Thirteen Rosalind Problems", "An Optimization Framework, Not Just an Optimization Script").
- Description: name the source if the post is built on a book, paper or course (author and title), and the main artifacts or ideas.
- Assets live under `assets/blog/YYYY-MM-DD/` (the post's date) and are referenced as `/assets/blog/YYYY-MM-DD/name.ext`. Name figures `figNN-short-name.svg` (keep a `.png` export next to any `.svg`).

## Structure

1. **Intro, 1 to 4 short paragraphs, before the table of contents.** Cover, in this order:
   - Why the post exists (a personal trigger or a problem I ran into). Open with a concrete scene or observation, not a definition ("Almost every analysis I have worked on ends in the same small moment: someone needs to *see* it.").
   - What the reader gets (the main ideas or artifacts).
   - How the post is organised, in a sentence or two ("This post walks through that choice in four moves. First, ... Second, ... Last, ...").
   - Prerequisites, as one plain sentence inside the intro (no separate `**Prerequisite**` line).
   - The companion repo, if there is one, as an inline link in the intro.
2. **Table of contents**: a bullet list of anchor links to the `##` sections, placed right after the intro. No "Table of Contents" heading. Sub-bullets are allowed for long posts.
3. **Body**: one `##` per major idea, `###` for sub-parts. Headings are short phrases with a hook ("Two Gaps That Bite Later", "One Table, Three Questions"), not generic labels. No section numbers, except for a real sequence ("Stage 1: ...", "Use Case 2: ...").
4. **Ending**: finish on `## Summary` or `## Closing Thought`. The last paragraph is the takeaway. Nothing follows it, no trailing separators or link lists.

### Inside a section

- Open each `##` with one or two sentences that frame the question the section answers, then the detail.
- Use a **running example** for the whole post, introduced in bold in the intro or first section ("As the use case, I picked **NYC taxi fare prediction**").
- For a sequence of steps or stages, use a numbered list with a bold label ending in a period: `1. **Fully local.** Code and data on the laptop.`
- To split a long section without another heading, start paragraphs with a bold lead-in: `**Feature engineering.** For each day ...`
- End major sections on the practical consequence: "My default for anyone working on a laptop: ...".

### Summary or Closing Thought

- Name the throughline in bold in the first sentence ("The throughline is to **decompose before you choose**.").
- Then the two to four lessons, as a short paragraph or a bullet list with bold lead-ins.
- Optionally one line on what comes next or where the code lives. The final sentence is the idea I want the reader to keep.

## Tone

- First person, practitioner voice: "I have worked on", "the rule I keep repeating to myself", "my default suggestion".
- Address the reader as "you" when giving advice.
- Concrete before abstract. Show the example, then name the idea.
- Be honest about limits: say what was and was not verified or run ("Everything shown ran end to end on Free Edition in October 2026", "I have not run every starter snippet myself"). Say when a snippet is simplified from the real code.
- Plain words, short paragraphs. Expand an acronym on first use: "single-page application (SPA)".
- Bold key terms on first use; use italics for light emphasis (*see*, *lightest*).
- Use `>` blockquotes for the one rule or definition worth remembering, at most one or two per post.
- British spelling (organised, behaviour, recognise).

## Punctuation

- **No em-dashes.** Do not use `—` (or ` — `). Rewrite with a comma, colon, period, or parentheses instead.
- Prefer commas and periods over long chained clauses.
- A colon to introduce a list, table or punchline: "The shape of the fix is always the same: ...".

## Figures

Use either form, with a caption and the figure class:

```markdown
![Fig01](/assets/blog/YYYY-MM-DD/name.svg){:data-width="900" data-height="500"}
Fig. 1. What the figure shows and what to notice.
{:.figure}
```

or `<img src="/assets/blog/YYYY-MM-DD/name.png" alt="Describe the image" />` for images with no caption. Number figures in order and always give alt text.

- The caption says what to notice, not only what it is, and can run two or three sentences.
- After the figure, a paragraph points out what is easy to miss ("Two things in this map are easy to miss. First, ...").

## Math, code, tables

- Math: `$$...$$` for both inline and block (kramdown).
- Code: fenced blocks with a language tag. Prefer real, runnable code over pseudocode. Use inline code for identifiers, file names and commands (`build_features`, `/docs`, `pip`).
- Tables for comparisons and lookups, with a short lead-in sentence ending in a colon, and a paragraph after that draws the conclusion.

## References

- Cite inline: `text<sup><a href="URL">(1)</a></sup>`, numbered in order of appearance, or a normal markdown link `[text](URL)`.
- Do not add a trailing list of bare URLs; the inline links are the references.

## Pre-publish checklist

- [ ] `layout: post` is set and not commented out.
- [ ] Slug matches the title; date matches the assets folder.
- [ ] Intro covers why, what, how, prerequisites; sits before the ToC.
- [ ] Every ToC anchor matches a heading.
- [ ] A blank line before every heading.
- [ ] No em-dashes (search for `—`).
- [ ] Figures use `/assets/blog/YYYY-MM-DD/` paths, are numbered and captioned, and have alt text.
- [ ] Every claim about a tool or version links to its source.
- [ ] No trailing references list; post ends on the Summary or Closing Thought.

## Skeleton

````markdown
---
layout: post
title: "Short Hook: Subtitle That Says What the Reader Gets"
description: >
  What the post covers and the angle it takes. The concrete thing the reader
  leaves with.
author: author1
comments: true
---

A concrete scene or problem that made me write this. Why it matters to someone who does what I do.

What changed, or what most explanations skip, and what this post offers instead.

This post walks through it in three moves. First, ... Second, ... Last, ... You only need X and Y; no Z background is assumed. The code is on [GitHub](https://github.com/hovinh/REPO).

- [First Idea With a Hook](#first-idea-with-a-hook)
- [Second Idea With a Hook](#second-idea-with-a-hook)
- [Summary](#summary)

## First Idea With a Hook

One or two sentences framing the question this section answers.

![Fig01](/assets/blog/YYYY-MM-DD/fig01-name.svg){:data-width="900" data-height="500"}
Fig. 1. What the figure shows and what to notice.
{:.figure}

> The one rule worth remembering.

## Second Idea With a Hook

### A Sub-part

A lead-in sentence for the table:

| Option | Good for |
|---|---|
| A | ... |

What the table tells you, and my default.

## Summary

The throughline is to **one bolded idea**. Two to four lessons that recur.

The sentence I want the reader to keep.
````
