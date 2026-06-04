# Repository Summary

Last reviewed: 2026-06-04

This file is a quick map of the repository so future updates do not need to
rescan the whole folder before making small changes.

## Purpose

This repository hosts DBMS and MySQL learning materials as a GitHub Pages site.

Published site:

- https://vudmvn.github.io/dbms-course

## Site Architecture

- GitHub Pages is built with Jekyll.
- `_config.yml` controls the site metadata, URL, base path, Markdown behavior,
  included content folders, and excluded temporary/build files.
- `index.md` is the homepage and table of contents for the course.
- Content Markdown files are stored under topic folders, usually with local
  `images/` subfolders beside each lesson.
- GitHub Actions builds the site into `_site` and deploys it to Pages.

Important Jekyll settings:

- `url`: `https://vudmvn.github.io`
- `baseurl`: `/dbms-course`
- `markdown`: `kramdown`
- `permalink`: `pretty`
- Explicitly included folders: `DBMS_Basic`, `MySQL`
- `Data-Modeling` is published as a normal top-level content folder and is also
  covered by the Pages workflow trigger paths.
- Excluded temporary/build files include `TEMP_MARKDOWN_INPUT.md`, `node_modules`,
  and common LaTeX/log output files.

## Top-Level Structure

- `.github/workflows/pages.yml`
  - GitHub Actions workflow for building and deploying GitHub Pages.
  - Runs on push to `master` or `main` when relevant site files change.
  - Can also be run manually with `workflow_dispatch`.

- `assets/`
  - Shared site assets.
  - `assets/files/` contains course files such as the syllabus PDF.
  - `assets/images/` contains shared images such as book covers.

- `DBMS_Basic/`
  - Core DBMS lessons.
  - Each lesson generally lives in its own folder.
  - Some lessons include generated/source formats such as `.html`, `.tex`, and `.pdf`
    in addition to the publishable `.md` file.
  - `DBMS_Basic/data-independence/` includes bilingual Beamer source `.tex`
    files and generated Vietnamese/English PDF slides.

- `Data-Modeling/`
  - Data modeling lessons.
  - Current lessons include `data-modeling`, `er-model`, and
    `lab-er-model-1.md`.
  - Lesson images are stored in each lesson's `images/` folder.
  - `Data-Modeling/er-model/images/` contains the ER diagram illustrations used
    by the ER model lesson.
  - `Data-Modeling/er-model/` includes Vietnamese/English Beamer source `.tex`
    files and generated PDF slides for the ER model lesson.
  - Prefer folder names without spaces. The old folder name `Data Modeling` was
    renamed to `Data-Modeling` to avoid awkward URL encoding and link issues.

- `MySQL/`
  - MySQL lessons and how-to guides.
  - Most lesson folders contain one `.md` lesson plus an `images/` folder.

- `_config.yml`
  - Jekyll/GitHub Pages configuration.

- `index.md`
  - Homepage and manual navigation index.
  - Supports a Vietnamese/English language switcher on the same page.
  - When adding a new published lesson, add its link here.

- `DEPLOYMENT_RULES.md`
  - Human-readable deployment notes and update process.

- `REPO_SUMMARY.md`
  - This quick repository map and maintenance note file.

- `TEMP_MARKDOWN_INPUT.md`
  - Temporary Markdown input file.
  - Currently contains the input metadata for `MySQL/storage-engine/` and is
    excluded from the published site.

## Content Inventory

Current homepage sections:

- Course syllabus
  - `assets/files/Syllabus_Database_VNUIS.pdf`

- Reference books
  - Shared cover images in `assets/images/`

- DBMS Basic
  - `DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi.md`
    - Slides: `gioi_thieu_csdl_vi_beamer.pdf`
    - Slides: `gioi_thieu_csdl_en_beamer.pdf`
  - `DBMS_Basic/gioi_thieu_dbms_vi/gioi_thieu_dbms_vi.md`
  - `DBMS_Basic/nhu_cau_su_dung_dbms_vi/nhu_cau_su_dung_dbms_vi.md`
  - `DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.md`
    - Slides: `kien_truc_dbms_vi.pdf`
    - Slides: `kien_truc_dbms_en.pdf`
  - `DBMS_Basic/data-abstraction/data-abstraction.md`
  - `DBMS_Basic/data-independence/data-independence.md`
    - Slides: `data_independence_beamer_pdflatex.pdf`
    - Slides: `data_independence_beamer_english_pdflatex.pdf`
  - `DBMS_Basic/physical-logical-independence/physical-logical-independence.md`
    - Present in the repo and linked in the English homepage table, but hidden
      in the Vietnamese homepage table as of this review.
  - `DBMS_Basic/database-schema/database-schema.md`
  - `DBMS_Basic/choose-right-dbms/choose-right-dbms.md`
    - Slides: `choose-right-dbms-vi.pdf`
    - Slides: `choose-right-dbms-en.pdf`

- Data Modeling
  - `Data-Modeling/data-modeling/data-modeling.md`
  - `Data-Modeling/er-model/er-model.md`
    - Slides: `er-model-vi.pdf`
    - Slides: `er-model-en.pdf`
    - Source: `er-model-vi.tex`
    - Source: `er-model-en.tex`

- Labs
  - `Data-Modeling/lab-er-model-1.md`
    - Shown on the homepage as "Lab 1: ER model basics".
    - Duration: 60m.
    - Difficulty: Beginner.

- MySQL
  - `MySQL/gioi-thieu-mysql/gioi-thieu-mysql.md`
  - `MySQL/huong_dan_cai_dat_mysql_windows/huong_dan_cai_dat_mysql_windows.md`
  - `MySQL/huong_dan_cai_dat_mysql_workbench_windows/huong_dan_cai_dat_mysql_workbench_windows.md`
  - `MySQL/huong_dan_ket_noi_mysql_command_options/huong_dan_ket_noi_mysql_command_options.md`
  - `MySQL/huong_dan_ket_noi_mysql_vscode/huong_dan_ket_noi_mysql_vscode.md`
  - `MySQL/mysql-sample-database/mysql-sample-database.md`
  - `MySQL/load-sample-database/load-sample-database.md`
  - `MySQL/storage-engine/storage-engine.md`
    - Created from `TEMP_MARKDOWN_INPUT.md`.
    - `images/` currently has no downloaded content images from the source page.
  - `MySQL/start-stop-MySQL/start-stop-MySQL.md`

## Deployment Flow

Current working branch and push target:

- Local branch: `deploy-dbms-pages`
- Tracking branch: `course/main`
- GitHub Pages repo remote: `course` -> `https://github.com/vudmvn/dbms-course.git`
- Source/development remote: `origin` -> `https://github.com/vudmvn/dbms.git`
- Because the local branch name differs from `main`, push with:

```powershell
git push course HEAD:main
```

Recommended commit/push process:

1. Check scope first:

```powershell
git status -sb
git diff -- <paths>
```

2. Stage only intended files. Do not stage `TEMP_MARKDOWN_INPUT.md` unless
   explicitly requested. For ER model lessons, keep `.webp` files for Markdown
   pages and matching `.png` files for LaTeX/PDF builds when both are referenced.

```powershell
git add -- <intended-files>
```

3. Commit with a short message:

```powershell
git commit -m "Short change summary"
```

4. Push to the Pages repository:

```powershell
git push course HEAD:main
```

5. Confirm the workflow:

```powershell
gh run list --repo vudmvn/dbms-course --limit 5
```

6. Check the public page and any images directly with `Invoke-WebRequest`.

General content flow:

1. Add or edit Markdown content under `DBMS_Basic/`, `Data-Modeling/`, or `MySQL/`.
2. Add any lesson images inside that lesson folder, usually under `images/`.
3. Update `index.md` if the lesson should appear on the homepage.
4. Commit and push to `course/main`.
5. GitHub Actions workflow `Deploy GitHub Pages` builds with Jekyll and deploys.

Workflow trigger paths:

- `DBMS_Basic/**`
- `Data-Modeling/**`
- `MySQL/**`
- `index.md`
- `_config.yml`
- `DEPLOYMENT_RULES.md`
- `.github/workflows/pages.yml`

## Update Notes

- 2026-06-04: Added the empty `MySQL/storage-engine/` lesson scaffold with an
  `images/` folder placeholder from `TEMP_MARKDOWN_INPUT.md`.
- 2026-06-04: Added `Data-Modeling/lab-er-model-1.md` and a bilingual Labs
  section in `index.md`.
- 2026-06-04: Refreshed the repository summary against current `index.md`,
  `_config.yml`, workflow paths, branch, and remotes.
- 2026-06-01: Added bilingual PDF slides and Beamer source files for
  `DBMS_Basic/data-independence/`, and linked the slides from `index.md`.
- 2026-06-01: Expanded `Data-Modeling/er-model/` with ER diagram illustrations
  for entities, attributes, relationships, cardinality, participation, and ER
  diagram construction steps.
- 2026-06-01: Added Vietnamese and English ER model Beamer slides/PDFs, linked
  them from `index.md`, and refreshed ER cardinality/participation images to
  match the GeeksforGeeks source page.
- 2026-06-01: Added `MySQL/mysql-sample-database/` with the classicmodels
  sample database lesson and local ER diagram image.
- 2026-05-31: Added `Data-Modeling/` lessons and renamed the folder from
  `Data Modeling` to avoid space-encoded URLs.
- 2026-05-31: Added `Data-Modeling/**` to the Pages workflow trigger paths so
  edits in that folder rebuild GitHub Pages.
- 2026-05-31: Documented the practical commit/push flow for the
  `deploy-dbms-pages` branch and `course/main` remote.
- 2026-05-28: Added this repository summary file.
- 2026-05-28: Updated `index.md` with a Vietnamese/English language switcher.
- 2026-05-27: Deployment rules note says the latest deploy activation was on this date.
- Current git status during review was clean before editing this file.

## Maintenance Notes

- Keep lesson links in `index.md` relative to the repository root.
- Because `_config.yml` uses `baseurl: /dbms-course`, avoid hard-coding root-relative
  links like `/assets/...` unless they are passed through Jekyll URL filters.
- For lesson pages using `permalink`, prefer a permalink that matches the lesson
  folder, for example `permalink: /Data-Modeling/er-model/`. This lets local
  image links such as `images/example.png` work both locally and on GitHub Pages.
- If a page is served at a nested pretty URL such as
  `/folder/lesson/lesson/`, relative image links like `images/x.png` will point
  to the wrong folder on the website. Either add an appropriate `permalink` or
  adjust image paths deliberately.
- Markdown files do not need YAML front matter because `jekyll-optional-front-matter`
  is enabled.
- If adding a new top-level publishable content folder, update
  `.github/workflows/pages.yml` trigger paths. Add it to `_config.yml` under
  `include` only when Jekyll needs explicit inclusion for that path.
- Avoid relying on `TEMP_MARKDOWN_INPUT.md` for published content; it is excluded.
- PowerShell currently prints a local profile warning about missing
  `D:\Anaconda\Scripts\conda.exe`; command results are still usable despite that
  warning.
