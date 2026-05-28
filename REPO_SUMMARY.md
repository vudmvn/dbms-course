# Repository Summary

Last reviewed: 2026-05-28

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
- Included published folders: `DBMS_Basic`, `MySQL`
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

- `TEMP_MARKDOWN_INPUT.md`
  - Temporary Markdown input file.
  - Currently untracked by git and excluded from the published site.

## Content Inventory

Current homepage sections:

- Course syllabus
  - `assets/files/Syllabus_Database_VNUIS.pdf`

- Reference books
  - Shared cover images in `assets/images/`

- DBMS Basic
  - `DBMS_Basic/gioi_thieu_csdl_vi/gioi_thieu_csdl_vi.md`
  - `DBMS_Basic/gioi_thieu_dbms_vi/gioi_thieu_dbms_vi.md`
  - `DBMS_Basic/nhu_cau_su_dung_dbms_vi/nhu_cau_su_dung_dbms_vi.md`
  - `DBMS_Basic/kien_truc_dbms_vi/kien_truc_dbms_vi.md`
  - `DBMS_Basic/data-abstraction/data-abstraction.md`
  - `DBMS_Basic/data-independence/data-independence.md`
  - `DBMS_Basic/physical-logical-independence/physical-logical-independence.md`
  - `DBMS_Basic/database-schema/database-schema.md`

- MySQL
  - `MySQL/gioi-thieu-mysql/gioi-thieu-mysql.md`
  - `MySQL/huong_dan_cai_dat_mysql_windows/huong_dan_cai_dat_mysql_windows.md`
  - `MySQL/huong_dan_cai_dat_mysql_workbench_windows/huong_dan_cai_dat_mysql_workbench_windows.md`
  - `MySQL/huong_dan_ket_noi_mysql_command_options/huong_dan_ket_noi_mysql_command_options.md`
  - `MySQL/huong_dan_ket_noi_mysql_vscode/huong_dan_ket_noi_mysql_vscode.md`
  - `MySQL/start-stop-MySQL/start-stop-MySQL.md`

## Deployment Flow

1. Add or edit Markdown content under `DBMS_Basic/` or `MySQL/`.
2. Add any lesson images inside that lesson folder, usually under `images/`.
3. Update `index.md` if the lesson should appear on the homepage.
4. Commit and push to `master` or `main`.
5. GitHub Actions workflow `Deploy GitHub Pages` builds with Jekyll and deploys.

Workflow trigger paths:

- `DBMS_Basic/**`
- `MySQL/**`
- `index.md`
- `_config.yml`
- `DEPLOYMENT_RULES.md`
- `.github/workflows/pages.yml`

## Update Notes

- 2026-05-28: Added this repository summary file.
- 2026-05-28: Updated `index.md` with a Vietnamese/English language switcher.
- 2026-05-27: Deployment rules note says the latest deploy activation was on this date.
- Current git status during review showed `TEMP_MARKDOWN_INPUT.md` as untracked.

## Maintenance Notes

- Keep lesson links in `index.md` relative to the repository root.
- Because `_config.yml` uses `baseurl: /dbms-course`, avoid hard-coding root-relative
  links like `/assets/...` unless they are passed through Jekyll URL filters.
- Markdown files do not need YAML front matter because `jekyll-optional-front-matter`
  is enabled.
- If adding a new top-level publishable content folder, also add it to `_config.yml`
  under `include` and update `.github/workflows/pages.yml` trigger paths.
- Avoid relying on `TEMP_MARKDOWN_INPUT.md` for published content; it is excluded.
- PowerShell currently prints a local profile warning about missing
  `D:\Anaconda\Scripts\conda.exe`; command results are still usable despite that
  warning.
