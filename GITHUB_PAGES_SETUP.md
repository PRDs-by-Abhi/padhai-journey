# GitHub Pages setup — PadhAI User Journey

The site is ready in **docs/index.html**. Follow these steps to put it on GitHub Pages.

## 1. Create a new repository on GitHub

1. Go to [github.com/new](https://github.com/new).
2. Repository name: e.g. **padhai-journey** or **projectdost** (your choice).
3. Choose **Public**.
4. Do **not** add a README, .gitignore, or license (we’ll add files locally).
5. Click **Create repository**.

## 2. Initialize Git and push (from your project folder)

In Terminal, run these from **ProjectDost** (replace `YOUR_USERNAME` and `REPO_NAME` with your GitHub username and repo name):

```bash
cd /Users/abhishekverma/Documents/ProjectDost

# Initialize repo (if not already)
git init

# Add the docs folder (this is what GitHub Pages will serve)
git add docs/
git add GITHUB_PAGES_SETUP.md
git commit -m "Add PadhAI User Journey for GitHub Pages"

# Add your GitHub repo as remote and push
git remote add origin https://github.com/YOUR_USERNAME/REPO_NAME.git
git branch -M main
git push -u origin main
```

**If you prefer to push the whole project** (including Android app), run instead:

```bash
git add .
git commit -m "Initial commit: ProjectDost + PadhAI journey docs"
git remote add origin https://github.com/YOUR_USERNAME/REPO_NAME.git
git branch -M main
git push -u origin main
```

## 3. Turn on GitHub Pages

1. Open your repo on GitHub.
2. Go to **Settings** → **Pages** (left sidebar).
3. Under **Build and deployment**:
   - **Source**: Deploy from a branch
   - **Branch**: `main` (or `master`)
   - **Folder**: `/docs`
4. Click **Save**.

## 4. Your live URL

After a minute or two, the site will be at:

**https://YOUR_USERNAME.github.io/REPO_NAME/**

Example: if your repo is `github.com/abhishekverma/padhai-journey`, the site is **https://abhishekverma.github.io/padhai-journey/**.

## Updating the site

When you change `user-journey-flowchart.html` in the project:

```bash
cp user-journey-flowchart.html docs/index.html
git add docs/index.html
git commit -m "Update journey/PRD"
git push
```

The site will refresh in 1–2 minutes.
