# Publications Page Update Workflow

## Goal

Plan the next refinement of the Publications page after the latest online edits were pulled into the local `academic` branch.

Requested outcomes:

- Add clear instructions for adding a new publication through GitHub online editing.
- Add a reusable publication template in `_publications`.
- Fix the publication numbering.
- Keep only the first 10 papers visible in each section.
- Fold older papers in both `Journal Articles` and `Conference Papers`.
- Review this plan before changing the live Publications page behavior.

## Current Status

The local repo has already been updated from GitHub:

```bash
git pull --ff-only origin academic
```

The pull brought in online edits to:

```text
_config.yml
_pages/about.md
_pages/cv.md
```

Current branch:

```text
refine-publications-numbering-folding
```

Implementation status:

```text
Implemented locally for review.
```

The live `academic` branch should be updated only after the local build and GitHub Actions pass.

## Publication Files

Each publication is one Markdown file in:

```text
_publications/
```

Example:

```text
_publications/2024-01-05-occluded-person-retrieval-with-hierarchical-feature-optimization.md
```

The filename controls sorting together with the `date` field. Use this pattern:

```text
YYYY-MM-DD-short-paper-title.md
```

The important front matter fields are:

```yaml
title: "Paper title"
collection: publications
category: manuscripts
permalink: /publication/short-paper-url
date: 2026-01-01
venue: "Journal or conference name, year"
paperurl: "https://paper-link"
citation: "Full citation"
```

Use this category value for journal papers:

```yaml
category: manuscripts
```

Use this category value for conference papers:

```yaml
category: conferences
```

## New Publication Template

A reusable template file has been added here:

```text
_publications/publication-template.md
```

This file includes:

```yaml
published: false
```

Keep `published: false` in the template file so it does not appear as a real publication.

When adding a real new publication, copy the template, rename the copy, edit the fields, and remove:

```yaml
published: false
```

## How To Add A New Publication Online

Use GitHub online editing when you only need to add or edit one publication file. For larger changes, local editing is better because you can run the site build before pushing.

Online workflow:

1. Open the repository on GitHub.
2. Switch to the `academic` branch.
3. Open `_publications/publication-template.md`.
4. Click the copy/raw/edit route you prefer and create a new file in `_publications/`.
5. Name the file with this pattern:

```text
YYYY-MM-DD-short-paper-title.md
```

Example:

```text
_publications/2026-01-01-example-paper-title.md
```

6. Paste the template content into the new file.
7. Replace the title, date, venue, paper URL, citation, permalink, and body text.
8. Set the category:

```yaml
category: manuscripts
```

or:

```yaml
category: conferences
```

9. Remove this line from the real publication file:

```yaml
published: false
```

10. Commit the change directly to `academic`, or create a pull request if you want to review first.

After editing online, always pull before doing local work:

```bash
git checkout academic
git pull --ff-only origin academic
```

## Where To Fix Publication Numbering

The publication numbering is controlled in:

```text
_pages/publications.html
```

This workflow implementation updates that file directly.

The current numbering is wrong because it starts from the total number of all publications and subtracts through every paper, even when the current section is only showing journals or only showing conferences.

That means conference papers can show numbers like:

```text
20, 17, 16, 7, 5, 4
```

instead of a clean section-local sequence.

Recommended behavior:

- `Journal Articles` should number only journal papers.
- `Conference Papers` should number only conference papers.
- Each section should count backward inside that section.

Example if there are 14 journal papers:

```text
14, 13, 12, ... 1
```

Example if there are 6 conference papers:

```text
6, 5, 4, ... 1
```

Recommended Liquid logic:

```liquid
{% assign journal_papers = site.publications | where: "category", "manuscripts" | sort: "date" | reverse %}
{% assign journal_count = journal_papers | size %}

{% for post in journal_papers %}
  {% assign paper_number = journal_count | minus: forloop.index0 %}
  {% include publication-list-item.html number=paper_number %}
{% endfor %}
```

Use the same pattern for conference papers:

```liquid
{% assign conference_papers = site.publications | where: "category", "conferences" | sort: "date" | reverse %}
{% assign conference_count = conference_papers | size %}

{% for post in conference_papers %}
  {% assign paper_number = conference_count | minus: forloop.index0 %}
  {% include publication-list-item.html number=paper_number %}
{% endfor %}
```

## Folding Older Papers After 10

The requested page behavior:

- Show the newest 10 journal papers.
- Fold older journal papers.
- Show the newest 10 conference papers.
- Fold older conference papers.

This workflow implementation uses native HTML `<details>` sections, so visitors can open older papers without loading a separate page.

Recommended implementation:

- Use Liquid to split each section into visible and older papers.
- Use the HTML `<details>` element for the folded older papers.
- Avoid JavaScript unless the default `<details>` behavior is not enough.

Recommended setting inside `_pages/publications.html`:

```liquid
{% assign visible_limit = 10 %}
```

Recommended structure for each section:

```liquid
{% assign journal_papers = site.publications | where: "category", "manuscripts" | sort: "date" | reverse %}
{% assign journal_count = journal_papers | size %}

<h2>Journal Articles</h2>
<ul class="publication-list">
{% for post in journal_papers limit: visible_limit %}
  {% assign paper_number = journal_count | minus: forloop.index0 %}
  {% include publication-list-item.html number=paper_number %}
{% endfor %}
</ul>

{% if journal_count > visible_limit %}
<details class="publication-list__fold">
  <summary>Show older journal articles</summary>
  <ul class="publication-list">
  {% for post in journal_papers offset: visible_limit %}
    {% assign paper_number = journal_count | minus: forloop.index0 | minus: visible_limit %}
    {% include publication-list-item.html number=paper_number %}
  {% endfor %}
  </ul>
</details>
{% endif %}
```

Use the same pattern for `conference_papers`.

Important note: the folded-loop numbering needs separate testing, because Liquid's `forloop.index0` resets to `0` inside the folded loop. The intended first folded number is:

```text
total number in the section - visible_limit
```

For example, if there are 14 journal papers, the visible list should show:

```text
14, 13, 12, 11, 10, 9, 8, 7, 6, 5
```

The folded list should start with:

```text
4
```

## Styling To Add

The fold control can be styled in:

```text
_sass/_archive.scss
```

Suggested minimal styling:

```scss
.publication-list__fold {
  margin: 0 0 1.5em;
}

.publication-list__fold summary {
  cursor: pointer;
  color: mix(#fff, $gray, 25%);
}
```

This keeps the change consistent with the current theme colors.

## Recommended Implementation Steps

1. Create a new branch from the updated `academic` branch.

```bash
git checkout academic
git pull --ff-only origin academic
git checkout -b refine-publications-folding
```

2. Edit `_pages/publications.html`.
3. Replace the current mixed numbering logic with separate journal and conference arrays.
4. Add `visible_limit = 10`.
5. Add `<details>` folded sections for papers after the first 10.
6. Add optional fold styling in `_sass/_archive.scss`.
7. Run a local build if Ruby/Bundler is available.
8. Push the branch to GitHub.
9. Confirm GitHub Actions passes.
10. Merge into `academic` only after review and approval.

## Validation Checklist

Check the Publications page after implementation:

- Journal papers show clean numbers inside the journal section.
- Conference papers show clean numbers inside the conference section.
- Newest 10 journal papers are visible.
- Older journal papers are folded.
- Newest 10 conference papers are visible.
- Older conference papers are folded.
- Clicking a paper title opens the detail page.
- The template file does not appear as a publication.
- GitHub Pages deployment succeeds.

## Recommendation

Use GitHub online editing for simple publication additions. Use local editing for changes to `_pages/publications.html`, `_sass/_archive.scss`, navigation, or other site behavior because those changes should be built and checked before deployment.
