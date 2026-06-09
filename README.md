# Simple Secure CI/CD Pipeline (Static Site → GitHub Pages)

A deliberately simple project: a single static web page that is published to
**GitHub Pages** by a **GitHub Actions** pipeline, with a **secret-scanning
security gate** in front of the deploy.

The goal of this repository is to demonstrate a clear, working, secure CI/CD
pipeline end to end — not a complex application.

---

## What the application is

`index.html` — one self-contained static web page (HTML + a little CSS). No
build step, no dependencies, no server. What you see is what gets deployed.

---

## How the pipeline works

There is one workflow: `.github/workflows/deploy.yml`. It runs on every push to
`main` and has two jobs:

1. **Secret Scan** — [Gitleaks](https://github.com/gitleaks/gitleaks) scans the
   code and the full git history for accidentally committed secrets (passwords,
   API keys, tokens).
2. **Deploy to GitHub Pages** — publishes the site. This job has
   `needs: security`, so it **only runs if the secret scan passes**. If a secret
   is ever found, the deploy is blocked.

```
push to main
     │
     ├── Secret Scan (Gitleaks)
     │        │ (must pass)
     │        ▼
     └────► Deploy to GitHub Pages
```

---

## How to reproduce / view it

- **Live site:** after a successful run, it's published at
  `https://<your-github-username>.github.io/<repository-name>/`
- **Locally:** just open `index.html` in any web browser — there's nothing to
  build or install.
- **Re-run the pipeline:** push any change to `main`, or use **Run workflow** on
  the Actions tab (the `workflow_dispatch` trigger).

---

## Security choices and why

| Choice | Why |
|---|---|
| **Secret scanning (Gitleaks)** | Committing a credential by accident is one of the most common and highest-impact security mistakes. Catching it automatically, on every push, prevents secrets from ever reaching a published site or the git history unnoticed. |
| **Security gate before deploy** (`needs: security`) | The deploy can't run unless the scan passes, so a problem stops the pipeline instead of shipping. |
| **Least-privilege permissions** | The workflow defaults to `contents: read`. The deploy job is granted only the extra permissions it needs (`pages: write`, `id-token: write`) and nothing more. |

### What I would add next
- **CodeQL** static analysis and **dependency scanning** if the site grew to
  include build tooling or JavaScript packages.
- **Branch protection** on `main` (require the pipeline to pass before merging).
- **Pin actions to commit SHAs** (instead of tags like `@v4`) and keep them
  updated with **Dependabot**, to defend against a compromised third-party action.
- **GitHub's native secret scanning with push protection** as a second layer.

---

## Repository layout

```
.
├── index.html                       # the static site
├── README.md                        # this file
└── .github/workflows/deploy.yml     # secret scan + deploy to GitHub Pages
```
