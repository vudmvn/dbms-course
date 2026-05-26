# Deployment rules

Site dich: https://vudmvn.github.io/dbms

## Dieu kien deploy

- Workflow chay khi push len nhanh `master` hoac `main`.
- Workflow chi tu dong chay khi co thay doi trong `DBMS_Basic/**`, `MySQL/**`, `index.md`, `_config.yml`, `DEPLOYMENT_RULES.md`, hoac `.github/workflows/pages.yml`.
- Co the deploy thu cong bang nut `Run workflow` trong tab GitHub Actions.

## Cach day noi dung

1. Them hoac sua tai lieu trong `DBMS_Basic/` hoac `MySQL/`.
2. Neu them bai moi, cap nhat lien ket trong `index.md`.
3. Commit thay doi.
4. Push len `master` hoac `main`.
5. Kiem tra workflow `Deploy GitHub Pages` trong GitHub Actions.

## Thiet lap tren GitHub

- Vao `Settings` -> `Pages`.
- Chon `Source: GitHub Actions`.
- Sau lan deploy thanh cong dau tien, site se xuat hien tai `https://vudmvn.github.io/dbms`.

## Ghi chu

- `_config.yml` dat `baseurl` la `/dbms` de khop voi duong dan GitHub Pages.
- Cac file Markdown khong can them YAML front matter; Jekyll se render chung nho `jekyll-optional-front-matter`.
- `TEMP_MARKDOWN_INPUT.md` va cac file build tam thoi khong duoc publish.
