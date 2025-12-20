# OptiXDE Site (Template)

This repository is a **standalone public website** for OptiXDE (marketing + docs), while the main OptiXDE codebase can remain **private**.

## Quick start (local preview)

```bash
python -m venv .venv
source .venv/bin/activate  # (Windows) .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

Open the printed local URL (usually http://127.0.0.1:8000).

## Deploy to GitHub Pages

1. Push this repository to GitHub (public).
2. In GitHub: **Settings → Pages**
   - Source: **Deploy from a branch**
   - Branch: **gh-pages** / **root**
3. Every push to `main` will auto-deploy via GitHub Actions.

## Customize

- Edit `mkdocs.yml` (site name, repo_url, nav).
- Edit `docs/index.md` (landing page).
- Put images in `docs/assets/`.

## License

Use your preferred license for the website content (optional). The template does not include a license by default.
