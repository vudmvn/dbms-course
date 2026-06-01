# Deployment rules

Site dich: https://vudmvn.github.io/dbms-course

## Dieu kien deploy

- Workflow chay khi push len nhanh `master` hoac `main`.
- Workflow chi tu dong chay khi co thay doi trong `DBMS_Basic/**`, `Data-Modeling/**`, `MySQL/**`, `index.md`, `_config.yml`, `DEPLOYMENT_RULES.md`, hoac `.github/workflows/pages.yml`.
- Co the deploy thu cong bang nut `Run workflow` trong tab GitHub Actions.

## Remote va branch dang dung

- Thu muc lam viec local thuong o `E:\MinhVD\Github\DBMS`.
- Branch local hien tai: `deploy-dbms-pages`.
- Branch nay tracking `course/main`.
- Remote deploy GitHub Pages: `course` -> `https://github.com/vudmvn/dbms-course.git`.
- Remote repo nguon: `origin` -> `https://github.com/vudmvn/dbms.git`.
- Vi ten branch local khac `main`, lenh push nen dung la:

```powershell
git push course HEAD:main
```

## Cach day noi dung

1. Them hoac sua tai lieu trong `DBMS_Basic/`, `Data-Modeling/` hoac `MySQL/`.
2. Neu them bai moi, cap nhat lien ket trong `index.md`.
3. Kiem tra pham vi thay doi truoc khi stage:

```powershell
git status -sb
git diff -- <duong-dan-file>
```

4. Stage dung cac file can commit:

```powershell
git add -- <duong-dan-file>
```

5. Commit voi message ngan gon:

```powershell
git commit -m "Mo ta thay doi"
```

6. Push len repo GitHub Pages:

```powershell
git push course HEAD:main
```

7. Kiem tra workflow `Deploy GitHub Pages` trong GitHub Actions.

```powershell
gh run list --repo vudmvn/dbms-course --limit 5
```

8. Kiem tra trang public va file anh neu co:

```powershell
Invoke-WebRequest -Uri "https://vudmvn.github.io/dbms-course/" -UseBasicParsing
Invoke-WebRequest -Uri "https://vudmvn.github.io/dbms-course/<duong-dan-anh>" -UseBasicParsing
```

## Thiet lap tren GitHub

- Vao `Settings` -> `Pages`.
- Chon `Source: GitHub Actions`.
- Sau lan deploy thanh cong dau tien, site se xuat hien tai `https://vudmvn.github.io/dbms-course`.

## Ghi chu

- `_config.yml` dat `baseurl` la `/dbms-course` de khop voi duong dan GitHub Pages.
- Cac file Markdown khong can them YAML front matter; Jekyll se render chung nho `jekyll-optional-front-matter`.
- `TEMP_MARKDOWN_INPUT.md` va cac file build tam thoi khong duoc publish.
- Khong commit `TEMP_MARKDOWN_INPUT.md` tru khi co yeu cau ro rang.
- Khong commit file anh trung gian `.webp` neu bai giang da dung anh `.png`.
- Nen tranh ten folder co dau cach cho noi dung publish. Dung `Data-Modeling` thay vi `Data Modeling`.
- Neu mot bai co `permalink`, nen dat permalink trung voi folder bai hoc, vi du `permalink: /Data-Modeling/er-model/`. Khi do link anh local dang `images/file.png` se dung ca tren local va GitHub Pages.
- Neu trang bi render tai URL dang `/folder/lesson/lesson/`, link anh `images/file.png` se tro nham den `/folder/lesson/lesson/images/file.png`; can sua permalink hoac duong dan anh.
- Neu sua file trong folder moi ma workflow khong chay, kiem tra muc `paths` trong `.github/workflows/pages.yml`.
- Lan kich hoat deploy gan nhat: 2026-05-31.
