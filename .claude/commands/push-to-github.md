---
description: Security-scan, push to GitHub, and set up README, Pages, CI/CD and the repo About section. Usage - /push-to-github <repo-url>
argument-hint: <github-repo-url>
allowed-tools: Bash(git status:*), Bash(git remote:*), Bash(git branch:*), Bash(git push:*), Bash(git log:*), Bash(git ls-files:*), Bash(git diff:*), Bash(gh auth status:*), Bash(gh repo view:*), Bash(gh repo edit:*), Bash(gh api:*), mcp__playwright__browser_navigate, mcp__playwright__browser_resize, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_close
---

Publish this project to the GitHub repo the user supplied: $ARGUMENTS

Run the steps in order. Stop and report if a step fails. Never use `--force`, and never commit or push without the user's approval.

## 1. Validate input and tooling
- If `$ARGUMENTS` is empty or not a GitHub repo URL (https://github.com/<owner>/<repo>[.git] or git@github.com:<owner>/<repo>.git), ask for the link and stop.
- Derive `<owner>/<repo>` from the URL.
- Run `gh auth status`. Steps 6 and 7 need the GitHub CLI logged in. If it isn't, tell the user to run `gh auth login`, and carry on with the steps that don't need it.

## 2. Security scan (before anything leaves this machine)
Scan every file that would be uploaded: tracked files, untracked files that are not ignored, and `git log -p` history for commits not yet pushed. Look for:
- Secrets: API keys, tokens, passwords, private keys (`-----BEGIN ... PRIVATE KEY-----`), connection strings, AWS/GitHub/Slack/Stripe-style key patterns, JWTs, and `Authorization` or `Bearer` values.
- Sensitive files: `.env*`, `*.pem`, `*.key`, `*.pfx`, `id_rsa*`, credential or config dumps, and `.claude/settings.local.json`.
- Personal data: real email addresses, phone numbers and internal URLs, besides intentional placeholders like `YOUR_EMAIL@example.com`.
- Training documents (`.docx`, `.pdf`) that may be private or copyrighted.

Report each finding as file, line and what it is, with the value masked. If anything is found, stop. Ask the user to remove it or add it to `.gitignore` before continuing. Offer to create or extend `.gitignore` with sensitive patterns. If nothing is found, say so and continue.

## 3. Create or update the README
- If `README.md` doesn't exist, create it. If it does, edit it and keep content that is still correct.
- Base it on the actual project (read the code, don't guess): title, one-line summary, features, how to run it, tech and structure, the live GitHub Pages link (once known), and the CI/CD badge.
- Include a screenshot of the running site (see below) near the top of the README.
- Show the user the diff or new file before it is committed.

### Screenshot (Playwright MCP)
Use the Playwright MCP server configured in `.mcp.json` (load its tools with ToolSearch if they are deferred):
1. If the Pages URL is known and the site is live, open it with `mcp__playwright__browser_navigate`. Otherwise open the local `index.html` through a `file:///` URL.
2. Set the viewport with `mcp__playwright__browser_resize` to 1440x900.
3. Capture it with `mcp__playwright__browser_take_screenshot` (`fullPage: true`, `filename: docs/screenshot.png`), overwriting any earlier screenshot.
4. Look at the image. Confirm it shows the board with no personal data. Retake it if it is blank or broken. Ignore a missing `favicon.ico` console error.
5. Close the browser with `mcp__playwright__browser_close`.
6. Reference it in the README as `![Project Delivery Board](docs/screenshot.png)`.
7. Make sure `.playwright-mcp/` (Playwright's snapshot and log output) is in `.gitignore`. Commit `docs/screenshot.png` only after the user approves it in step 5.

## 4. Create or update CI/CD with GitHub Actions
- Check `.github/workflows/`. Update existing workflows instead of duplicating them.
- Make sure a workflow runs on push and pull request to the default branch. For this static site that means a validation job (for example an HTML check or a basic file check) plus the Pages deploy job that publishes `index.html`.
- Pin actions to major versions, use least-privilege `permissions`, and never put secrets in the workflow file.

## 5. Commit
- Run `git status` and list the new and changed files (README, workflows, `.gitignore`, anything else).
- Ask the user to approve the list. Commit only approved files, with a clear message.

## 6. Push to GitHub
- No `origin` remote: `git remote add origin <url>`.
- `origin` points elsewhere: show both URLs and ask before `git remote set-url origin <url>`.
- Push with `git push -u origin <current-branch>`. If the push is rejected, report the error and ask what to do.

## 7. Create or update GitHub Pages
- Check Pages with `gh api repos/<owner>/<repo>/pages`.
- If it isn't enabled, enable it with the Actions build source: `gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow`. If it is already enabled, make sure its source is GitHub Actions.
- Trigger or wait for the deploy workflow, then read the site URL from `gh api repos/<owner>/<repo>/pages --jq .html_url`.

## 8. Create or update the repo About section
- Read the current values with `gh repo view <owner>/<repo> --json description,homepageUrl,repositoryTopics`.
- Write or update the description (one sentence on what the project is) and relevant topics. Keep existing values the user may have set unless they are wrong. Ask before overwriting a non-empty description.
- Set the website field to the Pages URL from step 7:
  `gh repo edit <owner>/<repo> --description "<text>" --homepage "<pages-url>" --add-topic <topic>`

## 9. Final report
Give the user:
- Security scan result
- Branch pushed, remote URL and latest commit
- Files created or changed (README, workflows, `.gitignore`)
- Pages URL and the About section values now set
- Anything skipped, and why
