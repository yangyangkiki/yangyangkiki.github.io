# Publication, Teaching, CV, and Talks Refinement Workflow

## Goal

Plan the next refinement pass for the live Academic Pages site.

Requested outcomes:

- Publications page should show a compact numbered list, not full details.
- Each publication title should link to its own detail page.
- Publications page should show paper title and venue only.
- Publications should be numbered in reverse order, for example `20, 19, 18, ... 1`.
- Publications page should include a visible `Conference Papers` section.
- Teaching page should contain only teaching experience.
- CV page should remove the `Download CV` link.
- CV page should add an `Education` section.
- CV page should rename `Experience` to `Work Experience`.
- Awards and Grants should move out of CV/Teaching and into the Activities/Talks page.
- The current `Activities` page/navigation should be renamed to `Talks`, pending one naming decision below.

Current production branch:

```text
academic
```

Recommended branch for this refinement:

```bash
git checkout academic
git pull origin academic
git checkout -b refine-publications-teaching-cv
```

Use local editing for this pass. This change touches several files and Liquid templates, so GitHub web editing is not the best option.

## Important Naming Decision

There is a small ambiguity to confirm before implementation:

```text
"put Awards and Grants to page activities. and change page name activities to talks."
```

Current files:

```text
_pages/professional-activities.md   # current Activities page
_pages/talks.html                   # old template Talks page, currently empty because there are no _talks entries
_data/navigation.yml                # top navigation
```

Recommended interpretation:

- Rename the visible navigation label from `Activities` to `Talks`.
- Repurpose the current activities/service page as the new `Talks` page.
- Move Awards and Grants into that page.
- Avoid having two pages with the same `/talks/` permalink.

Implementation choice:

1. If only the navigation label should change, keep the URL as `/professional-activities/` and change the nav title to `Talks`.
2. If the page URL should also become `/talks/`, replace the old `_pages/talks.html` template page with the activities/talks content and remove or archive `_pages/professional-activities.md`.

Recommended choice: option 2, because the user-facing page name and URL will match.

## Files to Edit

| Area | Files |
| --- | --- |
| Publications list page | `_pages/publications.html` |
| Publication list rendering | new `_includes/publication-list-item.html` or direct Liquid in `_pages/publications.html` |
| Publication metadata | `_publications/*.md` |
| Teaching page | `_pages/teaching.md` |
| CV page | `_pages/cv.md` |
| Activities/Talks page | `_pages/professional-activities.md`, `_pages/talks.html` |
| Navigation | `_data/navigation.yml` |
| Site build check | `.github/workflows/jekyll-build-check.yml` |

## 1. Publications Page Refinement

### Desired Behavior

On `/publications/`, each paper should show only:

```text
[number] Paper title
Published in Venue, Year
```

The paper title should link to the individual publication detail page.

Do not show on the list page:

- full citation
- paper figure
- paper download link
- code/demo/dataset links
- body text from the publication file

Those details should remain available after clicking the paper title.

### Preserve Detail Pages

Keep each `_publications/*.md` file as the detail page source.

Example:

```text
_publications/2024-01-01-motion-avatar-generate-human-and-animal-avatars-with-arbitrary-motion.md
```

The individual page can still include:

- figure
- citation
- paper link
- code/demo/dataset links
- notes

### Add a Compact Publication List Include

Create:

```text
_includes/publication-list-item.html
```

Suggested content:

```liquid
{% include base_path %}

<li class="publication-list__item">
  <span class="publication-list__number">{{ include.number }}.</span>
  <a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a>
  <br />
  <span class="publication-list__venue">
    Published in <i>{{ post.venue }}</i>, {{ post.date | default: "1900-01-01" | date: "%Y" }}
  </span>
</li>
```

This avoids changing `_includes/archive-single.html`, which is shared by other pages.

### Update Publications Page

Edit:

```text
_pages/publications.html
```

Replace the current archive rendering with explicit sections for journal and conference papers.

Suggested structure:

```liquid
---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

{% if site.author.googlescholar %}
  <div class="wordwrap">You can also find my articles on <a href="{{site.author.googlescholar}}">my Google Scholar profile</a>.</div>
{% endif %}

{% include base_path %}

{% assign paper_number = site.publications | size %}

<h2>Journal Articles</h2>
<ol class="publication-list" reversed>
{% for post in site.publications reversed %}
  {% if post.category == "manuscripts" %}
    {% include publication-list-item.html number=paper_number %}
    {% assign paper_number = paper_number | minus: 1 %}
  {% endif %}
{% endfor %}
</ol>

<h2>Conference Papers</h2>
<ol class="publication-list" reversed>
{% for post in site.publications reversed %}
  {% if post.category == "conferences" %}
    {% include publication-list-item.html number=paper_number %}
    {% assign paper_number = paper_number | minus: 1 %}
  {% endif %}
{% endfor %}
</ol>
```

Note: this keeps one global reverse number across both sections. If numbering should restart in each section, use one counter for journal articles and another counter for conference papers.

### Confirm Conference Paper Metadata

The conference section depends on publication front matter:

```yaml
category: conferences
```

Check:

```bash
grep -R "category: conferences" _publications
grep -R "category: manuscripts" _publications
```

If a conference paper is missing from `Conference Papers`, fix its front matter.

Example:

```yaml
---
title: "Motion Avatar: Generate Human and Animal Avatars with Arbitrary Motion"
collection: publications
category: conferences
permalink: /publication/2024-motion-avatar-generate-human-and-animal-avatars-with-arbitrary-motion
date: 2024-01-01
venue: "The 35th British Machine Vision Conference (BMVC 2024), Glasgow, UK"
paperurl: "https://arxiv.org/abs/2405.11286"
citation: "..."
---
```

### Optional Publication Style

If the list needs spacing, add a small style block later in a scoped Sass file instead of inline styling.

Potential file:

```text
_sass/_archive.scss
```

Suggested class names:

```text
publication-list
publication-list__item
publication-list__number
publication-list__venue
```

## 2. Teaching Page Refinement

Current teaching page contains:

- Teaching
- Experience
- PhD Supervision
- Visiting Students, Research Interns, and Research Assistants
- Awards and Grants

Requested behavior: keep only teaching experience.

Edit:

```text
_pages/teaching.md
```

Keep:

```markdown
---
layout: single
title: "Teaching"
permalink: /teaching/
author_profile: true
---

## Teaching

- Computer Vision, Subject code: CSE5CV, Semester 2 2024; Subject Coordinator/Lecturer, La Trobe University.
- Artificial Intelligence Fundamentals, Subject code: CSE2AIF/CSE4002, Semester 1 2024; Subject Coordinator/Lecturer, La Trobe University.
- Applied Machine Learning, Course code: COMP SCI 4816, 2nd Semester 2022; Guest lecturer, The University of Adelaide.
- Image Processing and Machine Vision, Course code: 6302ENG, 2nd Semester 2022; Guest lecturer, Griffith University.
- Software Engineering and Project, Course code: COMP SCI 3006, 2nd Semester 2022; TA, The University of Adelaide.
```

Remove from Teaching:

- Experience
- PhD Supervision
- Visiting Students, Research Interns, and Research Assistants
- Awards and Grants

If the page should look more like the original Academic Pages template later, convert each teaching entry into a separate `_teaching/*.md` file and restore `_pages/teaching.html` as an archive page. For now, the direct Markdown page is simpler and matches the current content.

## 3. CV Page Refinement

Edit:

```text
_pages/cv.md
```

Requested changes:

- Remove `[Download CV](/files/CV_YangZHAO_2021.pdf)`.
- Add `Education`.
- Rename `Experience` to `Work Experience`.
- Remove `Awards and Grants` from CV.
- Keep link to publications.
- Optionally keep a link to the Talks/Activities page.

Suggested structure:

```markdown
---
layout: single
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

## Education

- PhD student, Griffith University, 02.2018-01.2022.
- Visiting PhD student, The University of Adelaide, 03.2018-01.2022.

## Work Experience

- Continuing Lecturer (Assistant Professor), La Trobe University, 08.2023-Now.
- Research Fellow, Australian Institute for Machine Learning (AIML), The University of Adelaide, 09.2021-08.2023.

## Publications

See the [Publications page](/publications/) for selected publications.

## Talks and Activities

See the [Talks page](/talks/) for awards, grants, service, and reviewing activities.
```

Before implementing, confirm whether there are additional education details to add, such as undergraduate or master's degrees.

## 4. Move Awards and Grants

Awards and Grants currently appear in:

```text
_pages/teaching.md
_pages/cv.md
```

Move them to the page that will replace or rename Activities/Talks.

Current awards:

```markdown
## Awards and Grants

- Top Publication Incentive Award, Griffith University, 2020/2021/2022.
- SCEMS CaRE and Beyond Scheme Grant, La Trobe University, 2024.
```

Remove that section from Teaching and CV after adding it to the new page.

## 5. Rename Activities to Talks

### Recommended Implementation

Use `/talks/` as the public page.

Steps:

1. Replace the old template `_pages/talks.html` with a Markdown page or update its content directly.
2. Move content from `_pages/professional-activities.md` into the new Talks page.
3. Add Awards and Grants to the same page.
4. Remove `_pages/professional-activities.md` after confirming no links still point to `/professional-activities/`.
5. Update `_data/navigation.yml`.

Suggested new `_pages/talks.md` or rewritten `_pages/talks.html` content:

```markdown
---
layout: single
title: "Talks"
permalink: /talks/
author_profile: true
---

## Awards and Grants

- Top Publication Incentive Award, Griffith University, 2020/2021/2022.
- SCEMS CaRE and Beyond Scheme Grant, La Trobe University, 2024.

## Journal Reviewer

- IEEE Transactions on Multimedia (TMM)
- Pattern Recognition (PR)
- IEEE Journal of Biomedical and Health Informatics (J-BHI)
- IEEE Transactions on Artificial Intelligence (TAI)
- IEEE Transactions on Circuits and Systems for Video Technology (TCSVT)

## Conference Service and Reviewing

- Co-organizer of AIiH 2024 Special Session
- PC member of CASA 2024
- Reviewer for CVPR 2024
- Reviewer for MICCAI 2023
- Reviewer for ECCV 2022
- Reviewer for CVPR 2022
- Reviewer for AAAI 2022
- Reviewer for ICCV 2021
- Reviewer for CVPR 2021
- Reviewer for IJCAI 2021
- Reviewer for ICONIP 2020
```

### Navigation Update

Edit:

```text
_data/navigation.yml
```

Change:

```yaml
  - title: "Activities"
    url: /professional-activities/
```

To:

```yaml
  - title: "Talks"
    url: /talks/
```

### Avoid Duplicate Talk Pages

Do not keep two files with:

```yaml
permalink: /talks/
```

If creating `_pages/talks.md`, delete or rewrite `_pages/talks.html`.

## 6. Validation Before Commit

Run lightweight checks:

```bash
git status --short
```

Check front matter:

```bash
python3 - <<'PY'
from pathlib import Path
import yaml

for folder in ["_pages", "_publications"]:
    for path in Path(folder).glob("*"):
        if path.suffix not in {".md", ".html"}:
            continue
        text = path.read_text(encoding="utf-8")
        if not text.startswith("---\n"):
            continue
        end = text.find("\n---", 4)
        if end == -1:
            raise SystemExit(f"{path}: missing closing front matter")
        yaml.safe_load(text[4:end]) or {}

print("front matter OK")
PY
```

Check publication categories:

```bash
grep -R "category: conferences" _publications | wc -l
grep -R "category: manuscripts" _publications | wc -l
```

Check for duplicate `/talks/` permalinks:

```bash
grep -R "permalink: /talks/" _pages
```

Expected result: only one file.

Check for stale Activities links:

```bash
grep -R "/professional-activities/" _pages _data _includes
```

Expected result after full rename: no stale links, unless intentionally preserving redirects.

## 7. Commit and Push Refinement Branch

```bash
git add -A
git commit -m "Refine publications teaching CV and talks"
git push -u origin refine-publications-teaching-cv
```

Wait for GitHub Actions:

```text
https://github.com/yangyangkiki/yangyangkiki.github.io/actions
```

Do not merge into `academic` unless `Jekyll build check` passes.

## 8. Live Verification After Merge

After merging to `academic`, verify:

```text
https://yangyangkiki.github.io/publications/
https://yangyangkiki.github.io/teaching/
https://yangyangkiki.github.io/cv/
https://yangyangkiki.github.io/talks/
```

Expected checks:

- Publications page has compact linked paper rows.
- Publications page has `Journal Articles`.
- Publications page has `Conference Papers`.
- Publication numbers count backward.
- Clicking a paper title opens the detail page.
- Teaching page only shows teaching experience.
- CV page has no `Download CV` link.
- CV page has `Education`.
- CV page has `Work Experience`.
- Awards and Grants appear on Talks page.
- Top navigation shows `Talks`, not `Activities`.

Optional command checks:

```bash
curl -L -sS -o /tmp/publications.html -w "%{http_code}\n" https://yangyangkiki.github.io/publications/
curl -L -sS -o /tmp/teaching.html -w "%{http_code}\n" https://yangyangkiki.github.io/teaching/
curl -L -sS -o /tmp/cv.html -w "%{http_code}\n" https://yangyangkiki.github.io/cv/
curl -L -sS -o /tmp/talks.html -w "%{http_code}\n" https://yangyangkiki.github.io/talks/
```

Expected result:

```text
200
200
200
200
```

## Approval Points

Ask for approval before:

- Replacing the old `_pages/talks.html` page.
- Removing `_pages/professional-activities.md`.
- Merging the branch into `academic`.
- Pushing directly to `academic`.

## Implementation Order

1. Confirm whether `Activities` should become `/talks/` or only be labeled `Talks`.
2. Refine publication list rendering.
3. Audit publication categories and fix missing `conferences` metadata.
4. Simplify Teaching page.
5. Refine CV page.
6. Move Awards and Grants into Talks/Activities page.
7. Update navigation.
8. Run validation checks.
9. Commit and push refinement branch.
10. Wait for GitHub Actions.
11. Merge to `academic` after approval.
12. Verify live site.

