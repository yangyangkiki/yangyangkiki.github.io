# Publications Direct-Link Workflow

## Goal

Plan the next Publications page refinement.

Requested outcome:

- Keep the current sorting and section numbering.
- Do not make publication titles open the local generated introduction/detail page.
- Make each publication title open an external paper page instead, such as DOI, publisher, arXiv, Google Scholar, or project paper page.
- Keep optional `Demo`, `Code`, `Dataset`, or `Project` links beside the title when those links are provided.
- Keep the venue line, for example:

```text
Published in The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK, 2024
```

- Make adding new publications easier by using front matter only, with no required local introduction text.

## Current Good Behavior To Keep

The current Publications page already has the correct sorting and section-local numbering:

- `Journal Articles` are sorted newest first and numbered within the journal section.
- `Conference Papers` are sorted newest first and numbered within the conference section.
- Only the newest 10 papers per section are shown before older papers fold.

Do not change this part unless the sorting rule changes later.

## Implementation Status

This workflow has been implemented in the `refine-publications-direct-links` branch:

- `_includes/publication-list-item.html` now links publication titles to `paperurl`.
- Optional `demourl`, `codeurl`, `dataseturl`, and `projecturl` fields render beside the title.
- Existing body `Links:` entries were moved into front matter.
- The current Publications sorting, numbering, and folding logic was kept unchanged.

## Recommended Data Model

Use each `_publications/*.md` file as a metadata record.

The title should link to:

```yaml
paperurl: "https://..."
```

The `paperurl` can be any preferred external page:

- DOI page
- publisher page
- arXiv page
- Google Scholar result page
- project paper page
- PDF link

Optional extra links should move into front matter:

```yaml
demourl: "https://..."
codeurl: "https://..."
dataseturl: "https://..."
projecturl: "https://..."
```

Only include optional fields when there is a real link. If a paper has no demo or code, delete those fields from the copied file.

## New Template

A direct-link template has been added here:

```text
_publications/publication-direct-link-template.md
```

It includes:

```yaml
published: false
```

Keep `published: false` in the template file. When creating a real publication by copying this template, remove `published: false` from the copied file.

## Example Publication Entry

Example for Motion Avatar:

```yaml
---
title: "Motion Avatar: Generate Human and Animal Avatars with Arbitrary Motion"
collection: publications
category: conferences
date: 2024-01-01
venue: "The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK"
paperurl: "https://arxiv.org/abs/2405.11286"
demourl: "https://steve-zeyu-zhang.github.io/MotionAvatar/"
citation: "Zeyu Zhang*, Yiran Wang*, Biao Wu*, Shuo Chen, Zhiyuan Zhang, Shiya Huang, Wenbo Zhang, Meng Fang, Ling Chen, Yang Zhao. Motion Avatar: Generate Human and Animal Avatars with Arbitrary Motion. The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK."
---
```

The Publications page should render it like:

```text
Motion Avatar: Generate Human and Animal Avatars with Arbitrary Motion, Demo
Published in The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK, 2024
```

If both demo and code are provided:

```text
Motion Avatar: Generate Human and Animal Avatars with Arbitrary Motion, Demo, Code
Published in The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK, 2024
```

## Files To Update During Implementation

Main rendering file:

```text
_includes/publication-list-item.html
```

Current behavior:

```liquid
<a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a>
```

Recommended new behavior:

```liquid
{% if post.paperurl %}
  {% assign title_url = post.paperurl %}
{% else %}
  {% assign title_url = base_path | append: post.url %}
{% endif %}

<a href="{{ title_url }}">{{ post.title }}</a>
{% if post.demourl %}, <a href="{{ post.demourl }}">Demo</a>{% endif %}
{% if post.codeurl %}, <a href="{{ post.codeurl }}">Code</a>{% endif %}
{% if post.dataseturl %}, <a href="{{ post.dataseturl }}">Dataset</a>{% endif %}
{% if post.projecturl %}, <a href="{{ post.projecturl }}">Project</a>{% endif %}
```

Keep the existing venue line:

```liquid
<span class="publication-list__venue">
  Published in <i>{{ post.venue }}</i>, {{ post.date | default: "1900-01-01" | date: "%Y" }}
</span>
```

The fallback to `post.url` is useful only when a publication has no `paperurl`. Normal new entries should use `paperurl`.

## Existing Link Migration

Some existing publication files currently store extra links in the body as Markdown:

```md
Links: [Demo](...)
Links: [Code](...)
Links: [Dataset](...)
```

For the new direct-link list page, move those links into front matter.

Example before:

```yaml
paperurl: "https://arxiv.org/abs/2405.11286"
```

```md
Links: [Demo](https://steve-zeyu-zhang.github.io/MotionAvatar/)
```

Example after:

```yaml
paperurl: "https://arxiv.org/abs/2405.11286"
demourl: "https://steve-zeyu-zhang.github.io/MotionAvatar/"
```

Recommended migration targets:

```text
Demo    -> demourl
Code    -> codeurl
Dataset -> dataseturl
Project -> projecturl
```

The body text can remain for now, but the Publications list should no longer need it.

## How To Add A New Publication Online

Use GitHub online editing for simple publication additions.

1. Open the repository on GitHub.
2. Switch to the `academic` branch.
3. Open:

```text
_publications/publication-direct-link-template.md
```

4. Copy the template content.
5. Create a new file in `_publications/`.
6. Use this filename pattern:

```text
YYYY-MM-DD-short-paper-title.md
```

Example:

```text
_publications/2026-01-01-example-paper-title.md
```

7. Paste the template content.
8. Replace the title, date, venue, paper URL, category, and citation.
9. Delete optional link fields that are not used.
10. Remove this line from the copied real publication:

```yaml
published: false
```

11. Commit the file to `academic`, or open a pull request if you want to review first.

After editing online, pull before doing local work:

```bash
git checkout academic
git pull --ff-only origin academic
```

## Category Rules

Journal paper:

```yaml
category: manuscripts
```

Conference paper:

```yaml
category: conferences
```

The category controls which section the paper appears in.

## Venue Rule

Use `venue` for the publication venue text, not the full citation.

Recommended:

```yaml
venue: "The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK"
date: 2024-01-01
```

This renders as:

```text
Published in The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK, 2024
```

Avoid putting a final duplicate year at the end of `venue`, because the template already adds the year from `date`.

## Implementation Steps

1. Create a new branch from `academic`.

```bash
git checkout academic
git pull --ff-only origin academic
git checkout -b refine-publications-direct-links
```

2. Update `_includes/publication-list-item.html` so the title links to `paperurl`.
3. Render optional `Demo`, `Code`, `Dataset`, and `Project` links beside the title.
4. Move existing body links into front matter for papers that already have demo/code/dataset links.
5. Keep `_pages/publications.html` sorting, numbering, and folding logic unchanged.
6. Run a local Jekyll build.
7. Push the branch and confirm GitHub Actions passes.
8. Merge into `academic` only after review.

## Validation Checklist

- Publication title opens `paperurl`, not the local generated publication page.
- If `demourl` exists, `Demo` appears beside the title.
- If `codeurl` exists, `Code` appears beside the title.
- If `dataseturl` exists, `Dataset` appears beside the title.
- Venue line still appears under each title.
- Journal and conference sorting remains correct.
- Journal and conference numbering remains correct.
- Folded older papers still work.
- The direct-link template does not appear as a real publication.
