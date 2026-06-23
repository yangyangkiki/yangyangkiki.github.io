# Site Refinement Workflow

## Goal

Refine the live Academic Pages website without risking the current production site.

Current setup:

- Live branch: `academic`
- Live URL: `https://yangyangkiki.github.io`
- Site type: Jekyll Academic Pages
- Build validation: GitHub Actions workflow in `.github/workflows/jekyll-build-check.yml`
- Local Ruby may not be available, so GitHub Actions is the main build check.

## Working Rule

Do not edit and push directly to `academic` for non-trivial refinements.

Use a short-lived branch for each refinement pass, push it, let GitHub Actions validate it, then merge into `academic` only after review.

## Local Laptop vs GitHub Web Editor

Recommended default: use the local laptop workflow for most changes.

Local editing is better when:

- You are changing more than one file.
- You are adding images, PDFs, or publication figures.
- You want to keep changes on a named refinement branch.
- You want to review the full diff before publishing.
- You want GitHub Actions to validate the branch before merging.

GitHub web editing is acceptable when:

- The change is small and isolated.
- You are editing one Markdown file only, such as adding one recent news item.
- You are comfortable using GitHub's branch option in the web editor.

Avoid direct GitHub web commits to `academic` unless the change is urgent and tiny. Prefer choosing "Create a new branch for this commit" in the GitHub web editor, then let GitHub Actions check the branch before merging.

### Local Laptop Workflow

Use this for publications, images, CV updates, navigation changes, or multiple-page edits.

```bash
git checkout academic
git pull origin academic
git checkout -b refine-short-description
```

Edit the files locally, then run the lightweight checks:

```bash
git status --short
```

Commit and push:

```bash
git add -A
git commit -m "Describe the refinement"
git push -u origin refine-short-description
```

Then check GitHub Actions before merging.

### GitHub Web Editor Workflow

Use this only for small Markdown-only edits.

1. Open the file on GitHub, for example `_pages/about.md`.
2. Click the pencil icon.
3. Make the edit.
4. In "Commit changes", choose "Create a new branch for this commit and start a pull request".
5. Name the branch, for example `refine-recent-news`.
6. Open the pull request.
7. Wait for `Jekyll build check` to pass.
8. Merge only after the check passes.

If GitHub offers only a direct commit to `academic`, do not use it for anything larger than a one-line correction.

## Recommended Refinement Cycle

### 1. Start from the Live Branch

```bash
git checkout academic
git pull origin academic
git status
```

Create a branch named for the refinement:

```bash
git checkout -b refine-homepage-copy
```

Examples:

```text
refine-homepage-copy
refine-publication-format
refine-sidebar-profile
refine-navigation
refine-cv-page
```

### 2. Choose the Refinement Type

Use this map to find the right files.

| Refinement | Main files |
| --- | --- |
| Homepage biography, news, contact | `_pages/about.md` |
| Sidebar name, avatar, links, title | `_config.yml` |
| Top navigation | `_data/navigation.yml` |
| Teaching, experience, supervision, awards | `_pages/teaching.md` |
| CV page text and PDF link | `_pages/cv.md`, `files/` |
| Professional activities/service | `_pages/professional-activities.md` |
| Publication metadata | `_publications/*.md` |
| Publication list rendering | `_pages/publications.html`, `_includes/archive-single.html` |
| Images and figures | `images/`, `images/paperFig/` |
| CSS/theme-level visual changes | `_sass/`, `assets/css/main.scss` |

### 3. Make Small, Reviewable Changes

Prefer one refinement theme per branch.

Good examples:

- Update homepage wording and recent news.
- Improve publication titles/citations and add missing project links.
- Replace profile photo and update sidebar links.
- Simplify navigation.
- Improve CV page organization.

Avoid mixing unrelated work in one branch, for example changing homepage copy, CSS, publications, and CV PDF all together.

### 4. Local Lightweight Checks

These checks do not require Ruby:

```bash
git status --short
grep -R "Your Name\\|academicpages\\|Red Brick University\\|Paper Title Number" _config.yml _pages _publications 2>/dev/null
```

Check important local asset references manually:

```bash
find images/paperFig -maxdepth 1 -type f | sort
find files -maxdepth 1 -type f | sort
```

For Markdown/front matter sanity, run:

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

### 5. Commit the Refinement

```bash
git add -A
git commit -m "Refine homepage copy"
```

Use a clear message that says what changed:

```text
Refine homepage copy
Update publication metadata
Improve teaching page content
Replace profile photo
```

### 6. Push the Refinement Branch

```bash
git push -u origin refine-homepage-copy
```

This should trigger the GitHub Actions Jekyll build check.

### 7. Check the Remote Build

Open:

```text
https://github.com/yangyangkiki/yangyangkiki.github.io/actions
```

The build should show:

```text
Jekyll build check: success
```

If it fails:

1. Open the failed workflow.
2. Read the failing step.
3. Fix the issue locally.
4. Commit and push again.

Do not merge into `academic` until the build passes.

### 8. Review Before Merge

Before merging, review the diff:

```bash
git diff academic...HEAD --stat
git diff academic...HEAD
```

Check for:

- No accidental deletion of important content.
- No old template content reintroduced.
- No broken image or PDF paths.
- No navigation links to pages that should not be public.
- Publication front matter has `title`, `collection`, `category`, `permalink`, `date`, and `venue`.

### 9. Merge into `academic`

Important approval point: merging updates the branch used by the live website.

If approved:

```bash
git checkout academic
git pull origin academic
git merge --ff-only refine-homepage-copy
git push origin academic
```

GitHub Pages should rebuild automatically after the push.

### 10. Verify the Live Site

After GitHub Pages finishes deploying, check:

```text
https://yangyangkiki.github.io/
https://yangyangkiki.github.io/publications/
https://yangyangkiki.github.io/teaching/
https://yangyangkiki.github.io/cv/
```

Optional command-line checks:

```bash
curl -L -sS -o /tmp/home.html -w "%{http_code}\n" https://yangyangkiki.github.io/
curl -L -sS -o /tmp/publications.html -w "%{http_code}\n" https://yangyangkiki.github.io/publications/
curl -L -sS -o /tmp/teaching.html -w "%{http_code}\n" https://yangyangkiki.github.io/teaching/
curl -L -sS -o /tmp/cv.html -w "%{http_code}\n" https://yangyangkiki.github.io/cv/
```

Expected result:

```text
200
200
200
200
```

## Common Refinement Tasks

### Update Recent News

Edit:

```text
_pages/about.md
```

Keep newest items at the top. Use a consistent date style, for example:

```markdown
- [June 2026] News text.
```

If the news list gets too long, keep only the newest items visible and fold older items with HTML details:

```markdown
## Recent News

- [June 2026] Newest news item.
- [May 2026] Another important visible item.

<details markdown="1">
<summary>Older news</summary>

- [Jan. 2025] Older news item.
- [Dec. 2024] Older news item.

</details>
```

This keeps the homepage easier to scan while preserving the old news archive.

Recommended rule:

- Keep 5-10 recent or high-value items visible.
- Move older items into `Older news`.
- Remove very old low-value items only after deciding they are no longer useful.

### Add Recent News from GitHub Web

For a one-line news update, GitHub web editing is usually fine.

1. Open `_pages/about.md` on GitHub.
2. Click edit.
3. Add the new item at the top of `## Recent News`.
4. Commit to a new branch, not directly to `academic`.
5. Wait for `Jekyll build check` to pass.
6. Merge the branch.

### Add Recent News Locally

Use local editing when the update touches more than the homepage.

```bash
git checkout academic
git pull origin academic
git checkout -b refine-recent-news
```

Edit `_pages/about.md`, then:

```bash
git add _pages/about.md
git commit -m "Update recent news"
git push -u origin refine-recent-news
```

### Add a Publication

Create a new file:

```text
_publications/YYYY-MM-DD-short-title.md
```

Use this pattern:

```yaml
---
title: "Paper Title"
collection: publications
category: conferences
permalink: /publication/YYYY-short-title
date: YYYY-MM-DD
venue: "Venue Name"
paperurl: "https://example.com/paper"
citation: "Authors. Paper Title. Venue Name."
---

![Paper Title](/images/paperFig/example.png)

Links: [Code](https://example.com/code)

Authors. Paper Title. Venue Name.
```

Use:

- `category: conferences` for conference papers.
- `category: manuscripts` for journal articles.

### Add a Publication Locally

This is the recommended way because publications often include a figure, paper link, code link, and metadata.

1. Create a branch:

```bash
git checkout academic
git pull origin academic
git checkout -b add-publication-short-title
```

2. Add the paper figure, if any:

```text
images/paperFig/short-title.png
```

3. Add a publication Markdown file:

```text
_publications/YYYY-MM-DD-short-title.md
```

4. Check the image path in the Markdown body:

```markdown
![Paper Title](/images/paperFig/short-title.png)
```

5. Commit and push:

```bash
git add _publications/YYYY-MM-DD-short-title.md images/paperFig/short-title.png
git commit -m "Add short title publication"
git push -u origin add-publication-short-title
```

6. Wait for the GitHub Actions build check before merging.

### Add a Publication on GitHub Web

Use this only if there is no new figure/PDF asset, or if the asset is already uploaded.

1. Open the `_publications` folder on GitHub.
2. Choose "Add file" > "Create new file".
3. Name it `YYYY-MM-DD-short-title.md`.
4. Paste the front matter and content.
5. Commit to a new branch.
6. Wait for `Jekyll build check` to pass.
7. Merge after review.

If the publication needs a new image or PDF, local editing is cleaner and less error-prone.

### Move a Section to Its Own Page

Use this when the homepage is getting too long.

Example: move professional activities out of the homepage.

1. Create a page:

```text
_pages/professional-activities.md
```

2. Give it front matter:

```yaml
---
layout: single
title: "Professional Activities"
permalink: /professional-activities/
author_profile: true
---
```

3. Move the content from `_pages/about.md` into the new page.
4. Add the page to `_data/navigation.yml`:

```yaml
  - title: "Activities"
    url: /professional-activities/
```

5. Remove or replace the old homepage section.

### Move Contact Details to the Sidebar

The left sidebar is controlled by `_config.yml`.

Useful fields:

```yaml
author:
  location: "Office 319, PS2 Building, La Trobe University, Bundoora VIC 3083, Australia"
  employer: "La Trobe University"
  uri: "https://scholars.latrobe.edu.au/y4zhao"
  email: "y.zhao2@latrobe.edu.au"
```

If these fields are present, the sidebar shows them automatically.

### Replace the Profile Photo

Add the new image to:

```text
images/
```

Then update:

```yaml
author:
  avatar: "new-profile-image.png"
```

in:

```text
_config.yml
```

### Change Navigation

Edit:

```text
_data/navigation.yml
```

Keep navigation short. Add a page only when the page is ready for public viewing.

### Update the CV PDF

Replace or add a PDF in:

```text
files/
```

Then update the link in:

```text
_pages/cv.md
```

### Adjust Visual Style

Start with small changes in:

```text
_sass/_variables.scss
assets/css/main.scss
```

Avoid large theme rewrites unless the content is already stable.

## Approval Points

Ask for approval before:

- Merging a refinement branch into `academic`.
- Pushing directly to `academic`.
- Removing pages, collections, or many files.
- Changing GitHub Pages settings.
- Replacing important assets such as the profile image or CV PDF.
- Making broad visual/theme changes.

## Rollback

If a refinement breaks the live site, revert the bad commit on `academic`.

```bash
git checkout academic
git pull origin academic
git revert <bad-commit-sha>
git push origin academic
```

Then wait for GitHub Pages to rebuild and verify the live site again.

## Suggested First Refinement Passes

1. Homepage polish: tighten biography, shorten old news, and add current selected highlights.
2. Publication polish: check paper years, venues, citations, and add missing code/demo links.
3. Teaching/CV polish: decide whether teaching, supervision, awards, and experience should be split into separate pages.
4. Visual polish: review profile image size, sidebar details, and typography after content is stable.
